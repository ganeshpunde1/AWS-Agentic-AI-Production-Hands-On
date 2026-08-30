# Lab 4: Securing and Observing in Production

**⏱️ Estimated time: \~15 minutes**

## [Overview](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/50-lab4-deploy#overview)

Your agent is deployed, remembers users, and can call remote tools. By default its endpoint is protected by **AWS IAM** — a caller needs signed AWS credentials with permission to invoke it, so it is *not* open to anyone with the URL. What it doesn't yet have is an **end-user identity** model: it can't tell one customer from another without handing out AWS credentials. In this lab you'll add that, and explore the observability AgentCore already generates for you.

In this lab, you'll lock things down and light things up:

- **Session continuity & isolation** — See how conversation context is scoped per session
- **Observability** — Explore the traces, metrics, and logs that AgentCore Runtime is already generating
- **Runtime management** — Status, logs, and traces via the AgentCore CLI
- **Security** — Add JWT-based authentication (Cognito) to both the Runtime and the Gateway so only authorized callers get through

### [What's already running](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/50-lab4-deploy#what's-already-running)

By this point, your deployed infrastructure includes:

| ResourceStatusCreated In |            |                        |
| ------------------------ | ---------- | ---------------------- |
| AgentCore Runtime        | ✅ READY    | Lab 2 (first deploy)   |
| AgentCore Memory         | ✅ Deployed | Lab 2                  |
| AgentCore Gateway        | ✅ Deployed | Lab 3                  |
| CloudWatch Observability | ✅ Active   | Automatic with Runtime |

## [Step 1: Verify Your Deployment](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/50-lab4-deploy#step-1:-verify-your-deployment)

In the Code Editor terminal, check the status of all deployed resources:

```bash
agentcore status
```

You should see all resources deployed and ready:

```bash
AgentCore Status (target: default, <your-region>)

Agents
  CustomerSupport: Deployed - Runtime: READY (arn:aws:bedrock-agentcore:...)

Memories
  SharedMemory: Deployed (SEMANTIC, SUMMARIZATION) (arn:aws:bedrock-agentcore:...)

Gateways
  my-gateway: Deployed (1 target) (customersupport-my-gateway-...)
```

## [Step 2: Test Session Continuity and Isolation](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/50-lab4-deploy#step-2:-test-session-continuity-and-isolation)

AgentCore Runtime provides built-in session management. You pass a `--session-id` with each invocation, and the Runtime keeps conversation context within that session while keeping different sessions completely isolated.

Each session runs in its own microVM, so a single user stays on the same execution environment for the entire conversation — up to 8 hours, which is the maximum session duration for AgentCore Runtime. In practice, there's a one-to-one mapping between a user and their session. Our memory module (from Lab 2) takes advantage of this — it stores and retrieves memories using the combination of `session_id` and `user_id`.

To see the difference between **session persistence** and **AgentCore Memory** clearly, you'll use a brand-new user ID (`Alex`) that has no prior memory, then compare with the existing user (`Sarah`) from Lab 2 who *does* have stored memories.

### [Part A: Session persistence (same session = remembers)](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/50-lab4-deploy#part-a:-session-persistence-\(same-session-remembers\))

Use a fresh user ID that has never interacted with the agent before:

- **macOS/Linux**
- **Windows**

```bash
SESSION_1=$(python3 -c 'import uuid; print(uuid.uuid4())')
agentcore invoke "My name is Alex and I just bought a Mechanical Keyboard" \
  --session-id $SESSION_1 \
  -H "X-Amzn-Bedrock-AgentCore-Runtime-Custom-User-Id: Alex" \
  --stream
```

Now continue within the same session:

- **macOS/Linux**
- **Windows**

```bash
agentcore invoke "What did I just buy?" \
  --session-id $SESSION_1 \
  -H "X-Amzn-Bedrock-AgentCore-Runtime-Custom-User-Id: Alex" \
  --stream
```

**Expected:** The agent remembers Alex bought a Mechanical Keyboard. This is **session persistence** — the running conversation is kept for that `session-id`. It is *not* AgentCore Memory; it's just the live conversation state in the microVM.

### [Part B: Session isolation (new session, same user = doesn't remember)](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/50-lab4-deploy#part-b:-session-isolation-\(new-session-same-user-doesn't-remember\))

Now start a **new session** for the same user — immediately, before memory aggregation has had time to process:

- **macOS/Linux**
- **Windows**

```bash
SESSION_2=$(python3 -c 'import uuid; print(uuid.uuid4())')
agentcore invoke "What did I just buy?" \
  --session-id $SESSION_2 \
  -H "X-Amzn-Bedrock-AgentCore-Runtime-Custom-User-Id: Alex" \
  --stream
```

**Expected:** The agent does **not** know what Alex bought. This new session has no in-session conversation history — sessions are isolated by `session-id`, each in its own microVM. Since `Alex` is a brand-new user with no prior memories stored, and we haven't waited for memory aggregation, there's nothing to recall. This is pure **session isolation** in action.

