# Lab 6: Building the Customer Interface

**⏱️ Estimated time: ~20 minutes**

## [Overview](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/70-lab6-frontend#overview)

Your agent is deployed, monitored, and evaluated, but customers need a way to interact with it that doesn't involve a terminal and a bearer token.

In this lab, you'll build a web chat interface using Flask that connects to your deployed AgentCore Runtime. The server authenticates against Cognito on startup using the same `USER_PASSWORD_AUTH` flow from Lab 4 and serves a chat page where the browser talks directly to the AgentCore REST API.

```text
User opens the frontend (via Code Editor port forwarding)
    ↓
Flask authenticates against Cognito (USER_PASSWORD_AUTH)
    ↓
Flask serves HTML page with access token + runtime ARN injected
    ↓
User types message in chat
    ↓
Browser calls AgentCore REST API directly with Authorization: Bearer header
    ↓
AgentCore Runtime validates JWT and processes the request
    ↓
Agent uses tools (local + Gateway) and memory
    ↓
Streaming response returned to browser
```

## [Step 1: Install Dependencies](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/70-lab6-frontend#step-1:-install-dependencies)

In your terminal, add Flask, boto3, and requests to your project:

```bash
cd app/CustomerSupport
uv add flask boto3 requests
cd ../..
```

## [Step 2: Create the Flask Server](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/70-lab6-frontend#step-2:-create-the-flask-server)

Create the frontend directory structure.

### macOS/Linux

```bash
mkdir -p app/CustomerSupport/frontend/templates
touch app/CustomerSupport/frontend/__init__.py
touch app/CustomerSupport/frontend/frontend.py
touch app/CustomerSupport/frontend/templates/index.html
```

### Windows PowerShell

```powershell
New-Item -ItemType Directory -Force app/CustomerSupport/frontend/templates

New-Item -ItemType File -Force app/CustomerSupport/frontend/__init__.py
New-Item -ItemType File -Force app/CustomerSupport/frontend/frontend.py
New-Item -ItemType File -Force app/CustomerSupport/frontend/templates/index.html
```

Open:

```text
app/CustomerSupport/frontend/frontend.py
```

