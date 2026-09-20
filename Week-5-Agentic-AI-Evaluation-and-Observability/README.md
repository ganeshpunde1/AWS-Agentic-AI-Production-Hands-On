# BeSA Cohort 10 - Week 5 Notes

## Week 5 Agenda

### Technical Topic
**Agentic AI Observability and Evaluation**

- Duration: 50 minutes
- Presenters: Ashish and Shorena

### Behavioural Topic
**Visual Facilitation and Real-Time Architecture**

- Duration: 30 minutes
- Presenters: Jeff and Krishna

### Workshop
**From Prototype to Production with AWS - Agentic AI Evaluation and Observability**

- Duration: 30 minutes
- Presenter: Robert

---

# Agentic AI Observability

## What is Observability?

Observability means understanding what is happening inside an AI agent.

It helps us answer questions like:

- What did the agent do?
- Which model did it call?
- Which tool did it use?
- Where did it fail?
- Why was it slow?
- How many tokens were used?
- How much did the request cost?

---

# Monitoring vs Observability

## Monitoring

Monitoring tells us:

> Something is wrong.

It mainly uses:

- Metrics
- Dashboards
- Alerts
- Thresholds

Example:

- Agent latency is too high.
- Error count increased.
- GPU utilization dropped.

## Observability

Observability tells us:

> Why is it happening?

It combines:

- Metrics
- Logs
- Traces
- Events
- System context

Example:

If an agent is slow, observability can help us identify whether the delay came from:

- The LLM
- A tool call
- A database
- A network request
- A retry
- Another dependency

### Simple Difference

**Monitoring = Find the problem**

**Observability = Understand why the problem happened**

---

# Why Agentic AI Observability is Difficult

AI agents are more difficult to observe than normal applications because:

1. Agents may take different paths for the same type of request.
2. One request may involve many chained calls.
3. We need to collect and analyze GenAI model calls.
4. We need to look at both system performance and AI response quality.

---

# Common Agent Failures

## 1. Quality Problems

Examples:

- Hallucinations
- Wrong facts
- Bad reasoning
- Poor planning
- Wrong tool selection
- Inconsistent answers

## 2. Reliability Problems

Examples:

- Context is lost
- Errors are not handled properly
- Security problems

## 3. Efficiency Problems

Examples:

- High cost
- High latency
- Poor customer experience
- Loss of customer trust

---

# Three Main Pillars of Observability

## 1. Metrics

Metrics are numbers collected over time.

Examples:

- Number of sessions
- Number of agent invocations
- Latency
- Token usage
- Error count

Example question:

> Why did agent latency increase by 15%?

## 2. Logs

Logs are timestamped records of events.

They can contain:

- Agent events
- Requests
- Responses
- Tool activity
- Errors

Example question:

> What error happened during agent or tool execution?

## 3. Traces

A trace shows the complete path of one request.

It can show:

- Agent steps
- Model calls
- Tool calls
- Dependencies
- Time spent in each step

Example question:

> What path did the agent take and where was the time spent?

---

# What is a Trace?

A **trace** is the complete record of one agent request from start to finish.

Example:

User Request  
→ Agent Plans  
→ LLM Call  
→ Tool Call  
→ Data Retrieval  
→ LLM Generates Answer  
→ Agent Response

One agent request normally creates one trace.

---

# What is Instrumentation?

Instrumentation means adding observability code to your application.

It allows the application to generate telemetry.

Telemetry means data about what is happening inside the application.

Examples of telemetry:

- Metrics
- Logs
- Traces

## Basic Flow

Agent Application  
→ Instrumentation  
→ Telemetry  
→ Observability System

---

# OpenTelemetry

**OpenTelemetry (OTel)** is an open-source standard for collecting and sending:

- Traces
- Metrics
- Logs

It is vendor-neutral.

On AWS, the slides mention **ADOT**:

**AWS Distro for OpenTelemetry**

---

# AWS Observability Flow

## Step 1: Instrument

Add observability to the agent.

AWS example:

- Agent application
- ADOT

## Step 2: Collect / Export

Send telemetry to the observability platform.

Possible options:

- Direct OTLP export to CloudWatch
- ADOT Collector
- CloudWatch agent

## Step 3: Store

