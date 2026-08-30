# Serverless AI Quiz – 15 Questions with Answers

## 1. Why do AI workloads benefit from serverless architectures?

**Question:** What is the primary reason AI workloads benefit from serverless architectures?

**Options:**
- A. AI models require dedicated GPU clusters at all times
- B. AI models cannot run on cloud infrastructure at all
- C. **AI workloads are intermittent, unpredictable and bursty, leading to overprovisioned infrastructure in traditional setups**
- D. Serverless is cheaper than all other deployment models regardless of workload

✅ **Correct Answer: C**

**Simple explanation:** AI traffic can suddenly go up or down. Serverless automatically scales based on demand, so you don't need to keep unused servers running.

---

## 2. Which layer prepares data before AI inference?

**Question:** In the Serverless AI application architecture, which layer is responsible for preparing and enriching incoming data before it reaches the AI model?

**Options:**
- A. Inference Layer
- B. Post-Processing / Decisioning Layer
- C. Event Trigger / Interface Layer
- D. **Processing Layer**

✅ **Correct Answer: D — Processing Layer**

**Simple explanation:** The Processing Layer cleans, transforms, validates, and prepares the data before giving it to the AI model.

---

## 3. Which AWS service connects agents asynchronously?

**Question:** Which AWS service is described as ideal for agentic AI because it provides event routing to trigger and connect agents asynchronously?

**Options:**
- A. AWS Step Functions
- B. Amazon SQS
- C. AWS Lambda
- D. **Amazon EventBridge**

✅ **Correct Answer: D — Amazon EventBridge**

**Simple explanation:** EventBridge receives events and sends them to the correct services or agents without requiring them to communicate directly.

---

## 4. Which service orchestrates the Legal Document Ingestor workflow?

**Question:** In the Legal Document Ingestor use case, which AWS service orchestrates the end-to-end multi-step workflow?

**Options:**
- A. **AWS Step Functions**
- B. Amazon EventBridge
- C. Amazon Textract
- D. Amazon Bedrock

✅ **Correct Answer: A — AWS Step Functions**

**Simple explanation:** Step Functions manages multiple steps in a workflow and decides which step should run next.

---

## 5. Which design consideration includes DLQs?

**Question:** Which design consideration recommends using dead-letter queues (DLQs) on Lambda to allow each layer to fail and retry independently?

**Options:**
- A. Observability
- B. **Resilience**
- C. Extensibility
- D. Cost optimization

✅ **Correct Answer: B — Resilience**

**Simple explanation:** DLQs save failed messages so they can be retried later without breaking the whole application.

---

## 6. Which AWS service runs ML models locally on edge devices?

**Question:** For edge inference in the Factory Equipment Monitoring use case, which AWS runtime executes the ML model locally on the device?

**Options:**
- A. Amazon Bedrock
- B. **AWS IoT Greengrass**
- C. Amazon SageMaker Serverless Inference
- D. AWS Lambda@Edge

✅ **Correct Answer: B — AWS IoT Greengrass**

**Simple explanation:** IoT Greengrass lets applications and ML models run directly on local devices instead of always sending data to AWS.

---

## 7. Which category includes AWS CDK, CloudFormation, and SAM?

**Question:** What category in the Implementation Strategies table covers tools like AWS CDK, CloudFormation, and AWS SAM for managing serverless AI deployments?

**Options:**
- A. CI/CD & Automation
- B. Testing & Validation
- C. Prompt, Agent & Model Lifecycle Management
- D. **Infrastructure as Code (IaC)**

✅ **Correct Answer: D — Infrastructure as Code (IaC)**

**Simple explanation:** These tools let you create AWS infrastructure using code or templates instead of manually creating resources.

---

## 8. Which layer stores AI results?

**Question:** Which layer in the Serverless AI architecture delivers AI results and preserves them for future use, analysis, and improvement?

**Options:**
- A. Inference Layer
- B. Post-Processing / Decisioning Layer
- C. Processing Layer
- D. **Output / Storage Layer**

✅ **Correct Answer: D — Output / Storage Layer**

**Simple explanation:** This layer sends the final result to users or applications and stores it for later use.

---

## 9. Which layers are part of Serverless AI architecture?