and paste the frontend server code from the workshop.
```bash
import json
import uuid
import urllib.parse
from pathlib import Path
from flask import Flask, render_template
import boto3

app = Flask(__name__)

# The Workshop Studio Code Editor exposes local ports through CloudFront.
# Disable origin caching so proxies that honor origin headers request fresh
# content while participants iterate on the frontend.
app.config["SEND_FILE_MAX_AGE_DEFAULT"] = 0


@app.after_request
def disable_cache(response):
    """Prevent browsers and compatible proxies from caching development output."""
    response.headers["Cache-Control"] = "no-store, no-cache, must-revalidate, max-age=0"
    response.headers["Pragma"] = "no-cache"
    response.headers["Expires"] = "0"
    return response


REGION = boto3.session.Session().region_name
if not REGION:
    raise RuntimeError("No AWS region configured. Set AWS_REGION or run 'aws configure'.")
ssm_client = boto3.client("ssm", region_name=REGION)
cognito_client = boto3.client("cognito-idp", region_name=REGION)
AGENTCORE_ENDPOINT = f"https://bedrock-agentcore.{REGION}.amazonaws.com"

# Workshop user credentials (created in Lab 4)
WORKSHOP_USER = "workshopuser@example.com"
WORKSHOP_PASS = "WorkshopPass1!"


def get_runtime_arn():
    state_file = Path(__file__).parent.parent.parent.parent / "agentcore" / ".cli" / "deployed-state.json"
    try:
        state = json.loads(state_file.read_text())
        runtimes = state.get("targets", {}).get("default", {}).get("resources", {}).get("runtimes", {})
        return runtimes.get("CustomerSupport", {}).get("runtimeArn", None)
    except Exception:
        pass
    return None

def get_ssm_param(name):
    return ssm_client.get_parameter(Name=name)["Parameter"]["Value"]

def get_access_token():
    """Authenticate against Cognito and return an access token."""
    try:
        client_id = get_ssm_param("/app/customersupport/agentcore/web_client_id")
        resp = cognito_client.initiate_auth(
            AuthFlow="USER_PASSWORD_AUTH",
            ClientId=client_id,
            AuthParameters={"USERNAME": WORKSHOP_USER, "PASSWORD": WORKSHOP_PASS},
        )
        token = resp["AuthenticationResult"]["AccessToken"]
        print(f"✅ Authenticated as {WORKSHOP_USER}")
        return token
    except Exception as e:
        print(f"❌ Authentication failed: {e}")
        return None


@app.route("/")
def index():
    token = get_access_token()
    if not token:
        return "<h2>Authentication failed. Check workshop user credentials and Cognito configuration.</h2>", 500
    runtime_arn = get_runtime_arn() or "NOT_DEPLOYED"
    return render_template("index.html",
                           token=token,
                           runtime_arn=runtime_arn,
                           region=REGION,
                           endpoint=AGENTCORE_ENDPOINT,
                           username=WORKSHOP_USER)

if __name__ == "__main__":
    print(f"Runtime ARN: {get_runtime_arn() or 'NOT FOUND'}")
    # Verify auth works on startup
    get_access_token()
    app.run(host="0.0.0.0", port=8501)
```
> **Why not boto3?**  
> boto3's `invoke_agent_runtime` uses IAM (SigV4) authentication. When your runtime is configured for Custom JWT authentication, you need to pass an `Authorization: Bearer` header directly, which boto3 doesn't support for this operation. Instead, use the `requests` library to call the AgentCore REST API, as shown in the [AWS documentation](https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/runtime-oauth.html).

### Production Authentication

For production, authenticate each end user with Cognito's Authorization Code flow with PKCE, or use a backend-for-frontend that keeps user tokens server-side.

Do not use hardcoded credentials or shared tokens in a production application.

## [Step 3: Create the Chat UI](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/70-lab6-frontend#step-3:-create-the-chat-ui)

Open:

```text
app/CustomerSupport/frontend/templates/index.html
```