AWS can store telemetry using:

- Amazon CloudWatch
- AWS X-Ray

## Step 4: Analyze

Use:

- CloudWatch dashboards
- Application Signals
- CloudWatch GenAI Observability

---

# Agentic AI Observability on AWS

AWS provides different options for observing AI agents.

## Amazon Bedrock AgentCore Observability

Used for agent-specific observability.

It can help observe:

- Sessions
- Traces
- Spans
- Tool execution
- Performance
- Errors

## Amazon CloudWatch GenAI Observability

Provides central dashboards and views for:

- GenAI resources
- Agents
- Traces
- Logs
- Metrics
- Troubleshooting

## Third-Party Observability Tools

Examples shown in the slides include:

- Datadog
- Dynatrace
- Arize Phoenix
- LangSmith
- Braintrust
- Confident AI

---

# Amazon Bedrock AgentCore Observability

AgentCore Observability can work with different parts of AgentCore, including:

- AgentCore Runtime
- AgentCore Memory
- AgentCore Identity
- AgentCore Gateway and Policy
- AgentCore Evaluations
- AgentCore Browser
- AgentCore Code Interpreter
- MCP-based tools

It can send OpenTelemetry logs and connect to:

- AgentCore Observability dashboards
- Third-party observability dashboards

---

# Evaluation

Observability tells us what happened.

But that is not enough.

We also need to know:

- Was the result correct?
- Did the agent make the right decision?
- Did it use the correct tool?
- Did it reach the goal?
- Was the answer useful?

This is called **evaluation**.

---

# Observability vs Evaluation

## Observability

Answers:

> What is happening?

It looks at:

- Calls
- Waits
- Loops
- Errors
- Latency
- Failures
- Tool activity

Main purpose:

**Visibility**

## Evaluation

Answers:

> Is it happening correctly?

It looks at:

- Correct actions
- Correct decisions
- Goal achievement
- Answer quality

Main purpose:

**Quality**

---

# Agent Evaluation

Agent evaluation checks whether the agent is doing a good job.

Questions include:

- Did the agent achieve its goal?
- Did the agent make the right decision?
- Did it choose the best tool?
- Was the answer accurate?

---

# Amazon Bedrock AgentCore Evaluations

AgentCore Evaluations provides managed quality assessment for AI agents.

It can:

- Analyze agent behavior
- Evaluate specific quality criteria
- Use built-in evaluators
- Use custom evaluators
- Send evaluation results into AgentCore Observability

AWS handles infrastructure tasks such as:

- Scaling and capacity
- Rate limits and throttling
- Infrastructure operations

---

# When Should We Evaluate an Agent?

Evaluation should happen:

- Periodically
- Continuously

You should especially evaluate after:

- Updating a model
- Creating a new version of the agent
- Changing a system/source prompt
- Adding tools
- Removing tools

---

# Important AgentCore Concepts

## Session

A session is the complete conversation or user interaction.

A session can contain multiple requests.

A session can contain multiple traces.

## Trace

A trace is one complete agent request.

Example:

User input  
→ Agent processing  
→ Final answer

## Span

A span is one operation inside a trace.

Examples:

- LLM call
- Tool call
- Retrieval call
- Database call

## Sub-Span

A sub-span is a smaller operation inside a span.

Examples:

- Retry
- Intermediate step
- Sub-agent call

### Hierarchy

Session  
→ Trace  
→ Span  
→ Sub-Span

---

# AgentCore Evaluation Metrics

Evaluation can happen at different levels.

## Session-Level Metrics

### Goal Success Rate

Checks:

> Did the complete session achieve its objective?

---

# Trace-Level Metrics

## Correctness

Was the output accurate?

## Helpfulness

Was the answer useful?

## Faithfulness

Was the answer grounded in the available facts?

## Response Relevance

Was the answer relevant to the user's question?

## Conciseness

Was the answer clear and not unnecessarily long?

## Coherence

Did the answer make logical sense?

## Instruction Following

Did the agent follow the instructions?

## Harmfulness

Did the response contain harmful content?

## Refusal

Did the agent refuse when appropriate?

## Stereotyping

Did the response contain unwanted stereotypes?

---

# Span / Tool-Level Metrics

