# Bonus: Guarding Refund Tool Inputs with Policy Guardrails

**⏱️ Estimated time: ~10–15 minutes**

Continue in the `CustomerSupport` project from Lab 7. You will add a semantic sensitive-information guardrail to the existing `ProcessRefund` target while keeping the refund limit, warranty access, JWT authentication, and default-deny behavior unchanged.

> **Run this bonus in a `us-east-1` workshop event**
>
> The Workshop Studio configuration lets the event operator select `us-east-1` or `us-west-2`. Policy guardrails are currently available in `us-east-1`, but not in `us-west-2`.
>
> Complete this bonus only when the event was launched in `us-east-1`. Do not manually switch Regions because the prerequisite resources follow the Region selected at event creation.

> **This is the shared Lab 7 project**
>
> Run every command from:
>
> ```text
> ~/workshop/CustomerSupport
> ```
>
> Do **not** run:
>
> ```bash
> agentcore remove all
> ```
>
> That would remove resources created throughout Labs 1–7. The optional cleanup removes only the new guardrail policy.

## [What You Will Learn](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/80-lab7-policies/83-guardrails#what-you-will-learn)

Lab 7 already uses deterministic Cedar policies to permit refunds under `$100` and warranty checks.

This bonus adds a semantic `forbid` policy that detects email addresses in the refund tool's `reason` argument immediately before the Gateway invokes the refund Lambda.

```text
Customer → CustomerSupport Runtime → my-gateway-secure → ProcessRefund Lambda
                                          │
                                          ├─ refund_limit_policy (deterministic)
                                          └─ BlockSensitiveRefundReasons (semantic)
```

This is a **tool-input guardrail**, not a pre-model user-message guardrail.

The `CustomerSupport` model sees the customer's message first and constructs the tool arguments. The guardrail then evaluates:

```text
context.input.reason
```

at the Gateway boundary.

This exercise uses an email address that is part of a legitimate product complaint because the model is more likely to preserve ordinary business data than an explicit prompt-injection instruction.

## [Step 1: Verify the Existing Lab 7 Environment](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/80-lab7-policies/83-guardrails#step-1:-verify-the-existing-lab-7-environment)

Open a terminal and confirm the Region and project:

```bash
cd ~/workshop/CustomerSupport
aws configure list
aws sts get-caller-identity
agentcore status
```

Before continuing, verify all of the following:

- The active Region is `us-east-1`.
- Gateway `my-gateway-secure` is deployed.
- Target `ProcessRefund` exists on that Gateway.
- Policy Engine `CustomerSupportPolicyEngine` is attached in `ENFORCE` mode.
- `refund_limit_policy` and `warranty_check_policy` from Lab 7 are deployed.

## [Step 2: Block Sensitive Information in the Refund Reason](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/80-lab7-policies/83-guardrails#step-2:-block-sensitive-information-in-the-refund-reason)

Create a sensitive-information guardrail scoped to the existing refund tool.

The important data path is:

```text
context.input.reason
```

This matches the `reason` property in `refund_schema.json`.

The policy checks that field for the `EMAIL` entity documented by Bedrock Guardrails.

### Retrieve the Gateway ARN

First, retrieve the deployed Gateway ARN again because it must appear in the explicit Cedar statement.

#### macOS/Linux

```bash
GATEWAY_ID=$(aws bedrock-agentcore-control list-gateways \
  --query "items[?contains(name, 'my-gateway-secure')].gatewayId | [0]" \
  --output text)

GATEWAY_ARN=$(aws bedrock-agentcore-control get-gateway \
  --gateway-identifier $GATEWAY_ID \
  --query "gatewayArn" --output text)

echo "Gateway ARN: $GATEWAY_ARN"
```

### Add the Guardrail Policy

Now add the policy with an explicit MCP tool action.

#### macOS/Linux

```bash
agentcore add policy \
  --name BlockSensitiveRefundReasons \
  --engine CustomerSupportPolicyEngine \
  --statement "forbid(principal, action == AgentCore::Action::\"ProcessRefund___process_refund\", resource == AgentCore::Gateway::\"${GATEWAY_ARN}\") when guardrails { BedrockGuardrails::SensitiveInformation([\"EMAIL\"], [context.input.reason]).maxConfidenceScore().greaterThanOrEqual(decimal(\"0.2\")) };" \
  --validation-mode IGNORE_ALL_FINDINGS \
  --enforcement-mode ACTIVE
```

Open:

```text
agentcore/agentcore.json
```

and inspect the new policy.

Confirm that it contains all of the following:

- Action: `ProcessRefund___process_refund`
- Data path: `context.input.reason`
- Safeguard: `BedrockGuardrails::SensitiveInformation(["EMAIL"], ...)`
- Required aggregation: `maxConfidenceScore()`
- Threshold: `greaterThanOrEqual(decimal("0.2"))`

> **Why use `maxConfidenceScore()`?**
>
> `SensitiveInformation` requires an aggregation such as `count()` or `maxConfidenceScore()`. It does not support per-category indexing such as:
>
> ```text
> ["EMAIL"].confidenceScore
> ```
>
> Because this policy requests only the `EMAIL` category, the maximum score is the email-detection score.

## [Step 3: Deploy the Guardrail](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/80-lab7-policies/83-guardrails#step-3:-deploy-the-guardrail)

```bash
agentcore deploy -y -v
```

The deployment adds the policy and updates the CLI-managed Gateway execution role with:

```text
bedrock:InvokeGuardrailChecks
```

The Policy service needs this permission to evaluate the content.

