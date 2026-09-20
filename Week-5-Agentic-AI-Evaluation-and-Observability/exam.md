Below is the consolidated version of all **20 questions**, including the options, correct answers, and explanations in **very simple words**.

## Agentic AI Observability – 20 Questions with Answers

### 1. Which are the three telemetry types used for Agentic AI Observability?

**Options:**

* Metrics
* Snapshots
* Traces
* Logs

✅ **Correct answers:** **Metrics, Traces, Logs**

**Simple explanation:**
These are the main types of observability data:

* **Metrics** = numbers such as latency and error rate
* **Traces** = complete request journey
* **Logs** = detailed events and errors

---

### 2. What are unique challenges for Agentic AI Observability?

**Options:**

* Agents follow non-deterministic control flows powered by model reasoning
* Root cause analysis involves complex chained calls
* Eliminating the need for telemetry collection
* Assessing system health for performance and quality

✅ **Correct answers:**

* **Agents follow non-deterministic control flows**
* **Root cause analysis involves complex chained calls**
* **Assessing system health holistically**

**Simple explanation:**
AI agents can behave differently each time and may call many models and tools, which makes troubleshooting harder.

---

### 3. Which are trace-level metrics in AgentCore Evaluation?

**Options:**

* Faithfulness
* Correctness
* Helpfulness
* Parameter selection accuracy

✅ **Correct answers:**
**Faithfulness, Correctness, Helpfulness**

**Simple explanation:**
These check the overall quality of an agent's response during a trace.

---

### 4. When should you evaluate an agent?

**Options:**

* When you modify a source prompt
* When you update a model
* When there is a new version of the agent
* Only once during initial deployment

✅ **Correct answers:**

* **When you modify a prompt**
* **When you update a model**
* **When there is a new agent version**

**Simple explanation:**
Whenever an important part of the agent changes, test it again.

---

### 5. Which are AWS options for Agentic AI Observability?

**Options:**

* Amazon Bedrock AgentCore Observability
* Amazon CloudWatch GenAI Observability
* Amazon Route 53 Observability
* AWS Snowball Observability

✅ **Correct answers:**

* **Amazon Bedrock AgentCore Observability**
* **Amazon CloudWatch GenAI Observability**

**Simple explanation:**
These AWS services help monitor AI agents and GenAI workloads.

---

### 6. What is the primary goal of Monitoring?

**Options:**

* Replace dashboards and alerts
* Detect known issues and track system health
* Understand why the system behaves a certain way
* Correlate metrics, logs, traces, and events for root cause

✅ **Correct answer:**
**Detect known issues and track system health**

**Simple explanation:**
Monitoring tells you **what is wrong** or whether the system is healthy.

---

### 7. Which question does Observability mainly answer?

**Options:**

* Who acknowledged the incident?
* Is something wrong?
* When did the alert fire?
* Why is it happening?

✅ **Correct answer:**
**Why is it happening?**

**Simple explanation:**
Monitoring says something is wrong. Observability helps explain **why**.

---

### 8. Which telemetry type captures agent steps, model calls, tool calls, dependencies, and timing?

**Options:**

* Metrics
* Logs
* Traces
* Alerts

✅ **Correct answer:** **Traces**

**Simple explanation:**
A trace shows the full path of an agent request from start to finish.

---

### 9. What is a trace?

**Options:**

* A single operation such as one model call
* Nested detail inside a span
* A complete user interaction across multiple turns
* Complete record of one agent request from start to finish

✅ **Correct answer:**
**The complete record of a single agent execution/request from start to finish**

**Simple explanation:**
A trace shows everything that happened during one agent request.

---

### 10. What does Instrumentation do?

**Options:**

* Visualizes dashboards and alerts
* Evaluates agent outputs
* Stores telemetry in a backend
* Adds observability code so telemetry can be collected

✅ **Correct answer:**
**Adds observability code to the application so telemetry can be collected**

**Simple explanation:**
Instrumentation allows the application to generate **metrics, logs, and traces**.

---

### 11. What is OpenTelemetry (OTel)?

**Options:**

* A CloudWatch dashboard product
* A proprietary AWS-only monitoring agent
* A foundation model used for evaluation
* An open-source, vendor-neutral toolkit for traces, metrics, and logs

