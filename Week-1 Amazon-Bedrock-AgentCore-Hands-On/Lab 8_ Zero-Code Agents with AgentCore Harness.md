# Lab 8: Zero-Code Agents with AgentCore Harness

**⏱️ Estimated time: \~25 minutes**

## [Overview](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/85-lab8-harness#overview)

What if you don't need custom orchestration code? What if a foundation model, a system prompt, and a set of tools is all your use case requires?

**AgentCore Harness** is a zero-code, declarative way to create and deploy agents using only CLI configuration. You describe *what* the agent should do, and Harness handles the *how*. No `main.py`, no dependency management, no build step.

In this lab you'll create a fully functional "Order Research Agent" using only the CLI, connect it to your existing secured Gateway (with Cedar policies), and explore capabilities like shell access, filesystem persistence, model overrides, and human-in-the-loop approval flows.

---

## [Step 1: Create the Harness](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/85-lab8-harness#step-1:-create-the-harness)

You define the entire agent — model, system prompt, and tools — in a single CLI command. No Python code, no `main.py`, no dependency files; Harness takes care of orchestration so you can focus on the agent's behavior.

- **macOS/Linux**
- **Windows**

```bash
1
2
3
4
5
6
agentcore add harness \
  --name OrderResearchAgent \
  --model-provider bedrock \
  --model-id us.anthropic.claude-sonnet-4-6 \
  --system-prompt "You are an order research specialist. Help customers investigate order issues, check warranties, and produce detailed analysis reports. Always be thorough and provide structured summaries. Save reports to /tmp/ when asked." \
  --tools agentcore_code_interpreter
```

**Zero-code by design**

No Python code, no `main.py`, no `pyproject.toml`. Just configuration.

This writes a declarative config to `app/OrderResearchAgent/harness.json`. After Step 1 it looks like this:

```json
1
2
3
4
5
6
7
8
9
10
11
12
{
  "name": "OrderResearchAgent",
  "model": { "provider": "bedrock", "modelId": "us.anthropic.claude-sonnet-4-6" },
  "systemPrompt": "You are an order research specialist. ...",
  "tools": [
    { "type": "agentcore_code_interpreter", "name": "code-interpreter" }
  ],
  "skills": [],
  "memory": { 
    "mode": "disabled" 
  }
}
```

---

## [Step 2: Connect the Existing Gateway (with OAuth outbound auth)](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/85-lab8-harness#step-2:-connect-the-existing-gateway-\(with-oauth-outbound-auth\))

Your Gateway from Labs 3–7 already has `check_warranty` and `process_refund` tools protected by Cedar policies. You connect the harness to that same Gateway so it inherits the tools and governance.

There's one important difference from the Runtime agent. In Lab 4 the Runtime authenticated to the **CUSTOM\_JWT** Gateway by *forwarding the caller's Cognito bearer token* (token passthrough). The zero-code harness has no inbound user token to forward, so it must obtain its own Cognito token using a **machine-to-machine (client\_credentials) OAuth credential provider**. You configure that as the gateway tool's **outbound auth**.

**If you skip the outbound auth**

`agentcore add tool --type agentcore_gateway` defaults outbound auth to `awsIam` (SigV4). Against a CUSTOM\_JWT Gateway that is rejected — the harness fails at invoke time with `Failed to load tool 'my-gateway-secure' ... 401 Unauthorized` for the gateway `/mcp` endpoint. The steps below wire OAuth so this works on the first deploy.

### [Retrieve the Cognito values and build the provider ARN](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/85-lab8-harness#retrieve-the-cognito-values-and-build-the-provider-arn)

You reuse the Cognito **machine** app client created in the prerequisites (the same one used for `client_credentials`). The egress credential provider's ARN is deterministic — `…:token-vault/default/oauth2credentialprovider/<provider-name>` — so you can reference it before it exists.

- **macOS/Linux**
- **Windows**

