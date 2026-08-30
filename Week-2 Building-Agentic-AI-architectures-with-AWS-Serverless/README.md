# Building Agentic AI Architectures with AWS Serverless

## Welcome 

Welcome to the **Building Agentic AI Architectures with AWS Serverless** workshop—where you'll discover how to build AI agent systems that actually work in production.

While synchronous agent interactions are easy to prototype, they can become problematic when agents need real time to:

- Think and reason
- Analyze data
- Call external APIs
- Process complex reasoning chains
- Wait for human approvals

In production, agents cannot afford to remain idle waiting for responses, consuming serverless execution time and potentially hitting timeout limits.

This workshop takes you beyond the basics. You'll master **asynchronous coordination patterns—choreography and orchestration**—that are essential for building resilient and scalable multi-agent systems.

Through hands-on exercises with a real-world travel booking scenario, you'll learn how serverless architectures on AWS solve challenges such as:

- Coordinating independent agents
- Managing long-running workflows
- Integrating human-in-the-loop decision points
- Maintaining resilience when agents experience delays or failures
- Scaling automatically
- Optimizing cost

You will build production-ready architectures using:

- **AWS Lambda**
- **Amazon EventBridge**
- **AWS Step Functions**

> **Workshop Studio Account Required**
>
> This workshop can only be accessed using AWS accounts provided by Workshop Studio at AWS-run events. You cannot run this workshop in your personal or organizational AWS account.
>
> If you are not participating in an official AWS event, contact AWS to inquire about upcoming workshop opportunities.

---

## What You’ll Learn

You will build the same multi-agent system twice, each time using a different coordination pattern.

### Architecture