## Tool Selection Accuracy

Did the agent choose the correct tool?

Example:

For weather information, did it use a weather tool instead of a calculator?

## Parameter Selection Accuracy

Did the agent send the correct parameters to the tool?

Example:

If the tool needs:

- Location
- Date
- Number of users

Did the agent provide the correct values?

---

# Built-In AgentCore Evaluation Metrics

The slides show these built-in evaluation areas:

### Session Level

- Goal Success Rate

### Trace Level

- Correctness
- Faithfulness
- Helpfulness
- Response Relevance
- Conciseness
- Coherence
- Instruction Following
- Refusal
- Harmfulness
- Stereotyping

### Tool / Span Level

- Tool Selection Accuracy
- Parameter Selection Accuracy

### Custom Metrics

You can also create your own evaluation metric.

---

# How AgentCore Evaluation Works

Simple flow:

AI Agent  
→ OpenTelemetry Traces  
→ CloudWatch Logs  
→ AgentCore Evaluation  
→ Observability Dashboard

---

# Custom Evaluators Using LLM-as-a-Judge

An LLM can also evaluate another AI response.

This is called:

**LLM-as-a-Judge**

## Step 1: Define Evaluation Criteria

Example:

> Purchase Intent Detection

We want to know whether the user wants to buy something.

## Step 2: Create an Evaluation Prompt

Ask the evaluator to check signals such as:

- Clear purchase statement
- Pricing question
- Checkout question
- Urgency

Example score:

- `0.0` = No purchase intent
- `0.5` = Some interest
- `1.0` = Clear purchase intent

## Step 3: Select Evaluation Model

Choose an LLM to act as the evaluator.

## Step 4: Configure Sampling

Choose which agent interactions should be evaluated.

---

# Best Practices for Observability and Evaluation

## 1. Start Simple

Do not build everything at once.

Start with basic metrics, logs, and traces.

## 2. Integrate Early

Add observability while developing the agent.

Do not wait until production.

## 3. Use Consistent Naming

Use clear and consistent names for:

- Sessions
- Traces
- Spans
- Tools
- Metrics

## 4. Filter Sensitive Data

Do not store sensitive data unnecessarily in:

- Logs
- Traces
- Prompts
- Responses

## 5. Configure for the Development Stage

Development and production may require different observability settings.

## 6. Review Regularly

Regularly review:

- Logs
- Traces
- Metrics
- Evaluation results

## 7. Stay Framework Agnostic

Use standards such as OpenTelemetry so your observability system can work with different frameworks and tools.

---

# Minimum Viable Observability

If you can only do three things, start with these.

## 1. Instrument Traces

Use OpenTelemetry.

Track:

- Agent activity
- Performance
- Cost

## 2. Define 3-5 Business Metrics

Measure the things that actually matter to the business.

Examples:

- Task success rate
- Customer satisfaction
- Cost per request
- Response time
- Tool success rate

## 3. Run Automated Evaluation Weekly

Regular evaluation helps detect quality problems or drift early.

---

# Jupyter Notebook Skills

The slides recommend learning basic Jupyter Notebook navigation.

Resources mentioned:

- A short YouTube getting-started video
- The official VS Code Jupyter Notebook tutorial

---

# Week 5 Next Steps

1. Complete the Knowledge Check.
2. Practice the hands-on workshop.
3. Be active and support other participants.
4. Share your learning on social media.

---

# Very Simple Summary

For an AI agent in production, remember these four things:

## 1. Metrics

**How much?**

Examples:

- Latency
- Errors
- Tokens
- Cost

## 2. Logs

**What happened?**

Examples:

- Error message
- Tool response
- Agent event

## 3. Traces

**How did the request move through the system?**

Example:

User  
→ Agent  
→ LLM  
→ Tool  
→ Database  
→ LLM  
→ Response

## 4. Evaluation

**Was the result good?**

Examples:

- Was it correct?
- Was it helpful?
- Did it use the correct tool?
- Did it achieve the goal?

---

# Easy Formula to Remember

**Observability = See what the agent is doing**

**Evaluation = Check whether the agent is doing it correctly**

And:

**Metrics + Logs + Traces = Observability**

**Correctness + Quality + Goal Success = Evaluation**
