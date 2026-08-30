# Lab 5: Evaluating Agent Quality

**⏱️ Estimated time: ~15 minutes**

## [Overview](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/60-lab5-evals#overview)

Your agent is deployed, secured, and observable. But how do you know if it's actually giving *good* answers? Are customers getting their problems solved? Is the agent picking the right tools?

In this lab, you'll set up continuous quality monitoring using AgentCore Evaluations. This automatically assesses your agent's performance on every interaction (or a sample) using built-in evaluators.

### [How online evaluation works](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/60-lab5-evals#how-online-evaluation-works)

AgentCore Evaluations monitors your deployed agent by sampling a percentage of live sessions and scoring them automatically using LLM-as-a-Judge. You configure which evaluators to apply and what sampling rate to use. Results flow into CloudWatch where you can track trends over time.

AgentCore provides 13 built-in evaluators (plus the ability to create custom ones). In this lab we'll use three:

| Built-in Evaluator | What It Measures |
| --- | --- |
| **Builtin.GoalSuccessRate** | Did the agent achieve the customer's goal? |
| **Builtin.Correctness** | Is the information factually accurate? |
| **Builtin.ToolSelectionAccuracy** | Did the agent pick the right tools? |

Other built-in evaluators cover helpfulness, faithfulness, harmfulness, coherence, conciseness, instruction-following, and more. You can also define custom evaluators with your own scoring prompts for business-specific criteria.

## [Step 1: Create Online Evaluation Configuration](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/60-lab5-evals#step-1:-create-online-evaluation-configuration)

In your code editor, add an online evaluation config that monitors your `CustomerSupport` agent with all three built-in evaluators.

### macOS/Linux

```bash
agentcore add online-eval \
  --name QualityMonitor \
  --runtime CustomerSupport \
  --evaluator Builtin.GoalSuccessRate Builtin.Correctness Builtin.ToolSelectionAccuracy \
  --sampling-rate 100 \
  --enable-on-create
```

### Windows

```bash
agentcore add online-eval \
  --name QualityMonitor \
  --runtime CustomerSupport \
  --evaluator Builtin.GoalSuccessRate Builtin.Correctness Builtin.ToolSelectionAccuracy \
  --sampling-rate 100 \
  --enable-on-create
```

You should see:

```text
Added online eval 'QualityMonitor'
```

> **Note:** We use `--sampling-rate 100` (100%) for this workshop so every interaction is evaluated. In production, you'd typically use 10-20% to balance cost and coverage. The `--enable-on-create` flag activates evaluation immediately after deployment.

## [Step 2: Deploy the Evaluation Configuration](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/60-lab5-evals#step-2:-deploy-the-evaluation-configuration)

```bash
agentcore deploy -y -v
```

This deploys the online evaluation configuration alongside your existing resources. The evaluators will start monitoring new interactions automatically.

After deployment, verify the evaluation is active:

```bash
agentcore status
```

## [Step 3: Generate Test Interactions](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/60-lab5-evals#step-3:-generate-test-interactions)

### Already have traces from Lab 4?

If you just completed Lab 4, you already have traces in CloudWatch. These additional invocations ensure the evaluators have fresh, varied data to score — but they're not strictly required if your Lab 4 traffic is recent.

Since the runtime is now secured with Cognito (Lab 4), make sure you have a valid token. If your token has expired or you're in a new terminal session, retrieve it again.

### macOS/Linux

```bash
# Skip this block if $TOKEN is still set from Lab 4
COGNITO_POOL_ID=$(aws ssm get-parameter \
  --name /app/customersupport/agentcore/pool_id \
  --query 'Parameter.Value' --output text)

COGNITO_WEB_CLIENT_ID=$(aws ssm get-parameter \
  --name /app/customersupport/agentcore/web_client_id \
  --query 'Parameter.Value' --output text)

TOKEN=$(aws cognito-idp initiate-auth \
  --auth-flow USER_PASSWORD_AUTH \
  --client-id $COGNITO_WEB_CLIENT_ID \
  --auth-parameters USERNAME=workshopuser@example.com,PASSWORD='WorkshopPass1!' \
  --query 'AuthenticationResult.AccessToken' --output text)

echo "Token obtained successfully"
```

