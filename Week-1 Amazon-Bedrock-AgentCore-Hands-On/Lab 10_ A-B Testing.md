# A/B Testing

## [Overview](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/86-lab9-optimization/87-ab-testing#overview)

A recommendation is a hypothesis. Before you route all your customers to a new prompt, you want evidence that it's actually better on live traffic. That's what A/B testing gives you: the Gateway splits sessions between the **control** (your current prompt) and the **treatment** (the recommended prompt), online evaluation scores both, and the service reports statistical significance.

**Why this step uses a dedicated A/B runtime**

A/B traffic is split **at the Gateway**, so the Gateway must invoke a runtime through an `http-runtime` target — and it does so using **SigV4/IAM**. But in **Lab 4 you locked the ****`CustomerSupport`**** runtime to ****`CUSTOM_JWT`****-only**, so it rejects the Gateway's SigV4 call with an *"Authorization method mismatch"* error (there's no JWT-passthrough flag on `http-runtime` targets in this preview).

Rather than weaken your production runtime, this step deploys a **dedicated, lightweight A/B runtime** (`CustomerSupportAB`) that keeps the **default IAM authorizer**. The Gateway can invoke it over SigV4, while clients still authenticate to the Gateway with their Cognito token. Your `CustomerSupport` runtime from Labs 4–8 stays untouched and `CUSTOM_JWT`-locked.

