# Amazon Bedrock AgentCore Workshop — Summary

## 🎯 What You Built

Over the course of this workshop, you went from **zero to a production-ready customer support agent** using the **AgentCore CLI**.

You progressively added runtime deployment, memory, gateway tools, authentication, observability, evaluation, governance, a web interface, zero-code agents, and A/B testing.

| Lab | What You Built | Main AgentCore Services |
|---|---|---|
| **1** | Created the project, added local tools, integrated an MCP tool using **Exa AI**, and tested locally with `agentcore dev` | AgentCore CLI, Runtime (local) |
| **2** | Added persistent memory so the agent can remember facts and conversation summaries across sessions, then deployed to AWS | Memory, Runtime |
| **3** | Exposed an existing Lambda function as an MCP-compatible agent tool through Gateway | Gateway |
| **4** | Secured Runtime and Gateway with Cognito JWT authentication, then explored sessions and observability | Identity, Observability |
| **5** | Added continuous quality monitoring with built-in evaluators for goal success, correctness, and tool selection | Evaluations |
| **6** | Built a web chat application with Cognito login connected to the deployed agent | Combined AgentCore services |
| **7** | Added fine-grained access control using Cedar policies based on tool input parameters | Policy |
| **8** | Built a zero-code **Order Research Agent** from CLI configuration and explored shell access, filesystem persistence, model overrides, and human-in-the-loop flows | Harness |
| **9** | Generated AI-driven prompt/tool recommendations, packaged them as configuration bundles, ran Gateway A/B testing, and validated the winning version statistically | Optimization |

---

## 🧠 Core Architecture You Learned

A production AgentCore application can be thought of as a set of layers:

```text
User / Web App
      │
      ▼
Identity / Cognito
      │
      ▼
AgentCore Runtime
      │
      ├── Memory
      ├── Observability
      ├── Evaluations
      │
      ▼
AgentCore Gateway
      │
      ├── MCP Tools
      ├── Lambda / APIs / Services
      └── Cedar Policies
      │
      ▼
Foundation Model
      │
      ▼
Optimization + A/B Testing
```

The important idea is that **production capabilities are separated from agent business logic**. Authentication, memory, policies, monitoring, and optimization do not all need to be hard-coded into the agent itself.

---

# 🔑 Key Takeaways

## 1. AgentCore CLI hides infrastructure complexity

The CLI handled most infrastructure work for you.

You did **not** need to manually:

- write a Dockerfile
- configure Amazon ECR
- create IAM roles by hand
- wire every AWS resource yourself

A typical deployment was simply:

```bash
agentcore deploy
```

The CLI handled packaging, uploading, provisioning, and resource integration.

### Remember

> **AgentCore CLI = agent deployment and infrastructure orchestration with much less manual AWS setup.**

---

## 2. AgentCore is framework and model agnostic

The workshop used:

- **Strands Agents**
- **Claude on Amazon Bedrock**

But AgentCore can also work with frameworks such as:

- LangGraph
- CrewAI
- OpenAI Agents SDK
- Google ADK

The runtime is also designed to work with different foundation models.

### Remember

```text
AgentCore ≠ one agent framework
AgentCore ≠ one LLM
```

It is infrastructure for running and managing production AI agents.

---

## 3. Memory turns a demo into a real product

Without memory:

```text
Conversation 1 → Agent learns something
Conversation ends
Conversation 2 → Agent starts from zero
```

With AgentCore Memory:

```text
Conversation 1
      │
      ▼
Store facts + summaries
      │
      ▼
Conversation 2
      │
      ▼
Recall useful past context
```

Memory allows the agent to:

- remember facts
- summarize previous conversations
- reuse relevant context
- provide continuity across sessions

### Key idea

> **Memory converts a stateless chatbot into a persistent customer experience.**

---

## 4. Gateway turns existing APIs into agent tools

Many organizations already have business logic inside:

- Lambda functions
- REST APIs
- internal services
- enterprise applications

AgentCore Gateway allows those systems to become **discoverable, authenticated MCP tools** without rewriting the original implementation.

Conceptually:

```text
Existing API / Lambda
        │
        ▼
AgentCore Gateway
        │
        ▼
MCP Tool
        │
        ▼
AI Agent
```

### Key idea

> **Gateway helps agents safely reuse existing enterprise systems.**

---

## 5. Security is mainly configuration, not an agent rewrite

You secured both Runtime and Gateway using **Cognito JWT authentication**.

The major changes involved:

- authentication configuration in `agentcore.json`
- passing tokens between Runtime and Gateway
- configuring authorization on the MCP client

The agent logic itself required relatively little change.

The same overall pattern can work with other OAuth 2.0-compatible identity providers.

### Security flow

```text
User
 │
 ▼
Cognito Login
 │
 ▼
JWT Token
 │
 ▼
AgentCore Runtime
 │
 ▼
Gateway
 │
 ▼
Authorized Tool
```

---

## 6. Governance should live outside the agent

AgentCore Policy uses **Cedar policies** to enforce business rules at the Gateway boundary.

Examples include:

- spending limits
- refund limits
- tool restrictions
- parameter-based permissions
- user- or role-specific access

The agent does not need to implement these rules itself.

```text
Agent requests tool
       │
       ▼
AgentCore Gateway
       │
       ▼
Cedar Policy Evaluation
       │
   ┌───┴────┐
   │        │
 Permit    Deny
```

Policies can be updated without redeploying the agent, and policy decisions are logged for auditing.

