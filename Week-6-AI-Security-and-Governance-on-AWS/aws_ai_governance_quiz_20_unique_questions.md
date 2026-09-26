# AWS AI Governance Quiz — 20 Unique Questions

## Question 1 of 20

**The deck maps AI-workload access needs to AWS capabilities. Which of these pairings are correct? (Select ALL that apply)**

- [ ] Define what the workload can access → Amazon Bedrock Guardrails
- [x] Retrieve application secrets securely → AWS Secrets Manager
- [x] Avoid embedded long-term credentials → Temporary credentials / AWS STS
- [x] Protect encryption keys and encrypted data → AWS KMS

**Correct answer:** Retrieve application secrets securely → AWS Secrets Manager; Avoid embedded long-term credentials → Temporary credentials / AWS STS; Protect encryption keys and encrypted data → AWS KMS

---

## Question 2 of 20

**Which controls does the deck list for enforcing approved-model governance in Amazon Bedrock? (Select ALL that apply)**

- [x] Service Control Policies (SCPs) to enforce across many accounts
- [ ] An Amazon S3 bucket policy that stores the approved model weights
- [x] IAM policies scoped to specific Bedrock model ARNs
- [x] A bedrock:GuardrailIdentifier condition to require an approved guardrail

**Correct answer:** Service Control Policies (SCPs) to enforce across many accounts; IAM policies scoped to specific Bedrock model ARNs; A bedrock:GuardrailIdentifier condition to require an approved guardrail

---

## Question 3 of 20

**According to the Amazon Bedrock Guardrails diagram, which of the following are guardrail policy components? (Select ALL that apply)**

- [x] Denied topics
- [ ] Automatic model fine-tuning
- [x] Content filters
- [x] Sensitive information filter

**Correct answer:** Denied topics; Content filters; Sensitive information filter

---

## Question 4 of 20

**The deck maps agent business needs to AgentCore Identity capabilities. Which pairings are correct? (Select ALL that apply)**

- [x] Manage identities centrally → Agent Identity Directory
- [ ] Continuously assess compliance → AWS Config conformance packs
- [x] Secure credentials → Credential providers / Token Vault
- [x] Access external services → OAuth 2.0 / API keys

**Correct answer:** Manage identities centrally → Agent Identity Directory; Secure credentials → Credential providers / Token Vault; Access external services → OAuth 2.0 / API keys

---

## Question 5 of 20

**How does the deck define AI Governance?**

- [x] How an organization decides which AI it will build or use, under what rules, who is accountable, and how it keeps systems lawful and fit for purpose
- [ ] A billing framework that allocates the cost of AI model inference across the different teams in an organization
- [ ] A model training technique that improves accuracy by testing answers against a set of approved reference scenarios
- [ ] A single AWS service that automatically blocks any AI model output judged unsafe before it reaches the end customer

**Correct answer:** How an organization decides which AI it will build or use, under what rules, who is accountable, and how it keeps systems lawful and fit for purpose

---

## Question 6 of 20

**An AWS Control Tower landing zone provisions baseline accounts for governance. Which of the following are accounts/capabilities the deck associates with this setup? (Select ALL that apply)**

- [x] Centralized CloudTrail and AWS Config logs
- [x] Audit account
- [x] Log Archive account
- [ ] A dedicated GPU training cluster account

**Correct answer:** Centralized CloudTrail and AWS Config logs; Audit account; Log Archive account

---

## Question 7 of 20

**A company is setting up a new multi-account AWS environment and wants a governed landing zone with a Log Archive and Audit account before any AI workloads are deployed. Which service is designed to set this up?**

- [x] AWS Control Tower, which sets up and governs a multi-account landing zone with baseline accounts and controls
- [ ] IAM Access Analyzer, which reports on which principals outside the boundary can reach a given resource
- [ ] AWS Secrets Manager, which stores and rotates the application secrets that the AI workloads need at runtime
- [ ] Amazon Bedrock Guardrails, which applies content and topic filters to every model response the workload returns