### Windows

```powershell
# Retrieve the Cognito pool ID
$COGNITO_POOL_ID = aws ssm get-parameter `
  --name /app/customersupport/agentcore/pool_id `
  --query "Parameter.Value" --output text

# Retrieve the Cognito web client ID
$COGNITO_WEB_CLIENT_ID = aws ssm get-parameter `
  --name /app/customersupport/agentcore/web_client_id `
  --query "Parameter.Value" --output text

# Retrieve a new access token
$TOKEN = aws cognito-idp initiate-auth `
  --auth-flow USER_PASSWORD_AUTH `
  --client-id $COGNITO_WEB_CLIENT_ID `
  --auth-parameters USERNAME=workshopuser@example.com,PASSWORD='WorkshopPass1!' `
  --query "AuthenticationResult.AccessToken" --output text

Write-Host "Token obtained successfully"
```

In your code editor, generate varied interactions to give the evaluators something to assess.

### macOS/Linux

```bash
SESSION_EVAL=$(python3 -c 'import uuid; print(uuid.uuid4())')

# Product information query
agentcore invoke "What can you tell me about the Smart Watch? What's the price and warranty?" \
  --session-id $SESSION_EVAL --bearer-token "$TOKEN" --stream

# Return policy query
agentcore invoke "I bought headphones last week but they're not working. What's the return policy for audio products?" \
  --session-id $SESSION_EVAL --bearer-token "$TOKEN" --stream

# Warranty check (via Gateway)
agentcore invoke "Check the warranty status for product PROD-001" \
  --session-id $SESSION_EVAL --bearer-token "$TOKEN" --stream

# Multi-tool query
agentcore invoke "I want to return my USB-C Hub. What's the policy, and can you check if it's still under warranty?" \
  --session-id $SESSION_EVAL --bearer-token "$TOKEN" --stream

# General capability query
agentcore invoke "What kind of support can you provide? List your capabilities." \
  --session-id $SESSION_EVAL --bearer-token "$TOKEN" --stream
```

> **Note:** Evaluation results take a few minutes to process after interactions are generated. The next two steps give traces time to propagate before we run an on-demand evaluation.

## [Step 4: Understand Evaluation Scores](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/60-lab5-evals#step-4:-understand-evaluation-scores)

While the traces are being indexed, let's understand what the evaluators measure and how to interpret the results.

### [What the Scores Mean](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/60-lab5-evals#what-the-scores-mean)

| Score Range | Interpretation | Action |
| --- | --- | --- |
| 80-100% | Excellent | Monitor and maintain |
| 60-80% | Good but improvable | Review low-scoring sessions |
| Below 60% | Needs attention | Investigate and fix root causes |

### [Common Improvements](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/60-lab5-evals#common-improvements)

- **Low Goal Success Rate** → Refine the system prompt, add more specific tool descriptions.
- **Low Correctness** → Update product data, improve tool response formatting.
- **Low Tool Selection** → Improve tool descriptions, add examples to the system prompt.

### [Via CloudWatch Console](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/60-lab5-evals#via-cloudwatch-console)

For a visual dashboard with trends and detailed scores:

1. Navigate to the [CloudWatch console](https://console.aws.amazon.com/cloudwatch/).
2. Go to **GenAI Observability** → **Bedrock AgentCore**.
3. Click on your **CustomerSupport** agent.
4. Click on the **DEFAULT** endpoint.
5. Click on the **Evaluations** tab to view scores.

The dashboard shows:

- **Goal Success Rate** — Are customers getting their problems solved?
- **Correctness** — Is the information accurate?
- **Tool Selection Accuracy** — Is the agent using the right tools?

## [Step 5: Run On-Demand Evaluation](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/60-lab5-evals#step-5:-run-on-demand-evaluation)

Your traces should be indexed by now. Run an on-demand evaluation against the last day's interactions:

```bash
agentcore run eval \
  --runtime CustomerSupport \
  --evaluator Builtin.GoalSuccessRate Builtin.Correctness \
  --days 1
```

This evaluates all traces from the last day using the specified evaluators. You should see output like:

```text
Agent: CustomerSupport | Jul 19, 2026, 09:54 PM | Sessions: 11 | Lookback: 1d
Builtin.GoalSuccessRate: 0.73
Builtin.Correctness: 0.84
```

This tells you that across 11 sessions in the last day, the agent achieved the customer's goal 73% of the time and gave factually correct information 84% of the time. These scores are generated by an LLM judge reviewing the actual traces (prompts, tool calls, and responses) from your past invocations.

## [Step 6: View Evaluation History](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/60-lab5-evals#step-6:-view-evaluation-history)

Check past evaluation results and detailed per-interaction scoring:

```bash
agentcore evals history --runtime CustomerSupport --limit 5
```

Example output:

```text
Date                   Agent                Evaluators                                            Sessions
────────────────────────────────────────────────────────────────────────────────────────────────────────
Jul 19, 2026, 09:54 PM CustomerSupport      Builtin.GoalSuccessRate=0.73, Builtin.Correctness=0.84 11

Results saved in: /home/participant/workshop/CustomerSupport/agentcore/.cli/eval-results
```

This is a summary view showing aggregate scores across sessions. The detailed results (per-session breakdowns) are saved locally in `.cli/eval-results` if you want to dig deeper.

View online evaluation logs:

```bash
agentcore logs evals --runtime CustomerSupport --since 30m
```

This is the most detailed view. The output is verbose (raw JSON), but it shows you exactly how the LLM judge scored each interaction and *why*. For example, you'll see entries like:

- **GoalSuccessRate = 1.0** with an explanation like: *"All five user goals were successfully achieved"* followed by a breakdown of each goal.
- **Correctness = 0.5 (Partially Correct)** with reasoning like: *"The assistant invented a specific product identification that was never established in the conversation"* when the agent hallucinated a product ID the user never mentioned.
- **ToolSelectionAccuracy = 1.0** with justification like: *"The user explicitly asked about the return policy for audio products. Calling get_return_policy with 'audio' directly addresses the user's question."*

This is where you go when an aggregate score looks off and you want to understand the root cause. The explanations point directly to what the agent did wrong (or right).

## [Step 7: Pause/Resume Evaluation (Optional)](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/60-lab5-evals#step-7:-pauseresume-evaluation-\(optional\))

You can pause online evaluation to reduce costs or during maintenance:

```bash
# Pause
agentcore pause online-eval QualityMonitor

# Resume
agentcore resume online-eval QualityMonitor
```

## [Architecture](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/60-lab5-evals#architecture)

After completing this lab, your deployed architecture includes continuous evaluation:

![Lab 5 Architecture](images/lab5_architecture_diagram.png)

## [Congratulations! ✅](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/60-lab5-evals#congratulations!)

You're past the halfway mark. Here's what you've built so far:

| Lab | What You Built |
| --- | --- |
| 1 | Agent prototype with local tools |
| 2 | Persistent memory across sessions |
| 3 | Centralized tools via Gateway |
| 4 | Security, observability, and session management |
| 5 | Continuous quality monitoring |

### [What's Next](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/60-lab5-evals#what's-next)

Your agent works, but customers can't reach it yet. They'd need a terminal and a bearer token. In Lab 6, you'll give them a real login page and chat interface.

→ Next: [Lab 6: Building the customer interface](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/70-lab6-frontend/)