### [Part C: Memory recall (new session, existing user = remembers via Memory)](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/50-lab4-deploy#part-c:-memory-recall-\(new-session-existing-user-remembers-via-memory\))

Now let's demonstrate the other side: what happens when a user *does* have stored memories. Use Sarah's user ID from Lab 2 — her facts (name, preferences, purchase history) were aggregated into long-term memory minutes ago:

- **macOS/Linux**
- **Windows**

```bash
SESSION_3=$(python3 -c 'import uuid; print(uuid.uuid4())')
agentcore invoke "Do you know anything about me?" \
  --session-id $SESSION_3 \
  -H "X-Amzn-Bedrock-AgentCore-Runtime-Custom-User-Id: Sarah" \
  --stream
```

**Expected:** Even though this is a brand-new session, the agent knows Sarah's name, her preference for email updates, and her Smart Watch purchase. That's **AgentCore Memory** — durable recall that spans different sessions and survives the runtime being recycled.

**Session persistence vs. AgentCore Memory — the key takeaway**

Two different mechanisms are at play — keep them separate:

| Session Persistence (Runtime)AgentCore Memory |                                                    |                                     |
| --------------------------------------------- | -------------------------------------------------- | ----------------------------------- |
| **Scope**                                     | Within one `session-id`                            | Across all sessions for a `user-id` |
| **Mechanism**                                 | Live conversation state in the microVM             | Aggregated facts stored durably     |
| **Lifetime**                                  | Until session terminates (15 min idle or 8 hr max) | Permanent (until expiry policy)     |
| **Requires setup**                            | No — built into the Runtime                        | Yes — configured in Lab 2           |

Part A showed session persistence. Part B showed session isolation (no memory yet for a new user). Part C showed Memory bridging the gap for an existing user.

When a session terminates (idle timeout or max lifetime), the microVM is destroyed and all in-process state is gone. A subsequent request with the same `session-id` creates a fresh execution environment — it does **not** resume the old one. If your agent needs to survive that boundary, it must use AgentCore Memory: short-term memory to reconstruct recent conversation history, and long-term memory for durable facts and preferences. Without Memory configured, the conversation is lost the moment the session ends.

## [Step 3: Explore Observability](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/50-lab4-deploy#step-3:-explore-observability)

AgentCore Runtime automatically instruments your agent with OpenTelemetry and sends traces to CloudWatch. Every invocation generates traces that capture the full conversation flow.

### [View traces via CLI](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/50-lab4-deploy#view-traces-via-cli)

List recent traces for your agent:

```bash
agentcore traces list --limit 10
```

Your output should look something like this:

```
Traces for CustomerSupport (target: default)

Trace ID                          Timestamp             Session ID
a1b2c3d400e1877800000000000001a1  2026-07-19 20:26:15Z  aaaaaaaa-bbbb-cccc-dddd-eeeeeeee0001
a1b2c3d400e1877800000000000001a2  2026-07-19 20:25:55Z  aaaaaaaa-bbbb-cccc-dddd-eeeeeeee0002
a1b2c3d400e1877800000000000001a3  2026-07-19 20:23:46Z  aaaaaaaa-bbbb-cccc-dddd-eeeeeeee0002
```

Next, if you download the most recent trace for detailed inspection, you should see the last invocation to our agent where you asked `What did I just buy`.

```bash
agentcore traces get <trace-id> --output trace.json
```

The resulting `trace.json` contains the full OpenTelemetry trace for that invocation: every span with timestamps, the user's prompt (`gen_ai.user.message`), the system prompt sent to the model, memory retrieval results (which namespaces were queried, how many records came back), HTTP calls to the Gateway and the final model response (`gen_ai.choice`). It's the complete story of a single request, useful for debugging why a tool wasn't called or why memory returned empty.

### [View logs via CLI](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/50-lab4-deploy#view-logs-via-cli)

Traces show you the structure of a request (what was called, in what order, how long each step took). Logs show you the content: what your agent actually printed, memory retrieval details, and any errors or warnings. Together, they give you the full picture.

In earlier labs you relied on terminal output to see what your agent was doing. Now that it's deployed, those logs live in CloudWatch. The CLI gives you direct access without opening the console:

```bash
agentcore logs
```

This streams live output, so you'll only see new entries when an invocation happens. More general debugging, you can search recent history, for example, to find errors from the last hour:

```bash
agentcore logs --since 1h --level error
```

Alternatively, you can search for a specific keyword across all log entries (like finding every time the warranty tool was called):

```bash
agentcore logs --since 1h --query "warranty"
```

In the output you'll see the full agent reasoning chain for each matching invocation: the `<user_context>` block (memory facts injected before the prompt), the tool call the model chose (`WarrantyCheck___check_warranty` with its input), the tool response (warranty status JSON from the Lambda), and the final assistant message. It's verbose, but it shows you exactly what the model saw and decided at each step.

### [View in CloudWatch console](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/50-lab4-deploy#view-in-cloudwatch-console)