Security note: `CustomerSupportAB` is **not** unauthenticated — it uses IAM auth, so only callers with the right IAM permissions (here, the Gateway's execution role) can invoke it. It's a scoped, disposable runtime for experimentation that you tear down at the end.

This gives the config-bundle A/B pattern from the optimization overview — two bundle versions on the same dedicated A/B runtime, split at the Gateway:

```bash
Client (Cognito JWT) → Gateway ──┬── Bundle v1 (control)   ──(SigV4)──→ CustomerSupportAB Runtime
                                 └── Bundle v2 (treatment) ──(SigV4)──→ CustomerSupportAB Runtime
```

### [Step 1: Deploy a dedicated, Gateway-reachable A/B runtime](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/86-lab9-optimization/87-ab-testing#step-1:-deploy-a-dedicated-gateway-reachable-ab-runtime)

Add a second runtime to the project. It's a lightweight, memoryless copy of the agent that reads its system prompt from the active configuration bundle — and, crucially, keeps the **default IAM authorizer** so the Gateway can invoke it over SigV4.

```bash
agentcore add agent \
  --name CustomerSupportAB \
  --language Python \
  --framework Strands \
  --model-provider Bedrock \
  --memory none \
  --build CodeZip
```

Replace the generated `app/CustomerSupportAB/main.py` with a config-bundle-aware agent. It keeps the local support tools, reads the system prompt from the bundle the Gateway assigns (via a Strands `BeforeModelCallEvent` hook), and requires no JWT or memory — so the Gateway's SigV4 invocation succeeds:

```python
"""Customer support agent — dedicated A/B variant (config-bundle aware, IAM auth)."""
from strands import Agent, tool
from strands.models.bedrock import BedrockModel
from strands.hooks.events import BeforeModelCallEvent
from bedrock_agentcore.runtime import BedrockAgentCoreApp, BedrockAgentCoreContext

app = BedrockAgentCoreApp()
log = app.logger

MODEL_ID = "global.anthropic.claude-sonnet-4-6"
DEFAULT_SYSTEM_PROMPT = "You are a helpful and professional customer support assistant for an e-commerce company. Always use tools to get accurate information rather than guessing."

RETURN_POLICIES = {
    "electronics": {"window": "30 days", "condition": "Original packaging required, must be unused or defective", "refund": "Full refund to original payment method"},
    "accessories": {"window": "14 days", "condition": "Must be in original packaging, unused", "refund": "Store credit or exchange"},
    "audio": {"window": "30 days", "condition": "Defective items only after 15 days", "refund": "Full refund within 15 days, replacement after"},
}

PRODUCTS = {
    "PROD-001": {"name": "Wireless Headphones", "price": 79.99, "category": "audio", "description": "Noise-cancelling Bluetooth headphones with 30h battery life"},
    "PROD-002": {"name": "Smart Watch", "price": 249.99, "category": "electronics", "description": "Fitness tracker with heart rate monitor, GPS, and 5-day battery"},
    "PROD-003": {"name": "Laptop Stand", "price": 39.99, "category": "accessories", "description": "Adjustable aluminum laptop stand for ergonomic desk setup"},
    "PROD-004": {"name": "USB-C Hub", "price": 54.99, "category": "accessories", "description": "7-in-1 USB-C hub with HDMI, USB-A, SD card reader, and ethernet"},
    "PROD-005": {"name": "Mechanical Keyboard", "price": 129.99, "category": "electronics", "description": "RGB mechanical keyboard with Cherry MX switches"},
}

@tool
def get_return_policy(product_category: str) -> str:
    """Get return policy information for a specific product category (electronics, accessories, audio)."""
    policy = RETURN_POLICIES.get(product_category.lower())
    if policy:
        return f"Return policy for {product_category.lower()}: Window: {policy['window']}, Condition: {policy['condition']}, Refund: {policy['refund']}"
    return f"No specific return policy found for '{product_category}'. Please contact support."

@tool
def get_product_info(query: str) -> str:
    """Search for product information by name, ID (e.g. PROD-001), or keyword."""
    if query.upper() in PRODUCTS:
        p = PRODUCTS[query.upper()]
        return f"{p['name']} ({query.upper()}): ${p['price']}, Category: {p['category']}, {p['description']}"
    q = query.lower()
    results = [f"{pid}: {p['name']} - ${p['price']} - {p['description']}" for pid, p in PRODUCTS.items()
               if q in p['name'].lower() or q in p['description'].lower() or q in p['category'].lower()]
    return "Found products:\n" + "\n".join(results) if results else f"No products found matching '{query}'."

agent = Agent(
    model=BedrockModel(model_id=MODEL_ID),
    system_prompt=DEFAULT_SYSTEM_PROMPT,
    tools=[get_return_policy, get_product_info],
)

def dynamic_config_hook(event: BeforeModelCallEvent):
    """Apply the system prompt from the active config bundle before each model call."""
    try:
        config = BedrockAgentCoreContext.get_config_bundle()
    except Exception as e:
        log.warning(f"Could not read config bundle, using default prompt: {e}")
        config = {}
    event.agent.system_prompt = config.get("system_prompt", DEFAULT_SYSTEM_PROMPT)

agent.hooks.add_callback(BeforeModelCallEvent, dynamic_config_hook)


@app.entrypoint
def invoke(payload, context):
    result = agent(payload.get("prompt", "Hello"))
    return {"response": result.message["content"][0]["text"]}


if __name__ == "__main__":
    app.run()
```

**Do not add a CUSTOM\_JWT authorizer to this runtime**

Leave `CustomerSupportAB` with its **default IAM authorizer**. Adding `CUSTOM_JWT` (as you did for `CustomerSupport` in Lab 4) would reintroduce the exact mismatch that blocks the Gateway from invoking it.

Deploy the new runtime:

```bash
1
agentcore deploy -y -v
```

### [Step 2: Point the Gateway at the A/B runtime](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/86-lab9-optimization/87-ab-testing#step-2:-point-the-gateway-at-the-ab-runtime)

Add an `http-runtime` target on your Lab 4 gateway (`my-gateway-secure`) that routes to `CustomerSupportAB`:

- **macOS/Linux**
- **Windows**

```bash
agentcore add gateway-target \
  --name customer-support-ab \
  --gateway my-gateway-secure \
  --type http-runtime \
  --runtime CustomerSupportAB
```

### [Step 3: Create the control and treatment configuration bundles](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/86-lab9-optimization/87-ab-testing#step-3:-create-the-control-and-treatment-configuration-bundles)

A **configuration bundle** is a versioned, immutable snapshot of agent configuration (like a system prompt) that can be swapped at runtime without redeploying code. The agent reads its active bundle on each invocation, so you can test different configurations side by side. Here you'll create two: one with your current prompt (control) and one with the optimizer's recommended prompt (treatment).

Create two bundles on the A/B runtime. Use the `{{runtime:CustomerSupportAB}}` placeholder — the CLI resolves it to the real runtime ARN at deploy time, so you don't have to paste ARNs.

**Control** — your current system prompt:

- **macOS/Linux**
- **Windows**

```bash
agentcore add config-bundle \
  --name customerSupportControl \
  --commit-message "Baseline prompt" \
  --components '{"{{runtime:CustomerSupportAB}}": {"configuration": {"system_prompt": "You are a helpful and professional customer support assistant for an e-commerce company. Provide accurate information using the tools available to you. Be friendly, patient, and understanding. Always offer additional help after answering. Always use tools to get accurate information rather than guessing."}}}'```

**Treatment** — uses the `$RECOMMENDED_PROMPT` you saved in the previous step:

- **macOS/Linux**
- **Windows**

```bash
agentcore add config-bundle \
  --name customerSupportTreatment \
  --commit-message "Recommended prompt from cs-prompt-rec" \
  --components "$(jq -n --arg prompt "$RECOMMENDED_PROMPT" '{"{{runtime:CustomerSupportAB}}": {"configuration": {"system_prompt": $prompt}}}')"
```

### [Step 4: Create an online eval for the A/B runtime](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/86-lab9-optimization/87-ab-testing#step-4:-create-an-online-eval-for-the-ab-runtime)

The `QualityMonitor` config from Lab 5 is bound to the `CustomerSupport` runtime's log group, so it won't score sessions on `CustomerSupportAB`. Create a dedicated online eval that monitors the A/B runtime, using the same target evaluator:

```bash
agentcore add online-eval \
  --name ABQualityMonitor \
  --runtime CustomerSupportAB \
  --evaluator Builtin.GoalSuccessRate \
  --sampling-rate 100 \
  --enable-on-create
```

Now deploy everything in one shot (both config bundles + the online eval):

```bash
agentcore deploy -y -v
```

At this point, take a look at everything you have deployed:

```bash
agentcore status
```

```bash
AgentCore Status (target: default, <your-region>)

Agents
  CustomerSupport: Deployed - Runtime: READY
  CustomerSupportAB: Deployed - Runtime: READY

Memories
  SharedMemory: Deployed (SEMANTIC, SUMMARIZATION)

Credentials
  gateway-egress-oauth: Deployed (OAuth)

Gateways
  my-gateway-secure: Deployed (2 targets)

Online Eval Configs
  QualityMonitor: Deployed (3 evaluators, 100% sampling — ACTIVE)
  ABQualityMonitor: Deployed (1 evaluator, 100% sampling — ACTIVE)

Config Bundles
  customerSupportControl: Deployed
  customerSupportTreatment: Deployed

Harnesses
  OrderResearchAgent: Deployed (v2)
```

Two runtimes, two eval configs, two config bundles, a Gateway with two targets, memory, credentials, and a harness. All from CLI commands, no CloudFormation templates written by hand.

### [Step 5: Create the A/B test](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/86-lab9-optimization/87-ab-testing#step-5:-create-the-ab-test)

Start an 80/20 test: 80% of new sessions keep the current prompt (control), 20% get the recommended prompt (treatment). Both variants run on `CustomerSupportAB` and share the `ABQualityMonitor` online eval.

- **macOS/Linux**
- **Windows**

```bash
agentcore run ab-test \
  --mode config-bundle \
  --name cs_prompt_abtest \
  --gateway my-gateway-secure \
  --runtime CustomerSupportAB \
  --control-bundle customerSupportControl \
  --control-version LATEST \
  --treatment-bundle customerSupportTreatment \
  --treatment-version LATEST \
  --online-eval ABQualityMonitor \
  --control-weight 80 \
  --treatment-weight 20
```

The test is **RUNNING** as soon as the command returns (pass `--disable-on-create` to start it stopped). Only one test can run per gateway at a time.

### [Step 6: Send traffic through the Gateway](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/86-lab9-optimization/87-ab-testing#step-6:-send-traffic-through-the-gateway)

The Cedar policy engine from Lab 7 is in ENFORCE mode and will block requests to the A/B target (no permit exists for it). Switch to LOG\_ONLY mode for the duration of the test:

```bash
# Switch policy engine to LOG_ONLY (policies are logged but not enforced)
jq '(.agentCoreGateways[] | select(.name == "my-gateway-secure") | .policyEngineConfiguration.mode) = "LOG_ONLY"' \
  agentcore/agentcore.json > agentcore/agentcore.json.tmp \
  && mv agentcore/agentcore.json.tmp agentcore/agentcore.json

agentcore deploy -y -v
```

Now get the A/B test ID and invocation URL:

```bash
AB_TEST_ID=$(agentcore view ab-test --json | jq -r '.abTests[0].id')
GW_BASE=$(agentcore status --json | jq -r '.deployedState.targets.default.resources.gateways."my-gateway-secure".gatewayUrl')
GATEWAY_URL="${GW_BASE}/customer-support-ab/invocations"

echo "A/B Test ID: $AB_TEST_ID"
echo "Gateway URL: $GATEWAY_URL"
```

Generate traffic by creating and running a load-gen script. The bearer token and gateway URL are baked in from the variables you set above:

```bash
export TOKEN GATEWAY_URL

cat > loadgen.sh << EOF
#!/bin/bash
TOKEN="$TOKEN"
GATEWAY_URL="$GATEWAY_URL"

PROMPTS=(
  "What's the price of the Smart Watch?"
  "My headphones are broken, what should I do?"
  "Is PROD-002 still under warranty?"
  "What's the return policy for audio products?"
  "It stopped working. Can I get a refund?"
  "I want to return my USB-C Hub and check its warranty."
)

for i in \$(seq 1 30); do
  PROMPT="\${PROMPTS[\$(( (i - 1) % \${#PROMPTS[@]} ))]}"
  SESSION_ID=\$(python3 -c "import uuid; print(str(uuid.uuid4()) + '-' + str(uuid.uuid4())[:8])")
  echo "=== Request \$i: \$PROMPT ==="
  curl -s \\
    -H "Authorization: Bearer \$TOKEN" \\
    -H "Content-Type: application/json" \\
    -H "X-Amzn-Bedrock-AgentCore-Runtime-Session-Id: \$SESSION_ID" \\
    -d "{\"prompt\": \"\$PROMPT\"}" \\
    -X POST "\$GATEWAY_URL"
  echo ""
  sleep 2
done
EOF

bash loadgen.sh
```

Each request returns a normal agent response. Because `CustomerSupportAB` uses the default IAM authorizer, the Gateway's SigV4 call now succeeds — no "Authorization method mismatch". The Gateway assigns each new session to control or treatment and injects the matching config bundle, so the two prompt variants run side by side on the same runtime.

### [Step 7: Read the results](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/86-lab9-optimization/87-ab-testing#step-7:-read-the-results)

In another tab, poll the test as sessions complete. Polling does not affect statistical validity.

```bash
1
agentcore view ab-test $AB_TEST_ID --json
```

**Forgot the test ID?**

Run `agentcore view ab-test --json | jq -r '.abTests[0].id'` to retrieve it.

**Results take time**

Results appear after sessions complete — typically within \~15 minutes of a session's last request, depending on the session timeout in your online eval config. Statistical significance improves as more sessions accumulate.

Each evaluator reports the control mean, the treatment mean, and the difference with a p-value and significance flag:

| ResultInterpretationAction                    |                                   |                                                   |
| --------------------------------------------- | --------------------------------- | ------------------------------------------------- |
| `p-value < 0.05` and positive `percentChange` | Treatment is significantly better | Promote the treatment                             |
| `p-value < 0.05` and negative `percentChange` | Treatment is significantly worse  | Keep the control                                  |
| `p-value >= 0.05`                             | Not enough evidence yet           | Keep collecting samples or raise treatment weight |

> ***Check every evaluator.**** A change can lift **`GoalSuccessRate`** while regressing **`Correctness`**. Review all three **`QualityMonitor`** evaluators before deciding.*

### [Step 8: Promote the winner (or stop)](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/86-lab9-optimization/87-ab-testing#step-8:-promote-the-winner-\(or-stop\))

Here's what a winning result looks like in the JSON output:

```json
"evaluatorMetrics": [{
  "controlStats": {"mean": 0.50, "sampleSize": 45},
  "variantResults": [{
    "mean": 0.78,
    "sampleSize": 42,
    "percentChange": 56.0,
    "pValue": 0.012,
    "isSignificant": true,
    "confidenceInterval": {"lower": 0.08, "upper": 0.48}
  }]
}]
```

The treatment wins when: `isSignificant: true`, `pValue < 0.05`, the confidence interval doesn't cross zero (both `lower` and `upper` are positive), and `percentChange` is positive. In this example, the optimized prompt achieved 78% goal success vs 50% for the control, with high confidence that the improvement is real.

Once you've validated the treatment is better, you could stop the test and apply the winning prompt to your production agent (update the system prompt in `main.py` or your production config bundle):

**Automated promote (same-bundle versioning)**

If you structured your test with two versions of the **same** bundle (rather than two separate bundles as in this lab), you can use `promote` to automatically update the bundle and roll it out: `agentcore promote ab-test -i <test-id>` followed by `agentcore deploy -y -v`.

## [Clean up the A/B test resources](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/86-lab9-optimization/87-ab-testing#clean-up-the-ab-test-resources)

Stop and archive the A/B test:

```bash
1
2
agentcore stop ab-test -i $AB_TEST_ID
agentcore archive ab-test -i $AB_TEST_ID
```

> *The full workshop teardown (all resources) is covered in the Summary.*

## [What just happened?](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/86-lab9-optimization/87-ab-testing#what-just-happened)

You ran the full production improvement cycle: generated AI-optimized recommendations from real traces, packaged them as versioned config bundles, validated them with a controlled A/B test on live traffic, and promoted the winner. The traces from the winning variant become the baseline for your next round.

## [Congratulations! ✅](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/86-lab9-optimization/87-ab-testing#congratulations!)

You've completed the final lab. Here's the full picture of what you built:

| LabWhat you built |                                                               |
| ----------------- | ------------------------------------------------------------- |
| 1                 | Agent prototype with local tools                              |
| 2                 | Persistent memory across sessions                             |
| 3                 | Centralized tools via Gateway                                 |
| 4                 | Security, observability, and session management               |
| 5                 | Continuous quality monitoring                                 |
| 6                 | Customer-facing chat interface                                |
| 7                 | Fine-grained governance with Cedar policies                   |
| 8                 | Zero-code agents with Harness                                 |
| 9                 | Data-driven optimization with recommendations and A/B testing |

### [What's next](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/86-lab9-optimization/87-ab-testing#what's-next)

Head to the summary to review everything and tear down your resources.

→ Next: [Summary](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/90-summary/)
