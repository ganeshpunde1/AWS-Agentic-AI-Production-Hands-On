# 🧠 Amazon EKS & Kubernetes Quiz — All 20 Questions with Answers and Simple Explanations

---

## **Question 1**
**What is Kubernetes used for?**  
- Managing containers  
- Managing virtual machines  
- Managing databases  
- Managing source code  

✅ **Correct Answer:** Managing containers  
**Explanation:** Kubernetes helps run and manage containerized applications automatically — handling scaling, deployment, and recovery.

---

## **Question 2**
**What is Amazon EKS?**  
- A service to manage EC2 instances  
- A managed Kubernetes service on AWS  
- A database service  
- A monitoring tool  

✅ **Correct Answer:** A managed Kubernetes service on AWS  
**Explanation:** EKS makes it easy to run Kubernetes on AWS without managing the control plane yourself.

---

## **Question 3**
**What does a Pod represent in Kubernetes?**  
- A group of containers running together  
- A single virtual machine  
- A database cluster  
- A storage volume  

✅ **Correct Answer:** A group of containers running together  
**Explanation:** A Pod is the smallest deployable unit in Kubernetes — it can contain one or more containers that share resources.

---

## **Question 4**
**What is the role of the Kubernetes Control Plane?**  
- To store application data  
- To manage and schedule Pods across nodes  
- To monitor CPU usage only  
- To handle network traffic  

✅ **Correct Answer:** To manage and schedule Pods across nodes  
**Explanation:** The control plane decides where Pods run and ensures the cluster stays in the desired state.

---

## **Question 5**
**What is a Node in Kubernetes?**  
- A container image  
- A worker machine that runs Pods  
- A database server  
- A network router  

✅ **Correct Answer:** A worker machine that runs Pods  
**Explanation:** Nodes are the actual servers (virtual or physical) that host and run your application Pods.

---

## **Question 6**
**What problem does GPU Time Slicing solve?**  
- It increases GPU clock speed for AI inference  
- ✅ It allows multiple Pods to share a single GPU by taking turns using it over time  
- It migrates GPU workloads between regions automatically  
- It splits a physical GPU into fully isolated hardware partitions  

**Explanation:** GPU Time Slicing lets several Pods use one GPU sequentially, improving resource sharing and efficiency.

---

## **Question 7**
**What is the key difference between Multi-Instance GPU (MIG) and GPU Time Slicing?**  
- Time Slicing can only be used with CPUs  
- MIG is cheaper than Time Slicing  
- MIG is a software feature; Time Slicing is hardware  
- ✅ MIG partitions the GPU into isolated hardware slices; Time Slicing shares the GPU by alternating access over time  

**Explanation:** MIG provides hardware-level isolation, while Time Slicing shares GPU access over time.

---

## **Question 8**
**Which of the following is NOT listed as a challenge in managing Kubernetes?**  
- Networking & multi-AZ resilience  
- Cluster security & compliance  
- Control plane setup & upgrade  
- ✅ Automatic model fine-tuning  

**Explanation:** Model fine-tuning is an AI task, not a Kubernetes management challenge.

---

## **Question 9**
**How does Amazon EKS address the 'Control plane setup & upgrade' challenge?**  
- EKS removes the need for a control plane entirely  
- ✅ EKS runs and manages the control plane with multi-AZ support, auto-patching, and an SLA  
- EKS replaces Kubernetes with a proprietary architecture  
- EKS delegates control plane management to on-premises teams  

**Explanation:** EKS automates control plane management, ensuring reliability and updates without manual work.

---

## **Question 10**
**Which scaling step is recommended first when an application needs more capacity on EKS?**  
- ✅ Scale the workload first (Pod Scaling), then add infrastructure (Node Scaling) if needed  
- Reduce replicas  
- Migrate to serverless  
- Add new GPU nodes immediately  

**Explanation:** Always scale Pods first to use existing node capacity before adding new nodes.

---