## [Step 4: Test All Policy Paths](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/80-lab7-policies/83-guardrails#step-4:-test-all-policy-paths)

> **Refresh the chat UI**
>
> Keep the Lab 6 frontend running and hard-refresh the chat UI after deployment.
>
> A refresh renews authentication but does not erase actor-level long-term Memory, so the model's tool arguments can still vary between attempts.

Test these prompts in order:

| Prompt | Expected Result | Policy Behavior |
| --- | --- | --- |
| `Hi, I need a $60 refund for order ORD-24680. The defective item was personalized with the wrong email address, alice@example.com. Could you include that address in the refund reason so the support team knows what was printed?` | ⚠️ Likely: the email-bearing call is denied; the model may retry without the address | The guardrail blocks `EMAIL` in `context.input.reason` |
| `Process a refund of $500 for order ORD-45646. I want a full refund.` | ❌ Tool call denied | Existing refund limit does not permit amounts ≥ `$100` |
| `Check the warranty for PROD-002` | ✅ Warranty returned | Existing warranty permit remains unchanged |

If long-term Memory says the test order was already refunded, repeat the prompt with a different unused order ID.

This changes only the simulated order record, not the policy path being tested.

### Important: What the Guardrail Actually Evaluates

The guardrail does **not** evaluate the original customer message.

It evaluates only the value the model places in:

```text
context.input.reason
```

Two outcomes remain valid.

#### Outcome 1: Email Reaches the Gateway

The model includes:

```text
alice@example.com
```

inside `reason`.

`BlockSensitiveRefundReasons` detects `EMAIL` and denies that invocation before Lambda execution.

The model may then retry without the email address.

#### Outcome 2: Email Is Removed Before the Gateway

The model generalizes the reason to something like:

```text
Defective item personalized with wrong email address
```

The Gateway sees no email entity, so no guardrail denial appears and the `$50` refund can succeed.

### Inspect the Actual Tool Arguments

Run:

```bash
agentcore logs --since 15m --query "process_refund"
```

In the logs, find:

```text
gen_ai.tool.call.arguments
```

Compare the result with these representative outcomes:

```text
Outcome A — email-bearing call denied, followed by a safe retry:

gen_ai.tool.call.arguments: reason="Defective item personalized with wrong email address alice@example.com"
ERROR: Policy evaluation denied due to BlockSensitiveRefundReasons-...

gen_ai.tool.call.arguments: reason="Defective item personalized with wrong email address"
SUCCESS: Refund processed


Outcome B — email removed before the first Gateway call:

gen_ai.tool.call.arguments: reason="Defective item personalized with wrong email address"
SUCCESS: Refund processed
```

A successful final refund does not by itself prove either that the guardrail triggered or that it was bypassed.

If the email is present in `gen_ai.tool.call.arguments`, expect that invocation to identify `BlockSensitiveRefundReasons` as the denying policy.

If the email is absent, the model removed it before policy evaluation.

### [What Happened](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/80-lab7-policies/83-guardrails#what-happened)

1. The Runtime received the JWT-authenticated customer message and selected `process_refund`.
2. The model constructed `order_id`, `amount`, and `reason`. This is where behavior can vary.
3. If `reason` contained `alice@example.com`, the sensitive-information check returned an `EMAIL` confidence score.
4. At or above the `0.2` threshold, the matching guardrail `forbid` overrode the amount-based `permit`, and the Gateway rejected the invocation before Lambda execution.
5. If the model omitted the address initially—or retried without it—the guardrail did not match, and the existing amount policy could permit the `$50` refund.

> **Production rollout**
>
> Guardrail scoring and model-generated tool arguments are probabilistic.
>
> In production:
>
> 1. Start the new policy in `LOG_ONLY`.
> 2. Run a representative test set.
> 3. Review confidence scores and false positives.
> 4. Switch the policy to `ACTIVE`.
>
> This bonus uses `ACTIVE` and the documented sensitive-information threshold of `0.2` so you can observe enforcement immediately.

## [Step 5: Optional Targeted Cleanup](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/80-lab7-policies/83-guardrails#step-5:-optional-targeted-cleanup)

You can keep this policy for later labs.

To remove only the bonus guardrail while preserving every Lab 1–7 resource and policy, run:

### macOS/Linux

```bash
cd ~/workshop/CustomerSupport

agentcore remove policy \
  --name BlockSensitiveRefundReasons \
  --engine CustomerSupportPolicyEngine \
  -y

agentcore deploy -y -v
```

The refund limit and warranty policies remain deployed.

> **Important:** Never use:
>
> ```bash
> agentcore remove all
> ```
>
> for this cleanup.

## [Congratulations! ✅](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/80-lab7-policies/83-guardrails#congratulations!)

You extended the existing Lab 7 architecture with semantic sensitive-information protection for refund tool inputs without changing the:

- Agent
- Lambda
- Gateway
- Policy Engine
- Deterministic authorization rules

### [Learn More](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/80-lab7-policies/83-guardrails#learn-more)

- [Getting started with guardrails in the AgentCore CLI](https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/policy-guardrails-getting-started.html)
- [Guardrails in policies: targets, categories, effects, thresholds, Regions, and IAM](https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/policy-guardrails-in-policies.html)

### [What's Next](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/80-lab7-policies/83-guardrails#what's-next)

In Lab 8, you'll create a zero-code agent using AgentCore Harness and connect it to the same Gateway and policies you extended in Lab 7.

→ Next: [Lab 8: Zero-Code Agents with AgentCore Harness](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/85-lab8-harness/)