✅ **Correct answer:**
**An open-source, vendor-neutral standard and toolkit for generating and exporting traces, metrics, and logs**

**Simple explanation:**
OpenTelemetry helps collect observability data and send it to different monitoring tools.

---

### 12. Which AWS implementation is used as the observability backend for traces?

**Options:**

* Bedrock AgentCore Memory
* ADOT Collector alone
* CloudWatch plus X-Ray with Transaction Search enabled
* Amazon S3

✅ **Correct answer:**
**CloudWatch (metrics/logs) plus X-Ray (traces) with Transaction Search enabled**

**Simple explanation:**
CloudWatch handles metrics and logs, while X-Ray helps analyze traces.

---

### 13. What does Evaluation focus on?

**Options:**

* How many tokens were used
* Where time was spent across spans
* Is it happening correctly — judging the journey
* What is happening — seeing the journey

✅ **Correct answer:**
**Is it happening correctly — judging the journey (quality)**

**Simple explanation:**
Observability shows **what happened**. Evaluation checks whether it happened **correctly**.

---

### 14. What is Amazon Bedrock AgentCore Evaluations?

**Options:**

* Dashboard showing only latency and errors
* Self-managed open-source evaluation library
* Fully managed continuous quality assessment service
* Replacement for CloudWatch logs

✅ **Correct answer:**
**A fully managed, continuous quality assessment service for AI agents**

**Simple explanation:**
It helps continuously check whether AI agents are producing good-quality results.

---

### 15. What represents a complete user interaction spanning multiple turns and multiple traces?

**Options:**

* Sub-Span
* Session
* Trace
* Span

✅ **Correct answer:** **Session**

**Simple explanation:**
A session can contain multiple user messages and multiple traces.

---

### 16. What represents a single logical operation within a trace?

**Options:**

* Instrumentation
* Trace
* Session
* Span

✅ **Correct answer:** **Span**

**Simple explanation:**
A span represents one operation, such as:

* model call
* tool call
* retrieval

---

### 17. At which level does Goal Success Rate operate?

**Options:**

* Span level
* Trace level
* Session level
* Sub-span level

✅ **Correct answer:** **Session level**

**Simple explanation:**
Goal Success Rate checks whether the user's overall goal was completed across the whole session.

---

### 18. Which are span/tool-level metrics in AgentCore Evaluation?

**Options:**

* Tool Selection Accuracy and Parameter Accuracy
* Correctness and Helpfulness
* Faithfulness and Coherence
* Goal Success Rate and Custom Metric

✅ **Correct answer:**
**Tool Selection Accuracy and Parameter Accuracy**

**Simple explanation:**
These check whether the agent:

* selected the correct tool
* passed the correct parameters

---

### 19. What is Step 1 when building a custom evaluator using LLM-as-a-Judge?

**Options:**

* Define your criteria
* Select the evaluation model
* Configure sampling
* Write the evaluation prompt

✅ **Correct answer:**
**Define your criteria**

**Simple explanation:**
First decide **what you want to measure**, such as purchase intent, quality, safety, or accuracy.

---

### 20. What are the three things to start with for Minimum Viable Observability?

**Options:**

* Instrument traces/costs/performance metrics, define 3–5 business metrics, and run automated evaluation weekly
* Buy a third-party tool, hire an SRE, and build a data lake
* Disable logging, sample 1% of traces, and review yearly
* Track only GPU temperature, network, and storage

✅ **Correct answer:**
**Instrument traces/costs/performance metrics, define 3–5 custom business-aligned metrics, and run automated evaluation weekly**

**Simple explanation:**
Start with:

1. **Observe technical performance**
2. **Measure business outcomes**
3. **Evaluate agent quality regularly**

### Quick Memory Sheet

**Metrics + Logs + Traces = Telemetry**

**Monitoring = What is wrong?**
**Observability = Why is it happening?**
**Evaluation = Is it happening correctly?**

**Session → Trace → Span**

**Session** = complete conversation
**Trace** = one agent request
**Span** = one model/tool operation

**Trace-level quality:** Faithfulness, Correctness, Helpfulness
**Tool/Span-level:** Tool Selection Accuracy, Parameter Accuracy
**Session-level:** Goal Success Rate
