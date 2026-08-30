# Lab 7: Governing Agent Actions with Policies

**⏱️ Estimated time: ~20 minutes**

## [Overview](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/80-lab7-policies#overview)

Your customer support agent is deployed, secured with JWT authentication, and monitored with evaluations. But authentication only answers *"who is calling?"* — it doesn't answer *"what are they allowed to do?"*

Consider this scenario: you add a refund processing tool to your agent. Should every authenticated user be able to issue refunds of any amount? What if a customer asks the agent to refund $10,000? Without governance, the agent will happily comply — it has no concept of business rules or spending limits.

**AgentCore Policy** solves this by adding fine-grained authorization at the Gateway boundary using [Cedar ](https://www.cedarpolicy.com/) policies. Policies are evaluated deterministically *outside* the agent's code, so the agent can't accidentally bypass them — even if it's tricked by a clever prompt.

### [Key concepts](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/80-lab7-policies#key-concepts)

| Concept | Description |
| --- | --- |
| **Policy Engine**  | A container for Cedar policies that evaluates authorization requests            |
| **Cedar Policy**   | A declarative rule that permits or forbids access to a tool based on conditions |
| **ENFORCE mode**   | Policy decisions are enforced — denied requests are blocked at the Gateway      |
| **LOG_ONLY mode** | Policy decisions are logged but not enforced (useful for testing)               |
| **Default Deny**   | All actions are denied unless explicitly permitted by a Cedar policy            |

## [Step 0: Set Up Your Terminals](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/80-lab7-policies#step-0:-set-up-your-terminals)

Lab 7 uses both the chat UI (to test policy enforcement) and the CLI (to create policies). Open a new terminal tab for the CLI commands — the frontend will keep running in the other tab.

**In your new terminal**, start the frontend from Lab 6:

```bash
cd app/CustomerSupport/frontend
uv run python frontend.py
```

You should see `✅ Authenticated as workshopuser@example.com` and the server running. Open the frontend using the Code Editor's port forwarding (click **Open in Browser** when the notification appears, or use the Ports tab).

**Open another terminal tab** for the CLI commands in this lab. Make sure you're in the project root:

```bash
cd ~/workshop/CustomerSupport
```

## [Step 1: Add the Refund Tool to Your Gateway](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/80-lab7-policies#step-1:-add-the-refund-tool-to-your-gateway)

The prerequisites stack includes a Lambda function (`workshop-process-refund`) that simulates processing customer refunds. Let's expose it through your secured Gateway so the agent can call it.

### [Retrieve the Lambda ARN](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/80-lab7-policies#retrieve-the-lambda-arn)

- **macOS/Linux**
- **Windows**

```bash
REFUND_LAMBDA_ARN=$(aws ssm get-parameter \
  --name /app/customersupport/agentcore/refund_lambda_arn \
  --query 'Parameter.Value' --output text)

echo "Refund Lambda ARN: $REFUND_LAMBDA_ARN"
```

### [Create the tool schema](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/80-lab7-policies#create-the-tool-schema)

Create the schema file that describes the refund tool to the agent:

- **macOS/Linux**
- **Windows**

```bash
touch app/CustomerSupport/tool/refund_schema.json
```

Open `app/CustomerSupport/tool/refund_schema.json` in your editor and add:

```json
[
  {
    "name": "process_refund",
    "description": "Process a customer refund for a given order. Requires the order ID, refund amount in dollars, and a reason for the refund.",
    "inputSchema": {
      "type": "object",
      "properties": {
        "order_id": {
          "type": "string",
          "description": "The order ID to refund (e.g., ORD-12345)"
        },
        "amount": {
          "type": "integer",
          "description": "Refund amount in whole dollars"
        },
        "reason": {
          "type": "string",
          "description": "Reason for the refund (e.g., defective item, wrong product, customer dissatisfied)"
        }
      },
      "required": ["order_id", "amount", "reason"]
    }
  }
]
```

> **Why `"type": "integer"`?** Cedar uses Long type for whole numbers, which maps directly from JSON Schema **`integer`**. This lets us write simple comparisons like **`context.input.amount < 100`** in our policies. If we used **`"type": "number"`** (which maps to Cedar Decimal), we'd need the more verbose **`.lessThan(decimal("100.00"))`** syntax.*

### [Add the refund target to the Gateway](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/80-lab7-policies#add-the-refund-target-to-the-gateway)

- **macOS/Linux**
- **Windows**

```bash
agentcore add gateway-target \
  --type lambda-function-arn \
  --name ProcessRefund \
  --lambda-arn $REFUND_LAMBDA_ARN \
  --tool-schema-file app/CustomerSupport/tool/refund_schema.json \
  --gateway my-gateway-secure
```

### [Deploy](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/80-lab7-policies#deploy)

```bash
agentcore deploy -y -v
```

### [Test the refund tool (no policy yet)](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/80-lab7-policies#test-the-refund-tool-\(no-policy-yet\))

At this point, the refund tool is available but has no policy restrictions. Open the chat UI in your browser (via port forwarding, same as Lab 6) and try:

**Chat not working?**

If the chat interface isn't responding after a redeploy, hard-refresh your browser with `Cmd+Shift+R` (macOS) or `Ctrl+Shift+R` (Windows/Linux) to clear any cached pages.

```bash
I'd like a refund of $500 for order ORD-12345 because the item was defective
```

The agent should successfully process the refund — there's nothing stopping it. Any authenticated user can request any refund amount. Let's fix that.

## [Step 2: Create a Policy Engine and Attach to Gateway](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/80-lab7-policies#step-2:-create-a-policy-engine-and-attach-to-gateway)

A Policy Engine is a container that holds your Cedar policies and evaluates them against incoming requests. Create one for your customer support application and attach it to your gateway:

- **macOS/Linux**
- **Windows**

```bash
agentcore add policy-engine \
  --name CustomerSupportPolicyEngine \
  --description "Governs customer support agent tool access — refund limits and tool permissions" \
  --attach-to-gateways my-gateway-secure \
  --attach-mode ENFORCE
```

You should see output like:

```bash
Added policy engine 'CustomerSupportPolicyEngine'
```

> **ENFORCE vs LOG_ONLY:** In **`ENFORCE`** mode, denied requests are blocked and the tool call fails. In **`LOG_ONLY`** mode, all requests are allowed but policy decisions are logged to CloudWatch — useful for testing policies before enforcing them.*

## [Step 3: Create Cedar Policies](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/80-lab7-policies#step-3:-create-cedar-policies)

Now write the authorization rules. We'll create two policies:

1. **Permit refunds under $100** — allows the refund tool only for small amounts
2. **Permit warranty checks** — explicitly allows the existing warranty tool for all users

First, retrieve your Gateway ARN — you'll need it in the Cedar policy statements:

- **macOS/Linux**
- **Windows**

```bash
GATEWAY_ID=$(aws bedrock-agentcore-control list-gateways \
  --query "items[?contains(name, 'my-gateway-secure')].gatewayId | [0]" \
  --output text)

GATEWAY_ARN=$(aws bedrock-agentcore-control get-gateway \
  --gateway-identifier $GATEWAY_ID \
  --query "gatewayArn" --output text)

echo "Gateway ARN: $GATEWAY_ARN"
```

> **Note:** You can also find the Gateway ARN in **`agentcore/.cli/deployed-state.json`** or from the **`agentcore status`** output.*

### [Policy 1: Refund limit](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/80-lab7-policies#policy-1:-refund-limit)

This policy permits the `process_refund` tool only when the amount is less than 100:

```cedar
permit(
  principal,
  action == AgentCore::Action::"ProcessRefund___process_refund",
  resource == AgentCore::Gateway::"<YOUR_GATEWAY_ARN>"
)
when {
  ((context.input).amount) < 100
};
```

> **Understanding the Cedar syntax:**
>
> - *`permit`** — allows the action (Cedar also supports **`forbid`** to deny)*
> - *`principal`** — any authenticated user (from the JWT token)*
> - *`action == AgentCore::Action::"ProcessRefund___process_refund"`** — the specific tool (format: **`TargetName___tool_name`** with triple underscores)*
> - *`resource == AgentCore::Gateway::"<arn>"`** — scoped to your Gateway ARN*
> - *`when { context.input.amount < 100 }`** — only when the refund amount is under $100*

Now create this policy using the CLI:

- **macOS/Linux**
- **Windows**

```bash
agentcore add policy \
  --name refund_limit_policy \
  --engine CustomerSupportPolicyEngine \
  --description "Allow refunds under 100 dollars only" \
  --statement "permit(principal, action == AgentCore::Action::\"ProcessRefund___process_refund\", resource == AgentCore::Gateway::\"${GATEWAY_ARN}\") when { context.input.amount < 100 };"
```

### [Policy 2: Warranty check access](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/80-lab7-policies#policy-2:-warranty-check-access)

This policy permits the warranty check tool unconditionally for all authenticated users:

**Why is this policy needed?**

Cedar uses **default deny** — once you attach a Policy Engine in ENFORCE mode, every tool call through the Gateway needs an explicit `permit` policy to succeed. Without this policy, the warranty check tool (which worked fine before) would start failing with authorization errors. This policy preserves existing functionality.

- **macOS/Linux**
- **Windows**

```bash
agentcore add policy \
  --name warranty_check_policy \
  --engine CustomerSupportPolicyEngine \
  --description "Allow all authenticated users to check warranties" \
  --statement "permit(principal, action == AgentCore::Action::\"WarrantyCheck___check_warranty\", resource == AgentCore::Gateway::\"${GATEWAY_ARN}\") when { (principal is AgentCore::OAuthUser) };" \
  --validation-mode IGNORE_ALL_FINDINGS
```

**Want help understanding Cedar syntax?**

Run `claude` in your terminal and ask it to explain the Cedar policy statements (e.g., "explain how the refund limit Cedar policy works"). Cedar is a new language for most developers — no shame in asking.

### [Deploy the policies](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/80-lab7-policies#deploy-the-policies)

```bash
agentcore deploy -y -v
```

## [Step 4: Test Policy Enforcement via the Chat UI](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/80-lab7-policies#step-4:-test-policy-enforcement-via-the-chat-ui)

Open the chat UI in your browser (hard-refresh with `Cmd+Shift+R` if needed). The agent now has the refund tool available, but it's governed by your Cedar policies. Try these:

| Prompt | Expected | Why |
| --- | --- | --- |
| `I need a refund of $50 for order ORD-12345. The item arrived damaged.` | ✅ Refund processed                           | amount=50, policy permits (< 100) |
| `Process a refund of $500 for order ORD-67890. I want a full refund.`   | ❌ Denied — agent suggests contacting support | amount=500, policy denies (≥ 100) |
| `Check the warranty for PROD-002`                                       | ✅ Warranty returned                          | explicit permit for all users     |

> **Key insight:** The agent code never changed, and neither did the Lambda function code. Policy enforcement happens entirely at the Gateway boundary — before the request ever reaches the Lambda. The agent simply receives an error when a policy denies the action.*

## [Step 5: (Bonus) Generate a Policy from Natural Language](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/80-lab7-policies#step-5:-\(bonus\)-generate-a-policy-from-natural-language)

AgentCore Policy can generate Cedar policies from plain English descriptions. This is useful when you want to add new rules without learning Cedar syntax:

- **macOS/Linux**
- **Windows**

```bash
agentcore add policy \
  --name refund_reason_policy \
  --engine CustomerSupportPolicyEngine \
  --generate "Forbid refunds when the reason does not contain the word defective" \
  --gateway my-gateway-secure
```

The CLI translates your natural language into a valid Cedar policy, validates it against the tool schema, and checks for safety issues — all before you deploy it.

### [Inspect the generated Cedar](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/80-lab7-policies#inspect-the-generated-cedar)

Open `agentcore/agentcore.json` and look at the new policy entry under `policyEngines` and `policies`. You should see something like:

```json
{
  "name": "refund_reason_policy",
  "statement": "forbid(\n  principal,\n  action == AgentCore::Action::\"ProcessRefund___process_refund\",\n  resource == AgentCore::Gateway::\"<YOUR_GATEWAY_ARN>\"\n) unless {\n  ((context.input).reason) like \"*defective*\"\n};",
  "validationMode": "FAIL_ON_ANY_FINDINGS"
}
```

Notice how the CLI automatically:

- Identified the correct action name (`ProcessRefund___process_refund`)
- Scoped the policy to your gateway ARN
- Translated "does not contain the word defective" into Cedar's `forbid ... unless { reason like "*defective*" }` pattern
- Used `forbid` with `unless` — this means the refund is **blocked** unless the reason contains "defective". Since Cedar's `forbid` overrides `permit`, this policy takes precedence over the amount-based permit policy

### [Deploy the generated policy](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/80-lab7-policies#deploy-the-generated-policy)

```bash
agentcore deploy -y -v
```

### [Test the refund reason policy](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/80-lab7-policies#test-the-refund-reason-policy)

| Prompt | Expected |
| --- | --- |
| `I need a refund of $50 for order ORD-99999 because the item was defective` | ✅ Permitted — reason contains "defective"     |
| `I need a refund of $50 for order ORD-11111 because I changed my mind`      | ❌ Denied — reason doesn't contain "defective" |

## [Architecture](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/80-lab7-policies#architecture)

After completing this lab, your architecture includes AgentCore Policy enforcement at the Gateway boundary:

![Lab 7 Architecture](images/lab7_architecture_diagram.png)

## [Why this matters](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/80-lab7-policies#why-this-matters)

| Without Policy | With Policy |
| --- | --- |
| Any authenticated user can call any tool            | Fine-grained control over what each tool can do            |
| Agent decides whether to process a $10,000 refund   | Gateway blocks it before it reaches the Lambda             |
| Business rules live in prompt engineering (fragile) | Business rules are deterministic Cedar policies (reliable) |
| No audit trail of authorization decisions           | Every decision logged to CloudWatch                        |
| Changing rules means changing agent code            | Changing rules means updating a policy, no redeploy needed |

## [Congratulations! ✅](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/80-lab7-policies#congratulations!)

You added governance to your agent without changing a single line of agent code. Policies enforce at the Gateway boundary, deterministically, regardless of what the agent tries to do.

### [Cedar patterns to explore](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/80-lab7-policies#cedar-patterns-to-explore)

If you want to go further, here are other patterns Cedar supports:

| Pattern | Example |
| --- | --- |
| Amount limits         | `context.input.amount < 1000`                         |
| Role-based access     | `principal.getTag("role") == "manager"`               |
| Required fields       | `forbid ... unless { context.input has description }` |
| Regional restrictions | `["US", "CA"].contains(context.input.region)`         |
| Emergency shutdown    | `forbid(principal, action, resource)`                 |

### [What's next](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/80-lab7-policies#what's-next)

**Choose the next step based on your workshop Region**

If your workshop event was launched in `us-east-1`, you can continue to the optional Policy Guardrails bonus. If it was launched in `us-west-2`, Policy Guardrails are not available there, so skip the bonus and continue directly to Lab 8. Do not manually switch Regions because the workshop resources were deployed in the Region selected when the event was created.

→ In `us-east-1` (optional): [Bonus: Guarding Refund Tool Inputs with Policy Guardrails](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/80-lab7-policies/83-guardrails/)

→ In another workshop Region: [Lab 8: Zero-Code Agents with AgentCore Harness](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/85-lab8-harness/)
