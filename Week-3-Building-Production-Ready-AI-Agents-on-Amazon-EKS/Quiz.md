#  🧠 Amazon EKS, Kubernetes, and AI Workloads — 20 Questions

## Question 1 of 20

**What is the primary benefit of using containers for AI applications?**

### Options
1. They replace the need for a container orchestrator
2. They automatically tune model hyperparameters
3. **✅They package complex, rapidly changing software dependencies into a consistent, portable deployment unit**
4. They eliminate the need for any hardware

### Correct Answer
**They package complex, rapidly changing software dependencies into a consistent, portable deployment unit**

### Simple Explanation
AI applications usually need Python libraries, AI frameworks, model dependencies, and system packages.

A container packages all of these together so the application works the same way everywhere.

**Simple idea:**

`AI App + Python + Libraries + Dependencies → Container`

Think of a container as a ready-to-run box containing everything the AI application needs.

---

## Question 2 of 20

**What is Kubernetes?**

### Options
1. A serverless compute service
2. A managed cloud database service
3. **✅An open-source platform for orchestrating containerized applications**
4. A GPU hardware accelerator from AWS

### Correct Answer
**An open-source platform for orchestrating containerized applications**

### Simple Explanation
Kubernetes helps manage containers automatically.

It can start containers, stop containers, scale them, restart failed containers, and distribute traffic.

**Simple idea:**

`Many Containers → Kubernetes → Manages them automatically`

Think of Kubernetes as a manager for containers.

---

## Question 3 of 20

**✅In the airport analogy for a Kubernetes cluster, what does the Air Traffic Controller represent?**

### Options
1. Pods
2. Applications inside a Pod
3. **The Control Plane**
4. Worker Nodes

### Correct Answer
**The Control Plane**

### Simple Explanation
The Control Plane manages and coordinates the whole Kubernetes cluster.

Just like an Air Traffic Controller manages airplanes, the Kubernetes Control Plane manages Pods and Worker Nodes.

**Simple idea:**

`Air Traffic Controller → manages airplanes`

`Kubernetes Control Plane → manages Pods and Worker Nodes`

Think of the Control Plane as the brain of Kubernetes.

---

## Question 4 of 20

**In a Kubernetes cluster, what is a Pod?**

### Options
1. The component that coordinates and manages the cluster
2. A type of storage volume attached to worker nodes
3. The hardware server providing compute capacity
4. **✅The smallest deployable unit that runs the actual application workload**

### Correct Answer
**The smallest deployable unit that runs the actual application workload**

### Simple Explanation
A Pod is the smallest unit Kubernetes deploys.

A Pod usually contains one application container, although it can contain multiple closely related containers.

**Simple idea:**

`Kubernetes → Pod → Container → Application`

Think of a Pod as a small box that holds and runs your application container.

---

## Question 5 of 20

**How does Kubernetes handle GPU resource placement for a Pod?**

### Options
1. AWS automatically assigns GPUs without any Kubernetes configuration
2. The developer manually assigns a GPU server IP in the deployment config
3. **✅The manifest declares the GPU resource requirement and Kubernetes selects a suitable worker node**
4. The application code explicitly picks a GPU worker node at runtime

### Correct Answer
**The manifest declares the GPU resource requirement and Kubernetes selects a suitable worker node**

### Simple Explanation
The Pod configuration tells Kubernetes that the application needs a GPU.

Kubernetes then looks for a worker node with an available GPU and schedules the Pod there.

**Simple flow:**

`Pod requests GPU → Kubernetes Scheduler checks nodes → GPU-capable node selected → Pod runs`

---

## Question 6 of 20

**What problem does GPU Time Slicing solve?**

### Options
1. It increases GPU clock speed for AI inference
2. **✅It allows multiple Pods to share a single GPU by taking turns using it over time**
3. It migrates GPU workloads between regions automatically
4. It splits a physical GPU into fully isolated hardware partitions

### Correct Answer
**It allows multiple Pods to share a single GPU by taking turns using it over time**

### Simple Explanation
GPU Time Slicing lets multiple Pods use the same GPU by taking turns very quickly.

**Simple idea:**

`Pod A → GPU → Pod B → GPU → Pod C → GPU`

Think of it like several people sharing one computer by taking turns.

---

## Question 7 of 20