### Key idea

> **The LLM decides what it wants to do. Policy decides what it is allowed to do.**

---

## 7. Observability and evaluation are built into the platform

AgentCore automatically instruments invocations with **OpenTelemetry**.

You can observe:

- agent calls
- tool calls
- session behavior
- traces
- model interactions

You also added continuous evaluation using built-in evaluators such as:

- Goal Success Rate
- Correctness
- Tool Selection

This changes monitoring from:

```text
"Is the service running?"
```

into:

```text
"Is the agent actually doing a good job?"
```

---

## 8. Optimization turns monitoring data into improvement

In the final lab, production traces and evaluator results became input for **AI-generated optimization recommendations**.

The workflow was:

```text
Production Traces
      │
      ▼
Evaluation Results
      │
      ▼
AI Recommendations
      │
      ▼
Configuration Bundles
      │
      ▼
A/B Test
      │
      ▼
Statistical Comparison
      │
      ▼
Promote Winner
```

This means agent changes can be validated using evidence instead of intuition.

### Key idea

> **Observe → Evaluate → Optimize → A/B Test → Promote → Repeat**

---

# 🔁 Complete Production Improvement Loop

The overall production lifecycle you learned is:

```text
Build
  ↓
Deploy
  ↓
Secure
  ↓
Add Memory
  ↓
Connect Tools
  ↓
Apply Policies
  ↓
Observe
  ↓
Evaluate
  ↓
Optimize
  ↓
A/B Test
  ↓
Promote Better Version
  ↓
Repeat
```

This is one of the most important concepts from the workshop.

---

# 🧩 AgentCore Services at a Glance

| Service | Purpose |
|---|---|
| **Runtime** | Runs the AI agent in production |
| **Memory** | Stores and recalls useful information across sessions |
| **Gateway** | Exposes APIs and services as agent-accessible tools |
| **Identity** | Adds authentication and authorization |
| **Observability** | Provides traces and operational visibility |
| **Evaluations** | Measures agent quality continuously |
| **Policy** | Enforces deterministic business and security rules |
| **Harness** | Supports zero-code/config-driven agent construction and execution |
| **Optimization** | Generates recommendations and supports controlled experimentation |
| **CLI** | Creates, configures, deploys, manages, and removes AgentCore resources |

---

# 🧹 Clean Up

After completing the workshop, remove resources to avoid unnecessary AWS charges.

## 1. Remove AgentCore resources

Run:

```bash
agentcore remove all
agentcore deploy
```

This removes the workshop AgentCore resources, including the deployed Runtime, Memory, Gateway, Identity, and Evaluation resources.

---

## 2. Delete the prerequisites CloudFormation stack

Run:

```bash
aws cloudformation delete-stack --stack-name prereqs
```

This removes prerequisite resources such as:

- Cognito User Pool
- warranty-check Lambda function
- IAM roles
- SSM parameters

---

## 3. Wait for CloudFormation deletion to finish

```bash
aws cloudformation wait stack-delete-complete --stack-name prereqs
```

When this command completes successfully, the `prereqs` stack has been deleted.

---

# 📚 What's Next?

## Deep-dive workshop

Continue with:

[Diving Deep into Bedrock AgentCore](https://catalog.workshops.aws/agentcore-deep-dive/en-US)

It goes deeper into individual services such as:

- Runtime
- Gateway
- Identity
- Memory
- Tools
- Observability
- Evaluations
- Policy
- Agent Registry

---

## Useful Resources

### AgentCore Samples

Ready-to-deploy examples covering multiple frameworks, use cases, and AgentCore capabilities:

[AgentCore Samples Repository](https://github.com/awslabs/agentcore-samples)

### AgentCore CLI

Advanced CLI topics such as VPC networking, container deployments, and policy engines:

[AgentCore CLI Documentation](https://github.com/aws/agentcore-cli)

### Official AWS Documentation

Full Amazon Bedrock AgentCore service documentation:

[Amazon Bedrock AgentCore Documentation](https://docs.aws.amazon.com/bedrock-agentcore/)

---

# 📝 Final Revision Notes

If you remember only these points, remember these:

1. **Runtime** runs the agent.
2. **Memory** gives the agent continuity.
3. **Gateway** converts enterprise services into agent tools.
4. **Identity** secures access with JWT/OAuth-style authentication.
5. **Policy** controls what tools and operations are allowed.
6. **Observability** shows what the agent is doing.
7. **Evaluations** measure whether the agent is doing it well.
8. **Harness** supports config-driven and zero-code agent workflows.
9. **Optimization** uses traces and evaluations to improve prompts/tools.
10. **A/B testing** proves whether a new configuration is actually better.

---

# ⭐ One-Line Workshop Summary

> **Amazon Bedrock AgentCore provides the production infrastructure around an AI agent — deployment, memory, tools, identity, governance, observability, evaluation, and continuous optimization — while allowing the underlying agent framework and model to remain flexible.**

---

## ✅ Workshop Complete

You successfully followed the complete journey from:

```text
Local Agent Prototype
        ↓
Cloud Deployment
        ↓
Persistent Memory
        ↓
Enterprise Tools
        ↓
Authentication
        ↓
Observability
        ↓
Continuous Evaluation
        ↓
Web Application
        ↓
Governance
        ↓
Zero-Code Agents
        ↓
AI Optimization
        ↓
A/B Testing
        ↓
Production Improvement Loop
```

**Congratulations — you completed the Amazon Bedrock AgentCore workshop. 🎉**