```bash
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
REGION=${AWS_REGION:-$(aws configure get region)}
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)

COGNITO_DISCOVERY_URL=$(aws ssm get-parameter --name /app/customersupport/agentcore/cognito_discovery_url --query 'Parameter.Value' --output text)
COGNITO_MACHINE_CLIENT_ID=$(aws ssm get-parameter --name /app/customersupport/agentcore/client_id --query 'Parameter.Value' --output text)
COGNITO_POOL_ID=$(aws ssm get-parameter --name /app/customersupport/agentcore/pool_id --query 'Parameter.Value' --output text)
COGNITO_SCOPE=$(aws ssm get-parameter --name /app/customersupport/agentcore/cognito_auth_scope --query 'Parameter.Value' --output text)
COGNITO_MACHINE_SECRET=$(aws cognito-idp describe-user-pool-client \
  --user-pool-id "$COGNITO_POOL_ID" --client-id "$COGNITO_MACHINE_CLIENT_ID" \
  --query 'UserPoolClient.ClientSecret' --output text)

# The egress credential provider ARN is deterministic — construct it up front
PROVIDER_ARN="arn:aws:bedrock-agentcore:${REGION}:${ACCOUNT_ID}:token-vault/default/oauth2credentialprovider/gateway-egress-oauth"

# The secured Gateway ARN from Lab 4
GATEWAY_ID=$(aws bedrock-agentcore-control list-gateways \
  --query "items[?contains(name, 'my-gateway-secure')].gatewayId | [0]" --output text)
GATEWAY_ARN=$(aws bedrock-agentcore-control get-gateway \
  --gateway-identifier "$GATEWAY_ID" --query "gatewayArn" --output text)
```

### [Create the egress OAuth credential provider](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/85-lab8-harness#create-the-egress-oauth-credential-provider)

- **macOS/Linux**
- **Windows**

```bash
1
2
3
4
5
agentcore add credential --type oauth --name gateway-egress-oauth \
  --discovery-url "$COGNITO_DISCOVERY_URL" \
  --client-id "$COGNITO_MACHINE_CLIENT_ID" \
  --client-secret "$COGNITO_MACHINE_SECRET" \
  --scopes "$COGNITO_SCOPE"
```

### [Add the Gateway tool with OAuth outbound auth](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/85-lab8-harness#add-the-gateway-tool-with-oauth-outbound-auth)

Reference the Gateway by ARN (the `--gateway <name>` form may not resolve from local state) and bind the credential provider with the `CLIENT_CREDENTIALS` grant:

- **macOS/Linux**
- **Windows**

```bash
1
2
3
4
5
6
7
agentcore add tool --harness OrderResearchAgent --type agentcore_gateway \
  --name my-gateway-secure \
  --gateway-arn "$GATEWAY_ARN" \
  --outbound-auth oauth \
  --provider-arn "$PROVIDER_ARN" \
  --scopes "$COGNITO_SCOPE" \
  --grant-type CLIENT_CREDENTIALS
```

Your `app/OrderResearchAgent/harness.json` now has the gateway tool with an `outboundAuth.oauth` block (account ID masked; your ARNs will differ):

```json
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
{
  "type": "agentcore_gateway",
  "name": "my-gateway-secure",
  "config": {
    "agentCoreGateway": {
      "gatewayArn": "arn:aws:bedrock-agentcore:REGION:123456789012:gateway/customersupport-my-gateway-secure-XXXXXXXXXX",
      "outboundAuth": {
        "oauth": {
          "providerArn": "arn:aws:bedrock-agentcore:REGION:123456789012:token-vault/default/oauth2credentialprovider/gateway-egress-oauth",
          "scopes": ["default-m2m-resource-server-XXXXXXXX/read"],
          "grantType": "CLIENT_CREDENTIALS"
        }
      }
    }
  }
}
```

---

## [Step 3: Deploy](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/85-lab8-harness#step-3:-deploy)

You deploy the harness agent to AgentCore the same way you deployed the Runtime agent — a single command provisions everything and makes the agent available for invocations.