**What is the key difference between Multi-Instance GPU (MIG) and GPU Time Slicing?**

### Options
1. Time Slicing can only be used with CPUs, not GPUs
2. MIG is cheaper than Time Slicing
3. MIG is a software feature; Time Slicing is a hardware feature
4. **✅MIG partitions the GPU into isolated hardware slices; Time Slicing shares the GPU by alternating access over time**

### Correct Answer
**MIG partitions the GPU into isolated hardware slices; Time Slicing shares the GPU by alternating access over time**

### Simple Explanation
MIG divides one physical GPU into separate isolated GPU sections.

Time Slicing does not divide the GPU. Multiple workloads simply take turns using it.

**Simple idea:**

`MIG → GPU divided into separate pieces`

`Time Slicing → One GPU shared by taking turns`

Think of MIG as separate apartments in one building and Time Slicing as people sharing the same room at different times.

---

## Question 8 of 20

**Which of the following is NOT listed as a challenge in managing Kubernetes?**

### Options
1. Networking & multi-AZ resilience
2. **✅Automatic model fine-tuning**
3. Cluster security & compliance
4. Control plane setup & upgrade

### Correct Answer
**Automatic model fine-tuning**

### Simple Explanation
Kubernetes management challenges include networking, security, availability, control plane setup, and upgrades.

Automatic model fine-tuning is an AI/ML task, not a Kubernetes management challenge.

**Simple idea:**

`Kubernetes → manages infrastructure and containers`

`Model fine-tuning → improves an AI model`

---

## Question 9 of 20

**How does Amazon EKS address the "Control plane setup & upgrade" challenge of Kubernetes?**

### Options
1. **✅EKS runs and manages the control plane with multi-AZ support, auto-patching, and an SLA, so teams just consume the API**
2. EKS removes the need for a control plane entirely
3. EKS replaces Kubernetes with a proprietary orchestrator
4. EKS delegates control plane management to the customer's on-premises team

### Correct Answer
**EKS runs and manages the control plane with multi-AZ support, auto-patching, and an SLA, so teams just consume the API**

### Simple Explanation
With Amazon EKS, AWS manages the Kubernetes control plane.

AWS handles areas such as availability, multi-AZ support, patching, maintenance, and reliability.

**Simple idea:**

`You manage applications and workloads`

`AWS EKS manages the Kubernetes control plane`

---

## Question 10 of 20

**Which of the following scaling approaches is recommended as the FIRST step when an agentic application needs more capacity on EKS?**

### Options
1. Reduce the number of replicas to free resources
2. **✅Scale the workload first (Pod Scaling), then add infrastructure (Node Scaling) if more capacity is needed**
3. Migrate the workload to a serverless function
4. Immediately provision new GPU worker nodes

### Correct Answer
**Scale the workload first (Pod Scaling), then add infrastructure (Node Scaling) if more capacity is needed**

### Simple Explanation
When your application needs more capacity, first increase the number of Pods.

If the existing worker nodes do not have enough CPU, memory, or GPU resources, then add more Nodes.

**Simple flow:**

`More traffic → Add Pods → If nodes are full → Add Nodes`

---

## Question 11 of 20

**What does 'Desired State' mean in the context of Kubernetes availability?**

### Options
1. A read-only snapshot of cluster metrics
2. The maximum number of Pods that can ever be scheduled on a cluster
3. **✅The declared configuration of what should be running, which Kubernetes continuously works to maintain**
4. The current actual state reported by worker nodes

### Correct Answer
**The declared configuration of what should be running, which Kubernetes continuously works to maintain**

### Simple Explanation
Desired State means how you want your Kubernetes application to look and run.

For example:

`Desired = 3 Pods`

If one crashes:

`Actual = 2 Pods`

Kubernetes starts another Pod so that:

`Actual = 3 Pods`

**Desired State = What you want running**

**Actual State = What is running right now**

---

## Question 12 of 20

**✅Why are AI agent workloads considered different from traditional application workloads on Kubernetes?**

### Options
1. **Agent workloads can be dynamic and long-running, unlike typical short-lived stateless requests**
2. Traditional applications require GPUs but agents do not
3. Agent workloads always use less memory than traditional applications
4. Agents cannot be containerized

### Correct Answer
**Agent workloads can be dynamic and long-running, unlike typical short-lived stateless requests**