## **Question 11**
**What does 'Desired State' mean in Kubernetes?**  
- A snapshot of metrics  
- The maximum number of Pods  
- ✅ The declared configuration of what should be running  
- The current state reported by nodes  

**Explanation:** Desired State is what you want your cluster to look like — Kubernetes keeps it that way automatically.

---

## **Question 12**
**Why are AI agent workloads different from traditional workloads?**  
- ✅ They can be dynamic and long-running, unlike short-lived stateless requests  
- Traditional apps require GPUs but agents don’t  
- Agents use less memory  
- Agents cannot be containerized  

**Explanation:** AI agents often run continuously and adapt over time, unlike short web requests.

---

## **Question 13**
**An AI inference Pod declares `nvidia.com/gpu: 1` but stays pending. Why?**  
- Container image too large  
- ✅ No worker node has a GPU that satisfies the request  
- Missing namespace  
- Control plane unavailable  

**Explanation:** The Pod waits until a GPU-enabled node is available to meet its resource needs.

---

## **Question 14**
**Which TWO ways does Amazon EKS help with Kubernetes challenges?**  
- ✅ Integrates with IAM and is certified for SOC, HIPAA, PCI  
- ✅ Runs and manages the control plane with multi-AZ support and auto-patching  
- Replaces containers with VMs  
- Automatically writes AI code  

**Explanation:** EKS improves security and reliability through IAM integration and managed control plane operations.

---

## **Question 15**
**Which TWO statements about Pod and Node Scaling are correct?**  
- ✅ Dynamic workloads need both layers of autoscaling  
- ✅ Pod and Node scaling are complementary approaches  
- Pod scaling increases node count  
- Node scaling should always happen first  

**Explanation:** Pod scaling handles app-level demand; Node scaling ensures enough infrastructure to support it.

---

## **Question 16**
**Which GPU sharing strategy should be used for isolated workloads?**  
- ✅ Multi-Instance GPU (MIG) — partitions GPU into isolated slices  
- Increase GPU nodes  
- Pod anti-affinity rules  
- GPU Time Slicing  

**Explanation:** MIG provides hardware isolation so workloads don’t affect each other’s GPU memory.

---

## **Question 17**
**What happens when a worker node fails in a Kubernetes cluster?**  
- ✅ Kubernetes reschedules the missing Pod on a healthy node  
- Reduces replica count  
- Scales down remaining Pods  
- Waits for manual recovery  

**Explanation:** Kubernetes automatically moves Pods to healthy nodes to maintain the desired state.

---

## **Question 18**
**Which EKS feature provisions GPU nodes only when needed?**  
- AWS Fargate  
- EKS Control Plane autoscaling  
- Managed Node Groups  
- ✅ Karpenter  

**Explanation:** Karpenter adds or removes nodes dynamically based on pending Pod requirements — perfect for AI workloads.

---

## **Question 19**
**Which Agentic AI pattern gives full control over model serving stack?**  
- ✅ Strategy 1: Self-Managed on Kubernetes — uses vLLM on EKS with Inferentia/GPU  
- Strategy 2: Fully Managed via Bedrock  
- Strategy 3: SageMaker endpoint  
- Strategy 4: Integrated via LiteLLM proxy  

**Explanation:** Self-managed Kubernetes gives total control over infrastructure and model customization.

---

## **Question 20**
**An application has high CPU usage but spare node capacity. What should you do?**  
- ✅ Scale out Pods (HPA) to use remaining node capacity  
- Add more nodes  
- Reduce Pods  
- Switch to serverless  

**Explanation:** Use Horizontal Pod Autoscaler to add more Pods and fully utilize existing resources before scaling nodes.

---

### 🏁 **Summary**
Amazon EKS simplifies Kubernetes by automating control plane management, scaling, and GPU resource handling.  
Understanding **Pods, Nodes, Scaling, and GPU strategies** helps optimize AI and cloud workloads efficiently.