**Correct answer:** AWS Control Tower, which sets up and governs a multi-account landing zone with baseline accounts and controls

---

## Question 8 of 20

**An AI workload needs to access data, APIs, models, and secrets — but only what its job requires. Which AWS capability gives the workload an identity to assume?**

- [ ] An AWS KMS key, which protects the encryption keys and the encrypted data the workload relies on
- [x] An IAM Role, which gives the workload an assumable identity scoped to the resources it needs
- [ ] A Bedrock guardrail, which screens the prompts and responses exchanged during model inference calls
- [ ] A Service Control Policy, which sets the maximum available permissions across the accounts in an org

**Correct answer:** An IAM Role, which gives the workload an assumable identity scoped to the resources it needs

---

## Question 9 of 20

**In the deck, which IAM Access Analyzer capability answers the governance question 'Which permissions are no longer being used?'**

- [ ] External access analysis, which identifies principals outside the boundary that can reach a resource
- [x] Unused access analysis, which highlights permissions that are no longer being exercised
- [ ] Internal access analysis, which shows who inside the organization is able to reach a resource
- [ ] Policy validation, which flags whether an IAM policy is overly permissive or otherwise invalid

**Correct answer:** Unused access analysis, which highlights permissions that are no longer being exercised

---

## Question 10 of 20

**The deck presents a model selection lifecycle. What is the correct order of its five steps?**

- [ ] Assess → Discover → Approve → Test → Enforce
- [x] Discover → Assess → Test → Approve → Enforce
- [ ] Enforce → Approve → Test → Assess → Discover
- [ ] Discover → Test → Assess → Enforce → Approve

**Correct answer:** Discover → Assess → Test → Approve → Enforce

---

## Question 11 of 20

**Since all serverless Bedrock models are enabled in every account by default, a team wants to make sure only approved models can actually be invoked. Which lifecycle step achieves this?**

- [ ] Discover, which lists the models that are currently available in the Bedrock model catalog
- [x] Enforce, which uses IAM policies and SCPs to limit which models can actually be invoked
- [ ] Assess, which reviews the provider, Region, terms, and lifecycle status of each model
- [ ] Test, which checks whether a model actually meets the intended business use case

**Correct answer:** Enforce, which uses IAM policies and SCPs to limit which models can actually be invoked

---

## Question 12 of 20

**A governance team must block one specific Bedrock model everywhere while allowing the rest. Which control matches this need?**

- [x] A policy that denies bedrock:InvokeModel / InvokeModelWithResponseStream for that model
- [ ] An IAM policy scoped to allow only the specific Bedrock model ARNs that were approved
- [ ] A bedrock:GuardrailIdentifier condition that requires an approved guardrail during inference
- [ ] An aws-marketplace:ProductId condition that restricts which Marketplace subscriptions are allowed

**Correct answer:** A policy that denies bedrock:InvokeModel / InvokeModelWithResponseStream for that model

---

## Question 13 of 20

**Why does the deck recommend Service Control Policies (SCPs) for enforcing approved-model decisions?**

- [ ] They generate least-privilege policies automatically from recent CloudTrail activity logs
- [ ] They screen each prompt and response for unsafe content during the model inference call
- [ ] They store the approved-model list as an encrypted secret that applications read at runtime
- [x] They enforce the restriction consistently across many AWS accounts in the organization

**Correct answer:** They enforce the restriction consistently across many AWS accounts in the organization

---

## Question 14 of 20

**According to the deck, why are Guardrails still needed even when developers use an approved model?**

- [ ] An approved model consumes far more tokens, so guardrails are applied mainly to control inference costs
- [ ] An approved model cannot be invoked at all until a guardrail identifier is attached to the account
- [x] An approved model can still receive unsafe input or return responses that violate safety, privacy, content, or grounding policies
- [ ] An approved model must be re-tested weekly, and guardrails are the mechanism that runs those tests