![Architecture diagram comparing orchestration with centralized Step Functions control and choreography with distributed EventBridge coordination](https://static.us-east-1.prod.workshops.aws/25e8af74-1e29-468a-b82f-ded9616f361c/static/img/orch_vs_choreo.jpg?Key-Pair-Id=K36Q2WVO3JP7QD&Policy=eyJTdGF0ZW1lbnQiOlt7IlJlc291cmNlIjoiaHR0cHM6Ly9zdGF0aWMudXMtZWFzdC0xLnByb2Qud29ya3Nob3BzLmF3cy8yNWU4YWY3NC0xZTI5LTQ2OGEtYjgyZi1kZWQ5NjE2ZjM2MWMvKiIsIkNvbmRpdGlvbiI6eyJEYXRlTGVzc1RoYW4iOnsiQVdTOkVwb2NoVGltZSI6MTc4ODY5MjM0NX19fV19&Signature=JOC6pn5wKSMtNEzfpJ2JMq1S9s3VEPFTZCI7lh59DkT73Gg4ODqaZvOLYnauB0QBwb-CEEI5tEQ%7EAJD8AQt3-5etqtjj66jRVwtR0FSJB2-DS7dxxObo9sg1Qm9P5eQH4M1XVJ1mGZHsU9cDFxWBT4GjMhgy4X4N8KdhgxNZLEv-naXdHXX-frjhsJxjdDwzJQUk6Ob3nkfk70Ksg3Z8AgknDYyemQlfx9PJ7Ietnh6xFRiNkD2AEYxBy608xfp7%7EIFR0MQ%7E2guI2VYXGox%7ETrWn0nW87-Yt8s1kiaMjm6iI2YJuIiNUglVEvficlOsYs9jHJW%7EmCVeKn6b0nKRTsg__)

### Choreography

Agents communicate through events using:

- **Amazon EventBridge**
- **AWS Lambda**

This approach emphasizes:

- Loose coupling
- Flexibility
- Independent agent behavior
- Event-driven communication

### Orchestration

Agents are coordinated centrally using:

- **AWS Step Functions**
- **AWS Lambda**

This approach emphasizes:

- Visibility
- Central control
- State management
- Workflow sequencing
- Error handling

By completing both implementations, you will understand the trade-offs between the two patterns and learn how to choose the right model for your applications.

> **Pre-Built Resources**
>
> To help you focus on coordination patterns, the workshop provides pre-deployed Lambda functions for all three agents:
>
> - Planner Agent
> - Weather Agent
> - Flight Manager Agent
>
> Supporting infrastructure is also provided, including:
>
> - Amazon S3 buckets for session management
> - IAM roles
> - Supporting AWS resources
>
> You will focus on connecting these agents using **EventBridge rules for choreography** and **Step Functions state machines for orchestration**, rather than building the agents from scratch.
>
> All agent code is available for review in the workshop resources.

---

## Workshop Features 

- **Hands-on Labs** – Step-by-step instructions to build real systems
- **Real-World Scenario** – Implement a travel planning system with specialized agents
- **Pattern Comparison** – Explore benefits and trade-offs of choreography vs. orchestration
- **Best Practices** – Apply principles for scalability, resilience, and fault isolation
- **Validation & Testing** – Confirm each implementation works as intended and compare outcomes

---

## Business Scenario

**TravelCorp** is a mid-sized corporate travel management company serving:

- **200+ enterprise clients**
- **50,000+ flight bookings annually**

Their current workflow involves sequential and manual processes.

Travel agents typically:

1. Receive booking requests through email or phone
2. Research weather conditions manually
3. Compare flight options across airline portals
4. Coordinate approvals through email chains

Although this traditional approach is reliable, it creates bottlenecks during high-demand periods and limits the company's ability to provide fast, intelligent service.

TravelCorp sees an opportunity to use **event-driven architecture and AI agents** to create a more responsive, intelligent, and scalable travel booking platform.

Their enterprise clients increasingly expect:

- Real-time decision-making
- Transparent processes
- Intelligent automation
- Human involvement when needed
- Support for complex travel scenarios

---

## Workshop Structure

### Duration

Approximately **1 hour** of guided labs and testing.

### Difficulty

**Intermediate**

Suitable for developers and architects who are familiar with AWS basics.

### Prerequisites

- AWS account with permissions to use:
  - AWS Lambda
  - Amazon EventBridge
  - AWS Step Functions
- Basic knowledge of AWS CLI
- Basic knowledge of the AWS Management Console
- Python programming experience for Lambda functions

The workshop is designed to be completed in a single session.

However, the modules are independent, so you can revisit the **choreography** and **orchestration** patterns separately.

---

## Workshop Modules

1. [Introduction](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/introduction)
2. [Module 1: Choreography Pattern](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/module-1-choreography)
3. [Module 2: Orchestration Pattern](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/module-2-orchestration)
4. [[Optional] Extending Multi-Agent Patterns](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/module-3-observability)
5. [Summary](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/summary)

---

## Key Takeaways

After completing this workshop, you should understand:

- How asynchronous AI agent systems work
- How agents communicate using events
- How choreography differs from orchestration
- When to use Amazon EventBridge for agent coordination
- When to use AWS Step Functions for centralized workflows
- How AWS Lambda supports serverless AI agent architectures
- How to handle long-running agent workflows
- How to integrate human approvals
- How to design resilient multi-agent systems
- How to choose between choreography and orchestration patterns

---

## Learning Resources

- [AWS Workshop Studio – Building Agentic AI Architectures with AWS Serverless](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop)
- [YouTube – Building Agentic AI Architectures with AWS Serverless](https://www.youtube.com/watch?v=Rx2LNrEXHS8)

# [Optional] Anatomy of an Agent


**Optional Reading** – If you're already familiar with agent concepts (personality, memory, tools, and reason-action loops), feel free to skip this chapter and proceed directly to Module 1. This section provides foundational context for those new to agentic systems.

Agents are specialized programs that can **reason**, **remember**, and **act**. Unlike traditional functions that simply execute predefined logic, agents make decisions based on context, maintain state across interactions, and use tools to accomplish goals.

## [The Reason and Action Loop](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/introduction/introduction-to-agents#the-reason-and-action-loop)

Agents operate in a continuous cycle of reasoning and action. This loop enables them to break down complex tasks, make decisions, and adapt based on results:

1. **Reason** – Analyze the current context and determine what needs to be done
2. **Select Tool** – Choose the appropriate tool based on the current goal
3. **Execute** – Invoke the tool and receive results
4. **Update Context** – Incorporate results into memory and reassess
5. **Repeat or Complete** – Continue the loop or return final results

Flowchart showing the agentic reasoning loop: reason, select tool, execute, update context, and repeat or complete
![Agentic Reasoning Loop](agentic_loop.png)

---

## [Workshop Agents](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/introduction/introduction-to-agents#workshop-agents)

In this workshop, you'll work with three pre-built agents that collaborate to handle travel bookings:

- **Planner Agent** – Coordinates the booking workflow and makes final decisions
- **Weather Agent** – Provides weather intelligence for travel planning
- **Flight Manager Agent** – Handles flight search and booking operations

Each agent has three core traits that enable intelligent behavior:

- **Personality** – How the agent behaves and makes decisions based on its role
- **Memory** – How the agent maintains context across steps and interactions
- **Tools** – The actions the agent can take to interact with data and services

![Agent Components - Personality, Memory, and Tools](AgentExplanation.png)

---

## [Personality](https://github.com/ganeshpunde1/Building-Agentic-AI-architectures-with-AWS-Serverless/blob/main/README.md#personality)


**Beyond Traditional Programming**

Traditional programs are deterministic—`if budget > limit then reject`. The same input always produces the same output. Every decision path must be explicitly programmed. Agents with personality work differently: they're probabilistic and context-aware, interpreting situations and making judgment calls based on their role—all guided by natural language instructions called system prompts.

This shift from deterministic to adaptive behavior is fundamental. A traditional function returns the same result every time. An agent with personality might handle the same budget scenario differently based on context: trip urgency, weather risks, traveler history, or seasonal factors. The personality provides consistent character and decision-making principles, while allowing creative problem-solving within those boundaries.

For example, instead of coding every budget scenario, you give the Planner Agent a personality: "You are a cautious travel coordinator who prioritizes traveler safety and budget compliance." The agent then applies this guidance to make nuanced decisions across countless situations you never explicitly programmed.

**Personality in Action**

Personality defines the role, behavior, and decision-making style of an agent through system prompts. In this workshop:

- **Planner Agent** acts as a travel coordinator who extracts trip requirements, evaluates options from other agents, and decides whether to auto-approve bookings or route them for human review based on risk factors. When faced with a borderline budget situation, it considers the full context—traveler history, trip importance, weather risks—rather than applying a simple threshold.
- **Weather Agent** serves as a meteorologist who analyzes weather conditions at destinations, assesses travel risks (storms, extreme temperatures), and provides clear recommendations. It doesn't just report data—it interprets what weather conditions mean for travelers and adjusts its advice based on the trip context.
- **Flight Manager Agent** functions as a booking specialist who searches available flights, evaluates options against budget and preferences, and determines the best matches. It balances competing priorities (cost, convenience, airline preference) without requiring explicit rules for every trade-off.

Each agent's personality ensures it stays focused on its specialized role while contributing to the overall booking workflow—making intelligent decisions you didn't have to code explicitly.

---

## [Memory](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/introduction/introduction-to-agents#memory)

**Beyond Stateless Functions**

Traditional serverless functions are stateless by design—each invocation starts fresh with no memory of previous calls. To maintain state across invocations, you typically use databases to store structured data: user records, transaction history, or application state. This works well for transactional data but becomes cumbersome for conversational context—you'd need to design schemas for conversation history, tool invocations, reasoning steps, and intermediate results, then manually manage serialization and retrieval.

Agents with memory work differently: they maintain rich conversational context including dialogue history, tool results, and reasoning chains across multiple invocations. When the Planner Agent receives weather data, it doesn't just process it in isolation—it recalls the original booking request, previous tool calls, and the reasoning that led to this point. This conversational memory is more fluid than traditional database state, capturing the full context of an ongoing interaction rather than just discrete data points.

**Serverless Challenge:**

AWS Lambda functions are stateless—they don't retain information between invocations. Agents need persistent storage to maintain memory across executions. The Strands SDK (next chapter) handles this by providing session management that works seamlessly with Lambda's stateless model.

**Memory Patterns:**

- **Choreography**: Each agent independently manages its own memory; coordination happens through event payloads
- **Orchestration**: Agents maintain local memory while Step Functions maintains global workflow state

---

## [Tools](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/introduction/introduction-to-agents#tools)

**Beyond Fixed Capabilities**

Traditional programs have fixed capabilities—they can only do what their code explicitly implements. Want to add a new feature? You write more code, deploy a new version, and restart the system. The program's abilities are frozen at deployment time.

Agents with tools work differently: they dynamically select and invoke capabilities based on what they're trying to accomplish. The agent doesn't have a predetermined execution path—it reasons about the goal, chooses appropriate tools, interprets results, and decides what to do next. The same agent can handle countless scenarios by creatively combining tools in ways you never explicitly programmed.

For example, the Planner Agent doesn't follow a script like "call weather API, then call flight API, then decide." Instead, it reasons: "I need weather data for this destination—I'll use the weather tool. The forecast shows storms—I should factor that into my decision. Now I need flight options—I'll use the flight search tool." The agent orchestrates its own workflow.

The Strands SDK (next chapter) simplifies tool integration using Python decorators, handling schema generation, invocation, and result formatting automatically.


**Workshop Simplification** – Tools use dummy data to demonstrate coordination patterns without external dependencies. In production, tools can connect to real APIs, enterprise systems, or leverage the Model Context Protocol (MCP) for remote tool access.

---

## [Coordinating Multiple Agents](https://catalog.us-east-1.prod.workshops.aws/event/dashboard/en-US/workshop/introduction/introduction-to-agents#coordinating-multiple-agents)

Coordinating multiple agents requires a coordination pattern. This workshop demonstrates two approaches: **Choreography** (Module 1) uses events without a central controller, while **Orchestration** (Module 2) uses a state machine to manage the workflow. You'll explore both hands-on to understand their trade-offs and when to use each.