```bash
1
agentcore deploy -y -v
```

---

## [Step 4: Test the Harness Agent](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/85-lab8-harness#step-4:-test-the-harness-agent)

Now you verify the agent works end-to-end. This invocation asks the agent to call `check_warranty` via the Gateway for two products and produce a written report — exercising both the Gateway tools and Code Interpreter in a single turn.

- **macOS/Linux**
- **Windows**

```bash
1
2
3
4
5
6
SESSION=$(uuidgen)
echo "Session: $SESSION"
agentcore invoke --harness OrderResearchAgent \
  --session-id "$SESSION" \
  --actor-id "analyst-1" \
  "Check the warranty for PROD-001 and PROD-003. Save a comparison report to /tmp/warranty_report.md summarizing which products are still covered."
```

The agent will:

1. Call `check_warranty` via the Gateway for both products
2. Use Code Interpreter or shell to write the report
3. Return a comprehensive analysis

---

## [Step 5: Shell Access — Inspect the Environment](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/85-lab8-harness#step-5:-shell-access-inspect-the-environment)

Unlike a pure API call, the harness gives you a full Linux environment. The `--exec` flag runs a command inside the harness microVM so you can inspect what the agent created, prepare the workspace before it runs, or execute deterministic scripts.

**Same session ID required**