and paste the HTML code from the workshop.
```text
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Customer Support Agent</title>
<style>
* { margin: 0; padding: 0; box-sizing: border-box; }
body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  background: linear-gradient(180deg, #d6eaf8 0%, #ebf0f5 40%, #f5f7fa 100%);
  height: 100vh;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}
.container {
  max-width: 800px;
  width: 100%;
  margin: 0 auto;
  padding: 20px;
  flex: 1;
  max-height: 100%;
  display: flex;
  flex-direction: column;
}
.header {
  text-align: center;
  padding: 60px 0 30px;
}
.header h1 { font-size: 28px; font-weight: 600; color: #1a1a2e; }
.status {
  text-align: center;
  font-size: 12px;
  color: #888;
  padding: 8px 0;
}
.status .connected { color: #27ae60; }
.status .btn-new {
  padding: 4px 12px;
  border-radius: 8px;
  border: 1px solid #ddd;
  background: white;
  font-size: 11px;
  cursor: pointer;
  color: #555;
  margin-left: 8px;
}
.status .btn-new:hover { border-color: #1a1a2e; }
.chat-area {
  flex: 1;
  overflow-y: auto;
  padding: 20px 0;
  min-height: 0;
}
.message {
  margin-bottom: 16px;
  display: flex;
  gap: 10px;
  animation: fadeIn 0.3s ease;
}
@keyframes fadeIn { from { opacity: 0; transform: translateY(8px); } to { opacity: 1; transform: translateY(0); } }
.message.user { justify-content: flex-end; }
.message .bubble {
  max-width: 75%;
  padding: 12px 16px;
  border-radius: 18px;
  font-size: 14px;
  line-height: 1.6;
  white-space: pre-wrap;
}
.message.user .bubble {
  background: #1a1a2e;
  color: white;
  border-bottom-right-radius: 4px;
}
.message.assistant .bubble {
  background: white;
  color: #1a1a2e;
  border-bottom-left-radius: 4px;
  box-shadow: 0 1px 3px rgba(0,0,0,0.08);
}
.message.assistant .avatar {
  width: 32px; height: 32px;
  background: linear-gradient(135deg, #667eea, #764ba2);
  border-radius: 50%;
  display: flex; align-items: center; justify-content: center;
  color: white; font-size: 14px; flex-shrink: 0;
}
.thinking { display: flex; gap: 4px; padding: 4px 0; }
.thinking span {
  width: 8px; height: 8px; background: #aaa; border-radius: 50%;
  animation: bounce 1.4s infinite;
}
.thinking span:nth-child(2) { animation-delay: 0.2s; }
.thinking span:nth-child(3) { animation-delay: 0.4s; }
@keyframes bounce { 0%,80%,100% { transform: scale(0.6); } 40% { transform: scale(1); } }
.input-area { padding: 20px 0; flex-shrink: 0; }
.input-bar {
  display: flex; align-items: center;
  background: white; border-radius: 28px;
  padding: 6px 6px 6px 20px;
  box-shadow: 0 2px 12px rgba(0,0,0,0.08);
  border: 1px solid #e8e8e8;
}
.input-bar input {
  flex: 1; border: none; outline: none;
  font-size: 15px; background: transparent; color: #1a1a2e;
}
.input-bar input::placeholder { color: #aaa; }
.input-bar button {
  width: 40px; height: 40px; border-radius: 50%; border: none;
  background: #1a1a2e; color: white; cursor: pointer;
  display: flex; align-items: center; justify-content: center;
  transition: background 0.2s;
}
.input-bar button:hover { background: #2d2d4e; }
.input-bar button:disabled { background: #ccc; cursor: default; }
.quick-actions {
  display: flex; gap: 8px; justify-content: center;
  margin-top: 12px; flex-wrap: wrap;
}
.quick-actions button {
  padding: 8px 16px; border-radius: 20px;
  border: 1px solid #ddd; background: white;
  color: #555; font-size: 13px; cursor: pointer;
  transition: all 0.2s;
}
.quick-actions button:hover { border-color: #1a1a2e; color: #1a1a2e; }
</style>
</head>
<body>
<div class="container">
  <div class="status">
    <span class="connected">● Connected</span> — {{ runtime_arn }}
    <button class="btn-new" onclick="newSession()">🔄 New Session</button>
    <span>� {{ username }}</span>
    <span id="sessionLabel"></span>
  </div>
  <div class="header" id="header">
    <h1>👋 How can I help you today?</h1>
  </div>
  <div class="chat-area" id="chatArea"></div>
  <div class="input-area">
    <div class="input-bar">
      <input id="msgInput" type="text" placeholder="Ask your customer support agent..." onkeydown="if(event.key==='Enter')sendMsg()" autofocus />
      <button onclick="sendMsg()" id="sendBtn">↑</button>
    </div>
    <div class="quick-actions" id="quickActions">
      <button onclick="quickSend('What products do you have?')">🛒 Products</button>
      <button onclick="quickSend('What is the return policy for electronics?')">↩️ Returns</button>
      <button onclick="quickSend('Check warranty for PROD-002')">🛡️ Warranty</button>
      <button onclick="quickSend('Do you remember me?')">🧠 Memory</button>
    </div>
  </div>
</div>
<script>
// Injected by Flask server
const TOKEN = "{{ token }}";
const RUNTIME_ARN = "{{ runtime_arn }}";
const ENDPOINT = "{{ endpoint }}";

let sessionId = crypto.randomUUID();
document.getElementById('sessionLabel').textContent = '  Session: ' + sessionId;

function newSession() {
  sessionId = crypto.randomUUID();
  document.getElementById('sessionLabel').textContent = '  Session: ' + sessionId;
  document.getElementById('chatArea').innerHTML = '';
  document.getElementById('quickActions').style.display = 'flex';
  document.getElementById('header').style.display = 'block';
}

function addMessage(role, text) {
  const chat = document.getElementById('chatArea');
  const div = document.createElement('div');
  div.className = 'message ' + role;
  if (role === 'assistant') {
    div.innerHTML = '<div class="avatar">🤖</div><div class="bubble">' + text.replace(/\n/g, '<br>') + '</div>';
  } else {
    div.innerHTML = '<div class="bubble">' + text + '</div>';
  }
  chat.appendChild(div);
  chat.scrollTop = chat.scrollHeight;
}

function showThinking() {
  const chat = document.getElementById('chatArea');
  const div = document.createElement('div');
  div.className = 'message assistant'; div.id = 'thinking';
  div.innerHTML = '<div class="avatar">🤖</div><div class="bubble"><div class="thinking"><span></span><span></span><span></span></div></div>';
  chat.appendChild(div);
  chat.scrollTop = chat.scrollHeight;
}

function removeThinking() { const el = document.getElementById('thinking'); if (el) el.remove(); }

async function sendMsg() {
  const input = document.getElementById('msgInput');
  const msg = input.value.trim();
  if (!msg) return;
  input.value = '';
  document.getElementById('quickActions').style.display = 'none';
  document.getElementById('header').style.display = 'none';
  addMessage('user', msg);
  showThinking();
  document.getElementById('sendBtn').disabled = true;
  try {
    const escapedArn = encodeURIComponent(RUNTIME_ARN);
    const url = `${ENDPOINT}/runtimes/${escapedArn}/invocations?qualifier=DEFAULT`;
    const resp = await fetch(url, {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${TOKEN}`,
        'Content-Type': 'application/json',
        'X-Amzn-Bedrock-AgentCore-Runtime-Session-Id': sessionId
      },
      body: JSON.stringify({prompt: msg})
    });
    const text = await resp.text();
    // Parse SSE-style streaming response
    let fullResponse = '';
    for (const line of text.split('\n')) {
      if (line.startsWith('data: ')) {
        let chunk = line.slice(6).trim();
        if (chunk) {
          if (chunk.startsWith('"') && chunk.endsWith('"')) {
            chunk = JSON.parse(chunk);
          }
          fullResponse += chunk;
        }
      }
    }
    removeThinking();
    addMessage('assistant', fullResponse || text || 'No response');
  } catch(e) {
    removeThinking();
    addMessage('assistant', 'Error: ' + e.message);
  }
  document.getElementById('sendBtn').disabled = false;
  input.focus();
}