**Question:** Which layers are part of the Serverless AI Application Architecture? *(Select ALL that apply)*

**Options:**
- A. **Event Trigger / Interface Layer**
- B. **Post-Processing / Decisioning Layer**
- C. Database Layer
- D. **Processing Layer**

✅ **Correct Answers: A, B, D**

**Simple explanation:** Event Trigger receives requests or events, Processing prepares data, and Post-Processing handles results and decisions. Database Layer is not listed as one of the architecture layers here.

---

## 10. Which services are useful for event-driven Serverless AI?

**Question:** Which of the following are useful AWS services for implementing Event-Driven Architecture in Serverless AI? *(Select TWO)*

**Options:**
- A. **Amazon EventBridge**
- B. Amazon SageMaker Serverless Inference
- C. **Amazon SQS**
- D. Amazon OpenSearch Service

✅ **Correct Answers: A and C**

**Simple explanation:** EventBridge routes events. SQS stores messages in queues so services can process them asynchronously.

---

## 11. What are Serverless AI design considerations?

**Question:** Which of the following are design considerations explicitly mentioned for Serverless AI architectures across layers? *(Select ALL that apply)*

**Options:**
- A. **Observability**
- B. **Resilience**
- C. **Cost optimization**
- D. Portability

✅ **Correct Answers: A, B, C**

**Simple explanation:** Observability helps you see what is happening, resilience helps recover from failures, and cost optimization avoids unnecessary spending.

---

## 12. Where do Bedrock Guardrails and AgentCore Policy belong?

**Question:** Which Implementation Strategy category explicitly mentions Amazon Bedrock Guardrails or Bedrock AgentCore Policy?

**Options:**
- A. **Security & Governance**
- B. Infrastructure as Code
- C. Testing & Validation
- D. Cost Optimization

✅ **Correct Answer: A — Security & Governance**

**Simple explanation:** Guardrails and policies control what AI systems and agents are allowed to do and help keep them safe.

---

## 13. Which services handle the Output / Storage layer in Factory Equipment Monitoring?

**Question:** In the Factory Equipment Monitoring use case, which two AWS services handle the Output / Storage layer? *(Select TWO)*

**Options:**
- A. **AWS IoT Core**
- B. Amazon Textract
- C. **Amazon EventBridge**
- D. AWS Lambda

✅ **Correct Answers: A and C**

**Simple explanation:** IoT Core handles communication with IoT devices, while EventBridge routes events and results to other systems.

---

## 14. Which services are used for Observability & Monitoring?

**Question:** Which of the following are listed under Observability & Monitoring in the Implementation Strategies table? *(Select ALL that apply)*

**Options:**
- A. **AWS X-Ray**
- B. AWS CodePipeline
- C. **Amazon CloudWatch Logs & Metrics**
- D. **Bedrock agent trace / model invocation logging**

✅ **Correct Answers: A, C, D**

**Simple explanation:** X-Ray traces requests, CloudWatch stores logs and metrics, and Bedrock tracing shows what agents and models are doing. CodePipeline is for CI/CD.

---

## 15. What are two core Serverless AI design questions?

**Question:** Which of the following are Core Principles questions that must be answered when building Serverless AI? *(Select TWO)*

**Options:**
- A. Which programming language should be used for Lambda?
- B. **How should orchestration be handled?**
- C. What is the monthly AWS bill?
- D. **Where should inference happen?**

✅ **Correct Answers: B and D**

**Simple explanation:** You need to decide how services and agents will work together and where the AI model should run.

---

## Quick Answer Key

| # | Correct Answer |
|---|---|
| 1 | C |
| 2 | D — Processing Layer |
| 3 | D — Amazon EventBridge |
| 4 | A — AWS Step Functions |
| 5 | B — Resilience |
| 6 | B — AWS IoT Greengrass |
| 7 | D — Infrastructure as Code |
| 8 | D — Output / Storage Layer |
| 9 | A, B, D |
| 10 | A, C |
| 11 | A, B, C |
| 12 | A — Security & Governance |
| 13 | A, C |
| 14 | A, C, D |
| 15 | B, D |


https://github.com/aws-samples/sample-getting-started-with-strands-agents-course#-course-2-advanced-strands-agents-with-mcp