### Simple Explanation
Traditional applications often handle a request quickly and return a response.

AI agents may think, call tools, query databases, call APIs, wait for results, and continue processing.

**Simple idea:**

`Normal app → Request → Response`

`AI Agent → Think → Call tool → Check result → Think again → Respond`

---

## Question 13 of 20

**An AI inference Pod on EKS declares `nvidia.com/gpu: 1` in its manifest but remains in Pending state even though there are worker nodes in the cluster. What is the most likely cause?**

### Options
1. The Pod's container image is too large to pull
2. **✅None of the existing worker nodes have a GPU that satisfies the Pod's resource request**
3. The Pod manifest is missing a namespace declaration
4. The Kubernetes Control Plane is unavailable

### Correct Answer
**None of the existing worker nodes have a GPU that satisfies the Pod's resource request**

### Simple Explanation
The Pod is asking Kubernetes for one GPU.

Kubernetes can schedule it only on a worker node with an available GPU.

If no suitable GPU is available, the Pod stays in Pending state.

**Simple flow:**

`Pod requests GPU → Kubernetes checks nodes → No available GPU → Pod stays Pending`

---

## Question 14 of 20

**Which of the following are ways Amazon EKS helps with Kubernetes challenges? (Select TWO)**

### Options
1. **✅EKS integrates with IAM and is certified for standards like SOC, HIPAA, and PCI**
2. **✅EKS runs and manages the control plane with multi-AZ support and auto-patching**
3. EKS replaces containers with virtual machines for AI workloads
4. EKS automatically writes and deploys application code for AI agents

### Correct Answers
**1. EKS integrates with IAM and is certified for standards like SOC, HIPAA, and PCI**

**2. EKS runs and manages the control plane with multi-AZ support and auto-patching**

### Simple Explanation
Amazon EKS helps with security, compliance, reliability, and Kubernetes control plane management.

**Simple idea:**

`IAM + compliance → Security`

`Managed control plane + Multi-AZ + patching → Reliability and maintenance`

---

## Question 15 of 20

**Which of the following statements about Pod Scaling and Node Scaling on EKS are correct? (Select TWO)**

### Options
1. **✅Dynamic or unpredictable workloads often need both layers of autoscaling**
2. **✅Pod and Node scaling are complementary, not competing approaches**
3. Pod scaling increases the number of physical worker nodes
4. Node scaling should always be triggered before Pod scaling

### Correct Answers
**1. Dynamic or unpredictable workloads often need both layers of autoscaling**

**2. Pod and Node scaling are complementary, not competing approaches**

### Simple Explanation
Pod Scaling changes the number of Pods.

Node Scaling changes the number of worker nodes.

They work together.

**Simple flow:**

`More traffic → More Pods → If nodes are full → More Nodes`

---

## Question 16 of 20

**A machine learning team has three inference workloads with strict latency SLAs that must not be affected by each other's GPU memory usage. Which GPU sharing strategy should they use on their Kubernetes nodes?**

### Options
1. **✅Multi-Instance GPU (MIG) — it partitions the GPU into isolated hardware slices with dedicated memory**
2. Increase the number of GPU nodes so each workload gets its own node
3. Pod anti-affinity rules to place workloads on separate CPU nodes
4. GPU Time Slicing — it alternates GPU access so workloads never overlap

### Correct Answer
**Multi-Instance GPU (MIG) — it partitions the GPU into isolated hardware slices with dedicated memory**

### Simple Explanation
The key requirement is isolation.

MIG divides one physical GPU into separate hardware sections. Each workload gets its own GPU slice and dedicated memory.

**Simple idea:**

`1 GPU → MIG Slice 1 + MIG Slice 2 + MIG Slice 3`

**MIG = better isolation and predictable performance**

**Time Slicing = shared GPU, less isolation**

---

## Question 17 of 20

**A Kubernetes cluster is running 3 replicas of an AI agent Pod. A worker node fails, reducing the actual count to 2. Without any manual intervention, what does Kubernetes do?**

### Options
1. Kubernetes automatically reduces the desired replica count to 2 to match the actual state
2. **✅Kubernetes continuously reconciles and reschedules the missing Pod on a healthy worker node**
3. Kubernetes scales down the remaining 2 Pods to prevent resource contention
4. Kubernetes waits for the failed node to recover before rescheduling the Pod