function quickSend(text) { document.getElementById('msgInput').value = text; sendMsg(); }
</script>
</body>
</html>
```
This single-page chat interface calls the AgentCore REST API directly from the browser using the token injected by Flask.

## [Step 4: Run the Frontend](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/70-lab6-frontend#step-4:-run-the-frontend)

The Flask server authenticates against Cognito on startup using `USER_PASSWORD_AUTH` — the same flow you used from the CLI in Lab 4.

This flow needs to be enabled on the Cognito web client.

Run:

```bash
COGNITO_POOL_ID=$(aws ssm get-parameter \
  --name /app/customersupport/agentcore/pool_id \
  --query 'Parameter.Value' --output text)

COGNITO_WEB_CLIENT_ID=$(aws ssm get-parameter \
  --name /app/customersupport/agentcore/web_client_id \
  --query 'Parameter.Value' --output text)

aws cognito-idp update-user-pool-client \
  --user-pool-id $COGNITO_POOL_ID \
  --client-id $COGNITO_WEB_CLIENT_ID \
  --explicit-auth-flows ALLOW_USER_PASSWORD_AUTH ALLOW_REFRESH_TOKEN_AUTH \
  --supported-identity-providers COGNITO \
  --allowed-o-auth-flows code \
  --allowed-o-auth-scopes openid email profile \
  --allowed-o-auth-flows-user-pool-client \
  --callback-urls "http://localhost:8501/" \
  --logout-urls "http://localhost:8501/" \
  --no-cli-pager