**Correct answer:** An approved model can still receive unsafe input or return responses that violate safety, privacy, content, or grounding policies

---

## Question 15 of 20

**A team runs a foundation model that is hosted outside Amazon Bedrock but wants to apply the same Bedrock Guardrails to it. Which capability supports this?**

- [ ] The bedrock:GuardrailIdentifier SCP condition, which requires a guardrail on approved model ARNs
- [ ] The Model Invocation Logging feature, which records the input and output of each interaction
- [ ] The InvokeModel API, which sends the prompt directly to the in-Bedrock model for inference
- [x] The ApplyGuardrail API, which can evaluate content for FMs outside Bedrock

**Correct answer:** The ApplyGuardrail API, which can evaluate content for FMs outside Bedrock

---

## Question 16 of 20

**Why does the deck argue an AI agent needs a distinct identity rather than only a static IAM role?**

- [ ] A static IAM role is billed per request, so a distinct agent identity is introduced to reduce those costs
- [x] Agents often act for different users across trust domains, so authorization may need both agent identity and user context
- [ ] A static IAM role cannot be assigned any IAM policies, so an agent would have no permissions at all
- [ ] An agent identity is required purely so that its inference responses can be filtered by guardrail policies

**Correct answer:** Agents often act for different users across trust domains, so authorization may need both agent identity and user context

---

## Question 17 of 20

**An AI agent must reach a CRM, a refund API, Lambda, a database, and SaaS tools, but giving it direct access to every backend creates governance risk. Which AgentCore capability centralizes and limits this access?**

- [x] AgentCore Gateway, which exposes only approved tools through a single governed endpoint
- [ ] AWS Config conformance packs, which continuously check whether the deployed controls are working
- [ ] AgentCore Token Vault, which stores the credentials the agent uses when it calls each service
- [ ] Bedrock Model Invocation Logging, which records what input was sent and what output was returned

**Correct answer:** AgentCore Gateway, which exposes only approved tools through a single governed endpoint

---

## Question 18 of 20

**In the deck, what is the difference between AWS CloudTrail and Bedrock Model Invocation Logging?**

- [ ] CloudTrail stores application secrets securely; Model Invocation Logging rotates the encryption keys in KMS
- [ ] CloudTrail records the model's input and output; Model Invocation Logging records which IAM principal called an API
- [ ] CloudTrail enforces which models can be invoked; Model Invocation Logging blocks any unsafe response content
- [x] CloudTrail records who called which AWS API; Model Invocation Logging records what happened inside the model interaction

**Correct answer:** CloudTrail records who called which AWS API; Model Invocation Logging records what happened inside the model interaction

---

## Question 19 of 20

**A governance team wants to continuously track resource configuration and evaluate whether resources stay compliant after deployment. Which service does the deck point to?**

- [ ] Amazon Bedrock evaluations, which check whether a model meets the intended business use case
- [ ] AWS Control Tower, which sets up the governed landing zone and its baseline accounts up front
- [x] AWS Config, which continuously tracks configuration and evaluates ongoing compliance
- [ ] IAM Identity Center, which centralizes workforce access and permission sets across accounts

**Correct answer:** AWS Config, which continuously tracks configuration and evaluates ongoing compliance

---

## Question 20 of 20

**During an audit, a team must show evidence that their deployed controls are actually working. Which capability is designed to provide that evidence?**

- [ ] AWS Secrets Manager, which retrieves and rotates the application secrets a workload depends on
- [x] AWS Config conformance packs, which continuously check controls and give auditors evidence
- [ ] Amazon Bedrock Guardrails, which apply content and topic filters during each model inference call
- [ ] AgentCore Observability, which captures the sessions, traces, and spans of an agent's execution

**Correct answer:** AWS Config conformance packs, which continuously check controls and give auditors evidence