### Correct Answer
**Kubernetes continuously reconciles and reschedules the missing Pod on a healthy worker node**

### Simple Explanation
Kubernetes knows the desired state is 3 Pods.

After the failure, the actual state becomes 2 Pods.

Kubernetes detects the mismatch and tries to restore the desired state.

**Simple flow:**

`Desired = 3 Pods`

`Actual = 2 Pods`

`Kubernetes detects mismatch → starts 1 new Pod`

`Actual = 3 Pods again`

This automatic correction is called **reconciliation**.

---

## Question 18 of 20

**An AI team on Amazon EKS wants to automatically provision GPU nodes only when a GPU-requesting Pod is pending, and remove them when idle. Which EKS feature best addresses this requirement?**

### Options
1. AWS Fargate — it provisions serverless compute for each Pod
2. EKS Control Plane autoscaling
3. Managed Node Groups with manual scaling policies
4. **✅Karpenter — it automates just-in-time node provisioning and removal based on pending Pod requirements**

### Correct Answer
**Karpenter — it automates just-in-time node provisioning and removal based on pending Pod requirements**

### Simple Explanation
Karpenter watches for Pods that cannot run because there is not enough suitable compute.

If a Pod needs a GPU and no current node can run it:

`Pending GPU Pod → Karpenter creates GPU node → Pod runs`

When the node is no longer needed:

`Node idle → Karpenter can remove it`

So Karpenter helps with automatic provisioning, removal, and cost savings.

---

## Question 19 of 20

**A team needs full control over their model serving stack, wants to use a custom open-source LLM on Inferentia chips, and is comfortable with high operational overhead. Which Agentic AI pattern on EKS should they choose?**

### Options
1. **✅Strategy 1: Self-Managed on Kubernetes — uses vLLM on EKS with Inferentia/GPU**
2. Strategy 3: Fully Managed — uses Amazon Bedrock for all components
3. Strategy 2 with a custom SageMaker endpoint replacing Bedrock
4. Strategy 2: Integrated — uses Amazon Bedrock via LiteLLM proxy

### Correct Answer
**Strategy 1: Self-Managed on Kubernetes — uses vLLM on EKS with Inferentia/GPU**

### Simple Explanation
The team wants full control, a custom open-source LLM, Inferentia or GPU, and is comfortable managing the infrastructure.

That matches a self-managed Kubernetes strategy.

**Simple idea:**

`Custom LLM + EKS + Inferentia/GPU + Full control = Self-Managed`

**More control → More operational work**

---

## Question 20 of 20

**An agentic application on EKS is experiencing increased request volume. All existing Pods are at high CPU utilization but there is still spare capacity on the worker nodes. What is the correct scaling action?**

### Options
1. Immediately add more worker nodes to increase infrastructure capacity
2. **✅Scale out the Pods (Pod Scaling / HPA) to utilise the remaining worker node capacity**
3. Reduce the number of Pods to free up CPU per Pod
4. Switch to a serverless function to avoid capacity planning altogether

### Correct Answer
**Scale out the Pods (Pod Scaling / HPA) to utilise the remaining worker node capacity**

### Simple Explanation
The worker nodes still have free CPU capacity, but the existing Pods are overloaded.

So Kubernetes should create more Pods first.

**Simple flow:**

`More requests → Existing Pods busy → Worker nodes still have space → Add more Pods`

This is commonly handled by:

**HPA = Horizontal Pod Autoscaler**

**Free node capacity available → Scale Pods first**

**Nodes full → Then scale Nodes**

---

# Quick Review

- **Containers** package applications and dependencies into portable units.
- **Kubernetes** orchestrates containerized applications.
- **Control Plane** manages the Kubernetes cluster.
- **Pods** are the smallest deployable units.
- **GPU requests** are declared in Pod manifests.
- **GPU Time Slicing** lets workloads take turns on one GPU.
- **MIG** creates isolated GPU hardware slices.
- **Amazon EKS** manages the Kubernetes control plane.
- **Pod Scaling** should usually happen before Node Scaling when node capacity exists.
- **Desired State** is what Kubernetes continuously tries to maintain.
- **Karpenter** can provision and remove nodes automatically.
- **HPA** scales Pods based on workload metrics such as CPU utilization.