```

Now start the Flask server:

```bash
cd app/CustomerSupport/frontend
uv run python frontend.py
```

You should see output similar to:

```text
✅ Authenticated as workshopuser@example.com
Runtime ARN: arn:aws:bedrock-agentcore:REGION:ACCOUNT:runtime/CustomerSupport_CustomerSupport-xxxxx
 * Running on http://127.0.0.1:8501
```

The Code Editor will show a notification:

> **Your application running on port 8501 is available**

Click **Open in Browser**.

This opens the frontend through the Code Editor's built-in port forwarding.

![Code Editor port forwarding notification](lab6_browser_notification.png)

### Refreshing Frontend Changes in Code Editor

The Flask server sends no-cache headers so the Code Editor port proxy requests fresh content while you iterate.

Flask is started without its development reloader, so after changing `frontend.py` or `index.html`:

1. Stop the server with `Ctrl+C`.
2. Start it again:

```bash
uv run python frontend.py
```

3. Reload the browser page.

If the proxy still serves an older page, add a cache-busting query parameter to the port URL.

For example:

```text
https://<code-editor-host>/ports/8501/?v=1
```

Change the value after each restart:

```text
?v=2
?v=3
```

A hard refresh may also help:

- **macOS:** `Cmd+Shift+R`
- **Windows/Linux:** `Ctrl+Shift+R`

A hard refresh may clear browser cache, but it cannot always bypass a cached CloudFront response.

### Running Self-Paced on Your Local Machine?

Navigate directly to:

```text
http://localhost:8501
```

The chat interface loads immediately — no login is required.

The server already authenticated as `workshopuser@example.com` on startup and injected the access token into the page.

![Chat interface running in the browser](lab6_chat_interface.png)

## [Step 5: Test the Interface](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/70-lab6-frontend#step-5:-test-the-interface)

Try the quick action buttons or type your own questions:

| Try This | What It Tests |
| --- | --- |
| `What products do you have?` | Product lookup (local tool) |
| `What's the return policy for electronics?` | Return policy (local tool) |
| `Check warranty for PROD-002` | Warranty check (Gateway → Lambda) |
| `Do you remember me?` | Long-term memory recall |

### Test Session Continuity

Send these messages in the same session:

| Message | Expected Result |
| --- | --- |
| `My name is Alex` | Agent acknowledges the name |
| `What's my name?` | Agent remembers "Alex" from earlier in the session |

Now click:

```text
🔄 New Session
```

Then ask:

```text
What's my name?
```

The agent won't know because the session context has reset.

Long-term memory facts, such as preferences, can still persist across sessions.

When you're done testing, press:

```text
Ctrl+C
```

in the terminal running Flask to stop the server before proceeding to the next lab.

## [Congratulations! ✅](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/70-lab6-frontend#congratulations!)

You now have a real user-facing application.

Customers can chat with your agent and get help with:

- Products
- Returns
- Warranties
- Session memory
- Long-term memory

All of this is backed by the same AgentCore infrastructure you've been building since Lab 1.

### [What's Next](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/70-lab6-frontend#what's-next)

Your agent can do anything an authenticated user asks. But should it?

What if someone asks for a `$10,000` refund?

Right now, the agent could potentially process it.

In Lab 7, you'll add deterministic guardrails that control:

- What tools the agent can use
- Under what conditions tools can be used
- Which actions require stronger policy enforcement

You can add these controls without changing the agent code.

→ Next: [Lab 7: Governing Agent Actions with Policies](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/80-lab7-policies/)