For a visual dashboard, navigate to the [CloudWatch console ](https://console.aws.amazon.com/cloudwatch/):

1. In the left panel, find **GenAI Observability** → **Bedrock AgentCore**
2. Click on **Agents** to see your CustomerSupport agent
3. Click on **Sessions** to see all conversation sessions
4. Click on **Traces** to see detailed request traces

Each trace shows:

- The complete conversation flow (user prompt → tool selection → tool execution → response)
- Latency breakdown for each step
- Memory retrieval and storage operations
- Gateway tool invocations

> ***Note:**** It takes \~10 minutes after the first invocation for traces to appear in CloudWatch. If you enabled Transaction Search in the prerequisites, traces should already be indexed.*

**Observability spans more than the Runtime**

These traces aren't limited to the Runtime. **AgentCore Gateway** and **AgentCore Identity** also emit system-generated telemetry — tool invocations routed through the Gateway, and token exchanges / authorization decisions from Identity — into the same CloudWatch GenAI Observability views. With Transaction Search enabled, you can follow a single request end to end across all three services (client → Runtime → Gateway → tool), which is invaluable once you add JWT auth in the next steps and want to confirm the token is flowing correctly at each hop.

## [Step 4: Secure Your Runtime with Cognito Authentication](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/50-lab4-deploy#step-4:-secure-your-runtime-with-cognito-authentication)

So far, your agent runtime uses the **default IAM authorization**: a caller must sign requests with AWS credentials that are allowed to invoke it, so the endpoint is not open to just anyone with the URL. That's a solid default for service-to-service access, but a customer-facing application needs to authenticate **end users** — without handing them AWS credentials.

AgentCore Runtime supports two authentication modes: **IAM** (the default, which restricts access to callers with the right AWS credentials) and **Custom JWT** (which validates tokens issued by an external identity provider like Cognito, Okta, or any OIDC-compatible service). IAM auth is fine for service-to-service calls, but for user-facing applications you typically want JWT so your app can authenticate end users without granting them AWS credentials.

In this step, you'll switch from the default IAM auth to a Custom JWT authorizer backed by Amazon Cognito. After this, every request must include a valid Cognito access token.

### [Retrieve Cognito configuration from Parameter Store](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/50-lab4-deploy#retrieve-cognito-configuration-from-parameter-store)

The prerequisites CloudFormation stack stored the Cognito configuration in SSM Parameter Store. You'll need three values: the **discovery URL** (tells AgentCore where to fetch token-signing keys), the **client IDs** (which apps are allowed to request tokens), and the **pool ID** (identifies your Cognito user pool for user management commands).

Retrieve them:

- **macOS/Linux**
- **Windows**

```bash
COGNITO_DISCOVERY_URL=$(aws ssm get-parameter \
  --name /app/customersupport/agentcore/cognito_discovery_url \
  --query 'Parameter.Value' --output text)

COGNITO_CLIENT_ID=$(aws ssm get-parameter \
  --name /app/customersupport/agentcore/client_id \
  --query 'Parameter.Value' --output text)

COGNITO_POOL_ID=$(aws ssm get-parameter \
  --name /app/customersupport/agentcore/pool_id \
  --query 'Parameter.Value' --output text)

COGNITO_WEB_CLIENT_ID=$(aws ssm get-parameter \
  --name /app/customersupport/agentcore/web_client_id \
  --query 'Parameter.Value' --output text)

echo "Discovery URL: $COGNITO_DISCOVERY_URL"
echo "Client ID:     $COGNITO_CLIENT_ID"
echo "Pool ID:       $COGNITO_POOL_ID"
echo "Web Client ID: $COGNITO_WEB_CLIENT_ID"
```

### [Update agentcore.json](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/50-lab4-deploy#update-agentcore.json)

Open `agentcore/agentcore.json` in the Code Editor.

In the `runtimes` array, find the `"CustomerSupport"` entry and add `Authorization` to the list of `requestHeaderAllowlist`, `authorizerType` and `authorizerConfiguration` fields:

```json
"runtimes": [
  {
    "name": "CustomerSupport",
    "build": "CodeZip",
    "entrypoint": "main.py",
    "codeLocation": "app/CustomerSupport/",
    "runtimeVersion": "PYTHON_3_13",
    "networkMode": "PUBLIC",
    "protocol": "HTTP",
    "requestHeaderAllowlist": [
      "X-Amzn-Bedrock-AgentCore-Runtime-Custom-User-Id",
      "Authorization"
    ],
    "authorizerType": "CUSTOM_JWT",
    "authorizerConfiguration": {
      "customJwtAuthorizer": {
        "discoveryUrl": "<COGNITO_DISCOVERY_URL value>",
        "allowedClients": ["<COGNITO_CLIENT_ID value>", "<COGNITO_WEB_CLIENT_ID value>"]
      }
    }
  }
]
```

Replace `<COGNITO_DISCOVERY_URL value>`, `<COGNITO_CLIENT_ID value>` and `<COGNITO_WEB_CLIENT_ID value>` with the values you retrieved above. The result should look something like (but with your own values for discoveryUrl and allowedClients):

```json
"requestHeaderAllowlist": [
  "X-Amzn-Bedrock-AgentCore-Runtime-Custom-User-Id",
  "Authorization"
],
"authorizerType": "CUSTOM_JWT",
"authorizerConfiguration": {
  "customJwtAuthorizer": {
    "discoveryUrl": "https://cognito-idp.<region>.amazonaws.com/<region>_AbCdEfGhI/.well-known/openid-configuration",
    "allowedClients": ["1a2b3c4d5e6f7g8h9i0j", "2b3c4d5e6f7g8h9i0j1k"]
  }
}
```

> ***What this does:**** The **`discoveryUrl`** points to the Cognito OIDC discovery endpoint, which tells AgentCore Runtime how to validate incoming JWT tokens (where to fetch the signing keys, the issuer, etc.). The **`allowedClients`** list restricts access to tokens issued for that specific Cognito app client — any token from a different client will be rejected. The **`requestHeaderAllowlist`** will be important for the next session, once we add the Cognito authentication to AgentCore Gateway. It is the header that will allow your agent to propagate the authentication token to your MCP client.*

### [Update agent to take authentication into account](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/50-lab4-deploy#update-agent-to-take-authentication-into-account)

Now that the inbound auth for the agent is set, the agent code needs to get the user\_id information, for memory access, from the brearer token upon successful authentication.

First, add the `PyJWT` library to your project dependencies:

```bash
cd app/CustomerSupport
uv add pyjwt
cd ../..
```

Then update `app/CustomerSupport/main.py` to add the `extract_user_id()` method that extracts the username from the claims information. The method is also backward compatible with respect to the previous labs. It fallsback to `user-id` in custom header if `Authorization` is not present in header. In production scenarios, this is unlikely to happen:

```python
from strands import Agent, tool
from bedrock_agentcore.runtime import BedrockAgentCoreApp
from model.load import load_model
from mcp_client.client import get_streamable_http_mcp_client, get_gateway_mcp_client
from memory.session import get_memory_session_manager
import jwt

app = BedrockAgentCoreApp()
log = app.logger

# MCP clients: Exa AI (web search) + AgentCore Gateway (Lambda tools)
mcp_clients = [get_streamable_http_mcp_client(), get_gateway_mcp_client()]

SYSTEM_PROMPT="""You are a helpful and professional customer support assistant for an e-commerce company.
Your role is to:
- Provide accurate information using the tools available to you
- Be friendly, patient, and understanding with customers
- Always offer additional help after answering questions
- If you can't help with something, direct customers to the appropriate contact

You have access to tools for looking up return policies, searching product information, and more.
Additional tools may be available at runtime — always check your full tool list and use the most appropriate tool for each customer request.
Always use tools to get accurate, up-to-date information rather than guessing."""

# --- Customer Support Tools ---

RETURN_POLICIES = {
    "electronics": {"window": "30 days", "condition": "Original packaging required, must be unused or defective", "refund": "Full refund to original payment method"},
    "accessories": {"window": "14 days", "condition": "Must be in original packaging, unused", "refund": "Store credit or exchange"},
    "audio": {"window": "30 days", "condition": "Defective items only after 15 days", "refund": "Full refund within 15 days, replacement after"},
}

PRODUCTS = {
    "PROD-001": {"name": "Wireless Headphones", "price": 79.99, "category": "audio", "description": "Noise-cancelling Bluetooth headphones with 30h battery life", "warranty_months": 12},
    "PROD-002": {"name": "Smart Watch", "price": 249.99, "category": "electronics", "description": "Fitness tracker with heart rate monitor, GPS, and 5-day battery", "warranty_months": 24},
    "PROD-003": {"name": "Laptop Stand", "price": 39.99, "category": "accessories", "description": "Adjustable aluminum laptop stand for ergonomic desk setup", "warranty_months": 6},
    "PROD-004": {"name": "USB-C Hub", "price": 54.99, "category": "accessories", "description": "7-in-1 USB-C hub with HDMI, USB-A, SD card reader, and ethernet", "warranty_months": 12},
    "PROD-005": {"name": "Mechanical Keyboard", "price": 129.99, "category": "electronics", "description": "RGB mechanical keyboard with Cherry MX switches", "warranty_months": 24},
}

@tool
def get_return_policy(product_category: str) -> str:
    """Get return policy information for a specific product category.

    Args:
        product_category: Product category (e.g., 'electronics', 'accessories', 'audio')

    Returns:
        Formatted return policy details including timeframes and conditions
    """
    category = product_category.lower()
    if category in RETURN_POLICIES:
        policy = RETURN_POLICIES[category]
        return f"Return policy for {category}: Window: {policy['window']}, Condition: {policy['condition']}, Refund: {policy['refund']}"
    return f"No specific return policy found for '{product_category}'. Please contact support for details."

@tool
def get_product_info(query: str) -> str:
    """Search for product information by name, ID, or keyword.

    Args:
        query: Product name, ID (e.g., 'PROD-001'), or search keyword

    Returns:
        Product details including name, price, category, and description
    """
    query_lower = query.lower()
    # Search by ID
    if query.upper() in PRODUCTS:
        p = PRODUCTS[query.upper()]
        return f"{p['name']} ({query.upper()}): ${p['price']}, Category: {p['category']}, {p['description']}, Warranty: {p['warranty_months']} months"
    # Search by keyword
    results = [f"{pid}: {p['name']} - ${p['price']} - {p['description']}" for pid, p in PRODUCTS.items()
               if query_lower in p['name'].lower() or query_lower in p['description'].lower() or query_lower in p['category'].lower()]
    if results:
        return "Found products:\n" + "\n".join(results)
    return f"No products found matching '{query}'."

tools = [get_return_policy, get_product_info]

# Add MCP client (Exa AI web search) to tools
for mcp_client in mcp_clients:
    if mcp_client:
        tools.append(mcp_client)

# --- Agent Setup ---

_agent = None

def get_or_create_agent(session_id, user_id):
    global _agent
    if _agent is None:
        _agent = Agent(
            model=load_model(),
            session_manager=get_memory_session_manager(session_id, user_id),
            system_prompt=SYSTEM_PROMPT,
            tools=tools
        )
    return _agent

def extract_user_id(context) -> str | None:
    """Extract user_id from JWT bearer token (username claim) or fall back to custom header."""
    headers = context.request_headers or {}

    # Try Authorization header first (Bearer JWT)
    auth_header = headers.get("Authorization") or headers.get("authorization")
    if auth_header and auth_header.startswith("Bearer "):
        try:
            token = auth_header.split(" ", 1)[1]
            claims = jwt.decode(token, options={"verify_signature": False})
            username = claims.get("username")
            if username:
                return username
        except Exception as e:
            log.warning(f"Failed to decode JWT for user_id: {e}")
    else:
        log.info(f"No Bearer token found. Auth header present: {auth_header is not None}")

    # Fall back to custom header
    return headers.get("x-amzn-bedrock-agentcore-runtime-custom-user-id")

@app.entrypoint
async def invoke(payload, context):
    log.info("Invoking Agent.....")

    session_id = context.session_id
    user_id = extract_user_id(context)

    if not session_id or not user_id:
        raise ValueError("session_id and user_id are required. Pass --session-id and --user-id when invoking.")

    agent = get_or_create_agent(session_id, user_id)
    stream = agent.stream_async(payload.get("prompt"))
    async for event in stream:
        if "data" in event and isinstance(event["data"], str):
            yield event["data"]

if __name__ == "__main__":
    app.run()
```

**Want help understanding this code?**

Run `claude` in your terminal and ask it to explain the `extract_user_id()` function or any other part of the code you'd like to understand better.

### [Validate and deploy the updated configuration](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/50-lab4-deploy#validate-and-deploy-the-updated-configuration)

Before deploying, validate that your `agentcore.json` changes are correct:

```bash
agentcore validate
```

If validation passes, you'll see a success message. If there are issues (e.g., a missing comma, invalid JSON, or a malformed discovery URL), the CLI will tell you exactly what's wrong so you can fix it before deploying.

```bash
agentcore deploy -y -v
```

### [Test with authentication](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/50-lab4-deploy#test-with-authentication)

After deployment, unauthenticated requests will be rejected. You need to create a Cognito user and obtain a token first.

The prerequisites stack created two Cognito app clients: a machine client (for `client_credentials`) and a web client (for user-based flows). You'll use the web client here to authenticate as a real user.

**Create a test user in the Cognito User Pool:**

- **macOS/Linux**
- **Windows**

```bash
aws cognito-idp admin-create-user \
  --user-pool-id $COGNITO_POOL_ID \
  --username workshopuser@example.com \
  --temporary-password 'TempPass1!' \
  --user-attributes Name=email,Value=workshopuser@example.com Name=email_verified,Value=true \
  --message-action SUPPRESS \
  --no-cli-pager

# Set a permanent password so the user is confirmed and ready to use
aws cognito-idp admin-set-user-password \
  --user-pool-id $COGNITO_POOL_ID \
  --username workshopuser@example.com \
  --password 'WorkshopPass1!' \
  --permanent \
  --no-cli-pager

echo "User 'workshopuser@example.com' created and confirmed"
```

> ***Note:**** The User Pool is configured with email as the username attribute, so the username must be a valid email address.*

**Authenticate as the user and obtain a token:**

- **macOS/Linux**
- **Windows**

```bash
TOKEN=$(aws cognito-idp initiate-auth \
  --auth-flow USER_PASSWORD_AUTH \
  --client-id $COGNITO_WEB_CLIENT_ID \
  --auth-parameters USERNAME=workshopuser@example.com,PASSWORD='WorkshopPass1!' \
  --query 'AuthenticationResult.AccessToken' --output text)

echo "Token obtained successfully"
```

Now invoke the agent with the bearer token:

- **macOS/Linux**
- **Windows**

```bash
SESSION_3=$(python3 -c 'import uuid; print(uuid.uuid4())')
agentcore invoke "What's the return policy for electronics?" \
  --session-id $SESSION_3 --bearer-token "$TOKEN" --stream
```

Try without the token to confirm it's rejected:

- **macOS/Linux**
- **Windows**

```bash
agentcore invoke "What's the return policy for electronics?" \
  --session-id $SESSION_3 --stream --json
```

You should see an authentication error — your runtime is now secured.

## [Step 5: Secure Your Gateway with Cognito Authentication](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/50-lab4-deploy#step-5:-secure-your-gateway-with-cognito-authentication)

You've secured the runtime endpoint, but the AgentCore Gateway also accepts requests independently. To fully lock down your application, you should apply the same JWT authentication to the Gateway so that only authorized agents (or clients) can invoke the tools behind it.

> ***Why apply the same JWT authorizer to the Gateway?**** The runtime and the Gateway are separate endpoints, each with its own authorization. Like the runtime, the Gateway isn't open — by default it's protected by IAM, which is a perfectly legitimate choice (an IAM-protected Gateway is a common, valid pattern for service-to-service tool access). What we want here is a single ****end-user identity**** model: the same Cognito token authenticates the client to the runtime, and the runtime propagates that token to the Gateway. Using one JWT authorizer across both hops means the end user's identity — not just an AWS principal — flows consistently from client → runtime → Gateway, which is what a customer-facing app needs.*

Gateway authorizer configuration can't be updated in-place, so we need to remove the existing gateway, deploy the removal, and then recreate it with authentication enabled.

### [Remove the existing gateway](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/50-lab4-deploy#remove-the-existing-gateway)

```bash
agentcore remove gateway --name my-gateway -y
```

### [Recreate the gateway with JWT authentication](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/50-lab4-deploy#recreate-the-gateway-with-jwt-authentication)

Now create a new gateway with the Cognito JWT authorizer configured from the start. We'll use the same SSM parameter values retrieved in Step 4:

- **macOS/Linux**
- **Windows**

```bash
agentcore add gateway --name my-gateway-secure --runtimes CustomerSupport \
  --authorizer-type CUSTOM_JWT \
  --discovery-url $COGNITO_DISCOVERY_URL \
  --allowed-clients $COGNITO_CLIENT_ID,$COGNITO_WEB_CLIENT_ID
```

### [Re-add the warranty check target](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/50-lab4-deploy#re-add-the-warranty-check-target)

The Lambda ARN should still be in your shell from Lab 3. If not, retrieve it again:

- **macOS/Linux**
- **Windows**

```bash
WARRANTY_LAMBDA_ARN=$(aws ssm get-parameter \
  --name /app/customersupport/agentcore/warranty_check_lambda_arn \
  --query 'Parameter.Value' --output text)
```

Add the target to the new gateway:

- **macOS/Linux**
- **Windows**

```bash
agentcore add gateway-target \
  --type lambda-function-arn \
  --name WarrantyCheck \
  --lambda-arn $WARRANTY_LAMBDA_ARN \
  --tool-schema-file app/CustomerSupport/tool/warranty_schema.json \
  --gateway my-gateway-secure
```

> ***Note:**** The new gateway name is **`my-gateway-secure`**, so the injected environment variable will be **`AGENTCORE_GATEWAY_MY_GATEWAY_SECURE_URL`**. You also need to configure the Authorization header of your MCPClient. You need to update **`app/CustomerSupport/mcp_client/client.py`** to read the new variable name and pass the authorization header:*

Open `app/CustomerSupport/mcp_client/client.py` and change the environment variable name:

**What changed from Lab 3**

Two changes: (1) The gateway URL environment variable changed from `AGENTCORE_GATEWAY_MY_GATEWAY_URL` to `AGENTCORE_GATEWAY_MY_GATEWAY_SECURE_URL` to match the new secured gateway name. (2) The `get_gateway_mcp_client` function now accepts an `auth_header` parameter and passes it as an `Authorization` header to the gateway, so the JWT token from the runtime is forwarded to authenticate gateway requests.

```python
import os
import logging
from mcp.client.streamable_http import streamablehttp_client
from strands.tools.mcp.mcp_client import MCPClient

logger = logging.getLogger(__name__)

# ExaAI MCP endpoint for web search
EXAMPLE_MCP_ENDPOINT = "https://mcp.exa.ai/mcp"


def get_streamable_http_mcp_client() -> MCPClient:
    """Returns an MCP Client for Exa AI web search"""
    return MCPClient(lambda: streamablehttp_client(EXAMPLE_MCP_ENDPOINT))


def get_gateway_mcp_client(auth_header: str) -> MCPClient | None:
    """Returns an MCP Client for AgentCore Gateway, if configured"""
    url = os.environ.get("AGENTCORE_GATEWAY_MY_GATEWAY_SECURE_URL")
    if not url:
        logger.warning("Gateway URL not set — gateway tools unavailable")
        return None
    return MCPClient(lambda: streamablehttp_client(
        url=url,
        headers={"Authorization": auth_header}
    ))
```

Since we are using the same Cognito client for AgentCore Runtime and AgentCore Gateway, you will need to update your `app/CustomerSupport/main.py` file to pass the authorization header to your gateway and let's use the Cognito username as the actor id for the agent memory:

**What changed from Lab 3**

The `invoke` function now extracts the `Authorization` header from the incoming request context and passes it to `get_gateway_mcp_client`. This propagates the caller's JWT token from the Runtime to the Gateway, enabling end-to-end authentication. The gateway MCP client is now created per-request (inside `invoke`) instead of at startup, since each request may carry a different token.

```python
from strands import Agent, tool
from bedrock_agentcore.runtime import BedrockAgentCoreApp
from model.load import load_model
from mcp_client.client import get_streamable_http_mcp_client, get_gateway_mcp_client
from memory.session import get_memory_session_manager
import jwt

app = BedrockAgentCoreApp()
log = app.logger

SYSTEM_PROMPT="""You are a helpful and professional customer support assistant for an e-commerce company.
Your role is to:
- Provide accurate information using the tools available to you
- Be friendly, patient, and understanding with customers
- Always offer additional help after answering questions
- If you can't help with something, direct customers to the appropriate contact

You have access to tools for looking up return policies, searching product information, and more.
Additional tools may be available at runtime — always check your full tool list and use the most appropriate tool for each customer request.
Always use tools to get accurate, up-to-date information rather than guessing."""

# --- Customer Support Tools ---

RETURN_POLICIES = {
    "electronics": {"window": "30 days", "condition": "Original packaging required, must be unused or defective", "refund": "Full refund to original payment method"},
    "accessories": {"window": "14 days", "condition": "Must be in original packaging, unused", "refund": "Store credit or exchange"},
    "audio": {"window": "30 days", "condition": "Defective items only after 15 days", "refund": "Full refund within 15 days, replacement after"},
}

PRODUCTS = {
    "PROD-001": {"name": "Wireless Headphones", "price": 79.99, "category": "audio", "description": "Noise-cancelling Bluetooth headphones with 30h battery life", "warranty_months": 12},
    "PROD-002": {"name": "Smart Watch", "price": 249.99, "category": "electronics", "description": "Fitness tracker with heart rate monitor, GPS, and 5-day battery", "warranty_months": 24},
    "PROD-003": {"name": "Laptop Stand", "price": 39.99, "category": "accessories", "description": "Adjustable aluminum laptop stand for ergonomic desk setup", "warranty_months": 6},
    "PROD-004": {"name": "USB-C Hub", "price": 54.99, "category": "accessories", "description": "7-in-1 USB-C hub with HDMI, USB-A, SD card reader, and ethernet", "warranty_months": 12},
    "PROD-005": {"name": "Mechanical Keyboard", "price": 129.99, "category": "electronics", "description": "RGB mechanical keyboard with Cherry MX switches", "warranty_months": 24},
}

@tool
def get_return_policy(product_category: str) -> str:
    """Get return policy information for a specific product category.

    Args:
        product_category: Product category (e.g., 'electronics', 'accessories', 'audio')

    Returns:
        Formatted return policy details including timeframes and conditions
    """
    category = product_category.lower()
    if category in RETURN_POLICIES:
        policy = RETURN_POLICIES[category]
        return f"Return policy for {category}: Window: {policy['window']}, Condition: {policy['condition']}, Refund: {policy['refund']}"
    return f"No specific return policy found for '{product_category}'. Please contact support for details."

@tool
def get_product_info(query: str) -> str:
    """Search for product information by name, ID, or keyword.

    Args:
        query: Product name, ID (e.g., 'PROD-001'), or search keyword

    Returns:
        Product details including name, price, category, and description
    """
    query_lower = query.lower()
    # Search by ID
    if query.upper() in PRODUCTS:
        p = PRODUCTS[query.upper()]
        return f"{p['name']} ({query.upper()}): ${p['price']}, Category: {p['category']}, {p['description']}, Warranty: {p['warranty_months']} months"
    # Search by keyword
    results = [f"{pid}: {p['name']} - ${p['price']} - {p['description']}" for pid, p in PRODUCTS.items()
               if query_lower in p['name'].lower() or query_lower in p['description'].lower() or query_lower in p['category'].lower()]
    if results:
        return "Found products:\n" + "\n".join(results)
    return f"No products found matching '{query}'."

# --- Agent Setup ---

_agent = None

def get_or_create_agent(session_id, user_id, auth_header):
    global _agent

    session_manager = get_memory_session_manager(session_id, user_id)

    # MCP clients: Exa AI (web search) + AgentCore Gateway (Lambda tools)
    mcp_clients = [get_streamable_http_mcp_client(), get_gateway_mcp_client(auth_header)]
    tools = [get_return_policy, get_product_info]

    # Add MCP client (Exa AI web search) to tools
    for mcp_client in mcp_clients:
        if mcp_client:
            tools.append(mcp_client)

    if _agent is None:
        _agent = Agent(
            model=load_model(),
            session_manager=session_manager,
            system_prompt=SYSTEM_PROMPT,
            tools=tools
        )
    return _agent

def extract_user_id(auth_header) -> str | None:
    """Extract user_id from JWT bearer token (username claim) or fall back to custom header."""

    if auth_header and auth_header.startswith("Bearer "):
        try:
            token = auth_header.split(" ", 1)[1]
            claims = jwt.decode(token, options={"verify_signature": False})
            username = claims.get("username")
            if username:
                return username
        except Exception as e:
            log.warning(f"Failed to decode JWT for user_id: {e}")
    else:
        log.info(f"No Bearer token found. Auth header present: {auth_header is not None}")
        raise Exception("No authorization header")

@app.entrypoint
async def invoke(payload, context):
    log.info("Invoking Agent.....")

    session_id = context.session_id
    request_headers = context.request_headers

    # Access request headers - handle None case
    request_headers = context.request_headers or {}

    # Get Client JWT token
    auth_header = request_headers.get('Authorization', '')

    if not auth_header:
        raise Exception("No authorization header")

    user_id = extract_user_id(auth_header)

    if not session_id or not user_id:
        raise ValueError("session_id and user_id are required. Pass --session-id and --user-id when invoking.")

    agent = get_or_create_agent(session_id, user_id, auth_header)
    stream = agent.stream_async(payload.get("prompt"))
    async for event in stream:
        if "data" in event and isinstance(event["data"], str):
            yield event["data"]

if __name__ == "__main__":
    app.run()
```

### [Validate and deploy](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/50-lab4-deploy#validate-and-deploy)

```bash
agentcore validate
```

```bash
agentcore deploy -y -v
```

### [Test the secured gateway](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/50-lab4-deploy#test-the-secured-gateway)

The warranty check tool goes through the Gateway. If the Gateway auth is working, this should succeed with a valid token:

- **macOS/Linux**
- **Windows**

```bash
SESSION_E=$(python3 -c 'import uuid; print(uuid.uuid4())')
agentcore invoke "Check the warranty for PROD-001" \
  --session-id $SESSION_E --bearer-token "$TOKEN" --stream
```

The agent should return the warranty details for PROD-001 (Wireless Headphones, active, expires 2027-03-01). Both the runtime and the Gateway are now secured with the same Cognito identity provider.

## [Step 6: Generate Test Traffic](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/50-lab4-deploy#step-6:-generate-test-traffic)

Let's generate some varied interactions to populate the observability dashboard. Since the runtime is now secured, include the bearer token:

- **macOS/Linux**
- **Windows**

```bash
SESSION_D=$(python3 -c 'import uuid; print(uuid.uuid4())')

agentcore invoke "What's the return policy for accessories?" \
  --session-id $SESSION_D --bearer-token "$TOKEN" --stream

agentcore invoke "Tell me about the USB-C Hub" \
  --session-id $SESSION_D --bearer-token "$TOKEN" --stream

agentcore invoke "Check the warranty for PROD-002" \
  --session-id $SESSION_D --bearer-token "$TOKEN" --stream

agentcore invoke "Do you remember my name?" \
  --session-id $SESSION_D --bearer-token "$TOKEN" --stream
```

After running these, wait a few minutes and check the CloudWatch dashboard to see the traces for each interaction, including which tools were called and how long each step took.

**Token expired?**

The Cognito access token is valid for 60 minutes (as configured in the prerequisites stack). If you get an authentication error after some time, just re-run the token command from Step 4:

- **macOS/Linux**
- **Windows**

```bash
TOKEN=$(aws cognito-idp initiate-auth \
  --auth-flow USER_PASSWORD_AUTH \
  --client-id $COGNITO_WEB_CLIENT_ID \
  --auth-parameters USERNAME=workshopuser@example.com,PASSWORD='WorkshopPass1!' \
  --query 'AuthenticationResult.AccessToken' --output text)
```

If you've started a new terminal session and lost the environment variables, re-run the full retrieval block from Step 4 ("Retrieve Cognito configuration" and "Test with authentication") to set them again.

## [Architecture](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/50-lab4-deploy#architecture)

After completing this lab, your deployed architecture looks like this:

![Lab 4 Secured and Observable AgentCore Architecture](lab4_architecture_diagram.png)

> ***Observability still works as before.**** Adding JWT authentication doesn't change how traces, logs, and metrics are collected. AgentCore Runtime continues to instrument every invocation with OpenTelemetry automatically. The only difference is that unauthenticated requests are now rejected before they reach your agent code, so you'll only see traces for legitimate, authorized calls.*

## [What Just Happened?](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/50-lab4-deploy#what-just-happened)

You secured both endpoints (Runtime + Gateway) with the same Cognito JWT authorizer, updated your agent code to extract the user identity from the token, and confirmed that unauthenticated requests are rejected. Meanwhile, AgentCore Runtime has been generating traces and metrics for every invocation automatically via OpenTelemetry.

## [Congratulations! ✅](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/50-lab4-deploy#congratulations!)

Your agent is secured with JWT authentication, observable via CloudWatch, and session-isolated. Only authorized callers can reach it, and you can trace every request end-to-end.

### [What's Next](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/50-lab4-deploy#what's-next)

Your agent is secure and observable, but how do you know if it's actually giving *good* answers? Is it picking the right tools? Are customers getting their problems solved?

In Lab 5, you'll set up continuous evaluation to measure that automatically.