These commands must use the same `$SESSION` value from Step 4. If you opened a new terminal or lost the variable, re-run `SESSION=$(uuidgen)` first (but note that a new session won't have the files the previous session created).

**\`--exec\` behavior in the preview CLI**

In AgentCore CLI, `--exec` runs your command in the harness environment but the result is returned **through the agent** (you'll see a short conversational summary around the output), so it is not a zero-token raw shell passthrough. The command itself still executes deterministically in the microVM — handy for retrieving files and checking the environment — just don't assume it bypasses the model entirely in this version.

- **macOS/Linux**
- **Windows**

```bash
1
2
3
4
5
6
7
agentcore invoke --exec --harness OrderResearchAgent \
  --session-id "$SESSION" \
  "cat /tmp/warranty_report.md"

agentcore invoke --exec --harness OrderResearchAgent \
  --session-id "$SESSION" \
  "python3 --version && ls -la /tmp/"
```

Notice the output: the commands executed deterministically in the microVM, but the *result* came back through the agent (it summarized what it found rather than giving you raw stdout). You can see the report exists at `/tmp/warranty_report.md`, Python 3.10 is installed, and the session is persistent. The "To resume" line at the bottom confirms you can pick up this same session later.

**When to use shell access**

Imagine your agent generates a CSV report. Rather than reasoning about row counts in prose, run `wc -l /tmp/report.csv` directly to execute it deterministically in the environment.

---

## [Step 6: Filesystem Persistence Within a Session](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/85-lab8-harness#step-6:-filesystem-persistence-within-a-session)

The harness maintains a real filesystem per session. Files created in one invocation are available in the next, so the agent can iteratively build documents, datasets, or code across multiple turns without losing context.

- **macOS/Linux**
- **Windows**

```bash
1
2
3
4
5
6
7
8
agentcore invoke --harness OrderResearchAgent \
  --session-id "$SESSION" \
  --actor-id "analyst-1" \
  "Read the report at /tmp/warranty_report.md and add a recommendation section for each expired product."

agentcore invoke --exec --harness OrderResearchAgent \
  --session-id "$SESSION" \
  "cat /tmp/warranty_report.md"
```

---

## [Step 7: Override Model Per Invocation](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/85-lab8-harness#step-7:-override-model-per-invocation)

Sometimes you want to test whether a cheaper model handles a task just as well, or switch to a more capable one for complex reasoning. You can override the model per invocation without redeploying — the harness configuration and session state remain unchanged.

- **macOS/Linux**
- **Windows**

```bash
1
2
3
4
5
agentcore invoke --harness OrderResearchAgent \
  --model-id us.amazon.nova-2-lite-v1:0 \
  --session-id "$SESSION" \
  --actor-id "analyst-1" \
  "Summarize the warranty report in exactly 3 bullet points."
```

---

## [Step 8: Verify Policy Enforcement Through Harness](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/85-lab8-harness#step-8:-verify-policy-enforcement-through-harness)

The Cedar policies you defined in Lab 7 don't care which agent is calling — they guard the Gateway itself. This means your harness agent inherits the exact same governance rules as your Runtime agent without any additional configuration.

- **macOS/Linux**
- **Windows**

```bash
1
2
3
4
agentcore invoke --harness OrderResearchAgent \
  --session-id "$(uuidgen)" \
  --actor-id "analyst-1" \
  'Process a refund of $500 for order ORD-12345 because the customer is unhappy.'
```

Expected: The policy denies it (amount >= 100, reason doesn't contain "defective"). The agent reports it cannot process the request.

**Same Gateway, same rules**

You built the governance layer once in Lab 7 — every agent that connects to this Gateway inherits the same rules without any additional configuration.

---

## [Step 9: Human-in-the-Loop with Inline Functions](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/85-lab8-harness#step-9:-human-in-the-loop-with-inline-functions)

Inline functions pause the agent and return control to the caller — this is the pattern for human approvals, external API calls, or any logic that can't run on the harness VM. You don't write any code; you declare a tool and the harness handles the pause/resume lifecycle for you.

The scenario: a customer needs a $200 refund (blocked by the <$100 policy). Instead of a hard deny, the agent escalates to a human for approval.

### [How inline functions work](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/85-lab8-harness#how-inline-functions-work)

```
1. Agent reasons and decides to call the inline function
2. Harness streams the response with stopReason: "tool_use"
3. Your code receives the tool call details (name, input, toolUseId)
4. Your code executes the logic (prompt human, call API, etc.)
5. Your code sends the result back via invoke_harness with:
   - The assistant's toolUse message (echoed back)
   - The user's toolResult message (your result)
6. Agent resumes reasoning with the tool result
```

### [Add the inline function tool](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/85-lab8-harness#add-the-inline-function-tool)

- **macOS/Linux**
- **Windows**

```bash
1
2
3
4
5
6
agentcore add tool --harness OrderResearchAgent --type inline_function \
  --name approve_exception \
  --description "Request manager approval for a refund that exceeds the automated limit. Returns approved or denied with approver name." \
  --input-schema '{"type":"object","properties":{"order_id":{"type":"string"},"amount":{"type":"number"},"reason":{"type":"string"}},"required":["order_id","amount","reason"]}'

agentcore deploy -y -v
```

### [Create the HITL test script](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/85-lab8-harness#create-the-hitl-test-script)

**CLI limitation**

The AgentCore CLI (`agentcore invoke --harness`) does not yet support resuming inline function calls with tool results. The CLI treats the resume input as a new user message rather than a `toolResult`. To demonstrate the full HITL round-trip (pause → human approval → resume), this step uses a short Python script that calls boto3's `invoke_harness` API directly. Once CLI support is added, this can be simplified to a single interactive command.

Create the file `app/OrderResearchAgent/test_hitl.py` in your editor (or use `touch app/OrderResearchAgent/test_hitl.py`) and paste the following. This script uses boto3's `invoke_harness` API to handle the full pause/resume flow:

```python
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
60
61
62
63
64
65
66
67
68
69
70
71
72
73
74
75
76
77
78
79
80
81
82
83
84
85
86
87
88
89
90
91
92
93
94
95
96
97
98
99
100
101
102
103
104
105
106
107
108
109
110
111
112
113
114
115
116
117
118
119
120
121
122
123
124
125
126
127
"""
Human-in-the-Loop test for AgentCore Harness inline functions.

Usage:
    export HARNESS_ARN=<arn>
    python3 test_hitl.py

    python3 test_hitl.py --harness-arn <arn>
    python3 test_hitl.py --prompt "Your custom prompt"
"""

import boto3
import json
import uuid
import os
import argparse

DEFAULT_PROMPT = (
    "A customer needs a $200 refund for order ORD-55555 because they received "
    "a damaged product. Try to process it, and if you can't, escalate for manager approval."
)

def parse_args():
    parser = argparse.ArgumentParser(description="Test HITL inline functions with AgentCore Harness")
    parser.add_argument("--harness-arn", default=os.environ.get("HARNESS_ARN"),
                        help="Harness ARN (default: HARNESS_ARN env var)")
    parser.add_argument("--prompt", default=DEFAULT_PROMPT, help="Prompt to send to the agent")
    return parser.parse_args()

def parse_stream(response):
    """Parse streaming response. Returns tool call info and stop reason."""
    tool_use_id = None
    tool_name = None
    tool_input_chunks = ""
    stop_reason = None
    current_block_is_tool = False

    for event in response["stream"]:
        if "contentBlockStart" in event:
            start = event["contentBlockStart"].get("start", {})
            if "toolUse" in start:
                tool_use_id = start["toolUse"]["toolUseId"]
                tool_name = start["toolUse"].get("name")
                tool_input_chunks = ""
                current_block_is_tool = True
            else:
                current_block_is_tool = False
        elif "contentBlockDelta" in event:
            delta = event["contentBlockDelta"].get("delta", {})
            if "text" in delta:
                print(delta["text"], end="", flush=True)
            elif "toolUse" in delta and current_block_is_tool:
                tool_input_chunks += delta["toolUse"].get("input", "")
        elif "contentBlockStop" in event:
            current_block_is_tool = False
        elif "messageStop" in event:
            stop_reason = event["messageStop"].get("stopReason")

    print()

    tool_input = None
    if tool_input_chunks:
        try:
            tool_input = json.loads(tool_input_chunks)
        except json.JSONDecodeError:
            decoder = json.JSONDecoder()
            try:
                tool_input, _ = decoder.raw_decode(tool_input_chunks)
            except json.JSONDecodeError:
                tool_input = {}

    return {
        "tool_use_id": tool_use_id,
        "tool_name": tool_name,
        "tool_input": tool_input,
        "stop_reason": stop_reason,
    }

def main():
    args = parse_args()
    harness_arn = args.harness_arn
    if not harness_arn:
        print("Error: HARNESS_ARN not set. Export it or pass --harness-arn.")
        exit(1)

    client = boto3.client("bedrock-agentcore")
    session_id = str(uuid.uuid4())

    # Step 1: Send the prompt
    response = client.invoke_harness(
        harnessArn=harness_arn,
        runtimeSessionId=session_id,
        messages=[{"role": "user", "content": [{"text": args.prompt}]}],
    )
    result = parse_stream(response)

    # Step 2: If the agent paused on an inline function, prompt for approval
    if result["stop_reason"] == "tool_use" and result["tool_name"] == "approve_exception":
        print(f"\nAgent called: {result['tool_name']}")
        print(f"Input: {json.dumps(result['tool_input'], indent=2)}")
        approval = input("\nApprove this refund? (yes/no): ").strip().lower()

        if approval in ("yes", "y"):
            tool_result = json.dumps({"approved": True, "approver": "manager-jane"})
        else:
            tool_result = json.dumps({"approved": False, "reason": "Manager denied"})

        # Step 3: Resume with toolUse + toolResult
        tool_input_value = result["tool_input"] if isinstance(result["tool_input"], dict) else {}
        response = client.invoke_harness(
            harnessArn=harness_arn,
            runtimeSessionId=session_id,
            messages=[
                {"role": "assistant", "content": [
                    {"toolUse": {"toolUseId": result["tool_use_id"], "name": result["tool_name"], "input": tool_input_value}}
                ]},
                {"role": "user", "content": [
                    {"toolResult": {"toolUseId": result["tool_use_id"], "content": [{"text": tool_result}], "status": "success"}}
                ]},
            ],
        )
        parse_stream(response)
    else:
        print(f"\nAgent completed. Stop reason: {result['stop_reason']}")

if __name__ == "__main__":
    main()
```

### [Run the test](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/85-lab8-harness#run-the-test)

- **macOS/Linux**
- **Windows**

```bash
1
2
3
4
5
6
7
export HARNESS_ARN=$(aws bedrock-agentcore-control list-harnesses \
  --query "harnesses[?contains(harnessName, 'OrderResearchAgent')].arn | [0]" \
  --output text)

echo "HARNESS_ARN=$HARNESS_ARN"

app/CustomerSupport/.venv/bin/python app/OrderResearchAgent/test_hitl.py
```

### [Expected output](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/85-lab8-harness#expected-output)

```bash
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
Sure! Let me first attempt to process the $200 refund directly.
The automated refund was denied by policy. Let me immediately escalate
this to a manager for approval!

Agent called: approve_exception
Input: {
  "order_id": "ORD-55555",
  "amount": 200,
  "reason": "Customer received a damaged product..."
}

Approve this refund? (yes/no): yes

### Refund processing summary — ORD-55555

1. **Automated Refund Attempt** — Blocked by policy (amount >= $100)
2. **Manager Escalation** — Approved by Manager Jane

The $200 refund for order ORD-55555 has been approved...
```

### [What just happened](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/85-lab8-harness#what-just-happened)

1. The agent tried `process_refund` via the Gateway → **Cedar policy blocked it** (amount >= 100)
2. The agent recognized the failure and called `approve_exception` → **harness paused** with `stopReason: "tool_use"`
3. The Python script detected the pause, printed the tool call, and prompted you
4. You typed "yes" → script sent the `toolUse` + `toolResult` messages back via `invoke_harness`
5. The agent resumed with the approval and provided a complete summary

### [Key API detail](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/85-lab8-harness#key-api-detail)

When resuming after an inline function pause, you must send **both messages** together:

```python
1
2
3
4
5
6
messages=[
    # Echo back the assistant's tool call
    {"role": "assistant", "content": [{"toolUse": {"toolUseId": ..., "name": ..., "input": ...}}]},
    # Provide your result
    {"role": "user", "content": [{"toolResult": {"toolUseId": ..., "content": [...], "status": "success"}}]},
]
```

The harness intentionally does not persist the inline function turn. If the client never returns a result, the session remains clean — this is why both messages are required.

### [Production patterns](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/85-lab8-harness#production-patterns)

In a real application, the "human" in the loop could be:

- A **Slack approval** — bot posts the request, waits for a reaction, sends the result
- A **queue** — request goes to SQS, a worker picks it up, human reviews in a dashboard
- A **another agent** — a supervisor agent evaluates the request programmatically
- A **web UI** — the frontend shows a modal, user clicks approve/deny

**Why this matters**

Lab 7 policies set hard boundaries deterministically. Inline functions create exception paths without changing those policies — the agent learns to escalate when the primary tool fails. In production, the "human" could be another system, a Slack approval, or a queue.

---

---

## [Architecture](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/85-lab8-harness#architecture)

## [What just happened?](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/85-lab8-harness#what-just-happened)

You created and deployed a fully functional agent without writing application code. The harness handled orchestration, session management, and execution. You connected it to the same secured Gateway from Lab 7, proving that Cedar policies apply uniformly regardless of which agent calls the tools. And you built a human-in-the-loop approval flow using nothing but a tool declaration.

## [Congratulations! ✅](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/85-lab8-harness#congratulations!)

You now have two agents deployed side by side: one built with custom Python code (Labs 1-7) and one built with zero code (this lab). Both share the same Gateway, the same tools, and the same governance policies.

Want to explore more Harness capabilities? Check out the [Bonus: Advanced Harness Features](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/85-lab8-harness/82-bonus/) page for session storage mounts, custom container images, agent skills, and additional resources.

### [What's next](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/85-lab8-harness#what's-next)

→ Next: [Lab 9: Optimizing Agent Quality](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/86-lab9-optimization/)
