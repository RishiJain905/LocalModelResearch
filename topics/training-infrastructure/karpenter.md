# Karpenter Autoscaler

## 📌 Overview

**Karpenter** is an open-source, high-performance Kubernetes node autoscaler built for flexibility, performance, and simplicity. It dynamically provisions and terminates nodes based on actual workload demand, improving cluster efficiency and cost-of-workload-running on Kubernetes.

> "Karpenter is an open-source node lifecycle management project built for Kubernetes. Adding Karpenter to a Kubernetes cluster can dramatically improve the efficiency and cost of running workloads on that cluster."
> — [Karpenter Documentation](https://karpenter.sh/docs/)

**Repository:** [kubernetes-sigs/karpenter](https://github.com/kubernetes-sigs/karpenter) (1,453+ commits, Apache 2.0)  
**AWS Provider:** [aws/karpenter-provider-aws](https://github.com/aws/karpenter-provider-aws)  
**Documentation:** [karpenter.sh/docs/](https://karpenter.sh/docs/)

---

## Official Resources

| Resource | URL |
|----------|-----|
| Documentation | https://karpenter.sh/docs/ |
| Official Getting Started | https://karpenter.sh/docs/getting-started/getting-started-with-karpenter/ |
| GitHub (core) | https://github.com/kubernetes-sigs/karpenter |
| GitHub (AWS provider) | https://github.com/aws/karpenter-provider-aws |
| Karpenter Blueprints | https://github.com/aws-samples/karpenter-blueprints |
| AWS EKS Workshop | https://www.eksworkshop.com/docs/fundamentals/compute/karpenter/ |

---

## Architecture

### Core Concepts

Karpenter operates through a **layered constraint scheduling** system across three levels:

| Layer | Source | Description |
|-------|--------|-------------|
| 1 | Cloud Provider | Instance types, architectures, zones, purchase types |
| 2 | NodePools | Administrator-defined constraints |
| 3 | Pod Specifications | Workload-specific requirements |

> **Critical Rule:** Pod scheduling constraints must fall within NodePool constraints or pods will not deploy.

### Components

- **NodePool** — Defines constraints on nodes Karpenter can create and pods that can run on them
- **EC2NodeClass** — AWS-specific resource defining AMIs, subnets, security groups, and instance profiles
- **NodeClaim** — Represents the actual node resource request (Kubernetes custom resource)
- **Disruption Controller** — Automatically discovers disruptable nodes and spins up replacements
- **Termination Controller** — Gracefully drains nodes before removing the underlying NodeClaim

### Key Workflow

1. Pods become unschedulable (e.g., due to resource lacking)
2. Karpenter observes these pods via the Kubernetes API
3. It uses pod specs and NodePool constraints to select the best instance type
4. Launches a node that fits the requirements
5. Once the node is ready, the pod is scheduled
6. Karpenter can later scale down idle nodes

---

## Node Provisioning Speed

Karpenter bypasses Auto Scaling Groups (ASGs), enabling dramatically faster node provisioning:

> "Because Karpenter bypasses Auto Scaling Groups (ASGs) it can put a node on‑line in **≈45‑60 s** in AWS tests, and even replace an interrupted Spot node inside the two‑minute notice window."
> — [chkk.io](https://chkk.io/blog/karpenter-vs-cluster-autoscaler)

**vs. Cluster Autoscaler:**
> "Cluster Autoscaler scans the full cluster every 10 s by default, **simulates** scheduling for each node group, then adjusts the desired capacity of that ASG... **Minutes-level scale-up latency** because it waits a full scan cycle and relies on the cloud provider to spin up ASG nodes."
> — [chkk.io](https://chkk.io/blog/karpenter-vs-cluster-autoscaler)

| Metric | Karpenter | Cluster Autoscaler |
|--------|-----------|---------------------|
| Node provisioning | ~55 seconds | 3–4 minutes |
| Scaling trigger | Event-driven (immediate) | Time-driven (10s scan cycle) |
| Approach | Pod-centric, direct instance launch | ASG-based, node group simulation |
| Consolidation | Proactive (seeks cheaper nodes) | Reactive (only removes idle nodes) |

---

## GPU Scheduling

### Supported Accelerator Resources

| Resource | Provider |
|----------|----------|
| `nvidia.com/gpu` | NVIDIA GPUs |
| `amd.com/gpu` | AMD GPUs |
| `aws.amazon.com/neuron` | AWS Neuron devices |
| `aws.amazon.com/neuroncore` | AWS Neuron cores |
| `habana.ai/gaudi` | Habana Gaudi accelerators |

### GPU Instance Requirements

```yaml
spec:
  template:
    spec:
      containers:
      - name: training
        image: pytorch-training:latest
        resources:
          limits:
            nvidia.com/gpu: "1"
```

> **Important:** You must deploy the appropriate device plugin daemonset for GPU nodes. Without it, Karpenter won't see nodes as initialized.

### GPU NodePool Example

```yaml
apiVersion: karpenter.sh/v1
kind: NodePool
metadata:
  name: gpu-pool
spec:
  template:
    spec:
      nodeClassRef:
        group: karpenter.k8s.aws
        kind: EC2NodeClass
        name: default
      requirements:
        - key: "karpenter.k8s.aws/instance-family"
          operator: In
          values: ["g4dn", "g5", "p3", "p4d", "p5"]
        - key: "karpenter.sh/capacity-type"
          operator: In
          values: ["spot", "on-demand"]
        - key: "kubernetes.io/arch"
          operator: In
          values: ["amd64"]
      taints:
        - key: "nvidia.com/gpu"
          effect: NoSchedule
          value: "true"
```

### GPU Time-Slicing with Karpenter

Karpenter can combine with NVIDIA time-slicing to share GPUs among multiple pods:

> "By leveraging **Karpenter**, we can dynamically scale GPU nodes up or down based on demand, ensuring cost-efficiency and optimal resource utilization. The combination of **NVIDIA Device Plugin**, **time slicing**, and **Karpenter** provides a powerful approach to manage, optimize, and scale GPU resources in a Kubernetes cluster."
> — [Cisco Developer Blog](https://blogs.cisco.com/developer/optimizing-ai-workloads-with-nvidia-gpus-time-slicing-and-karpenter-part-2)

**Time-Slicing ConfigMap:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nvidia-device-plugin
  namespace: kube-system
data:
  any: |-
    version: v1
    flags:
      migStrategy: none
    sharing:
      timeSlicing:
        resources:
          - name: nvidia.com/gpu
            replicas: 3  # Slices GPU into 3 virtual instances
```

---

## Spot Instance Management

### How Spot Interruption Handling Works

Karpenter handles EC2 Spot Instance interruptions through a sophisticated mechanism:

> "Spot interruptions have a **2 minute notice** before Amazon EC2 reclaims the instance. Once Karpenter has received this warning it will begin gracefully evicting pods and launching replacement nodes."
> — [Medium/@zghanem](https://medium.com/@zghanem/how-spot-interruption-handling-works-in-karpenter-c2c6d8271f22)

### Interruption Handling Architecture

Karpenter listens for interruption notices via:
1. **Instance metadata service** — immediate notice
2. **SQS queue** — configured for broader interruption events (Rebalance Recommendations, Spot Instance Interruption Warnings)

> "This setup allows Karpenter to automatically detect termination notices, drain affected nodes, and provision replacements."
> — [Medium/@snehatuladhar098](https://medium.com/@snehatuladhar098/automating-ec2-spot-interruption-handling-with-karpenter-and-terraform-8374487a4822)

### Spot Consolidation

Karpenter supports **Spot-to-Spot consolidation** to replace existing Spot nodes with cheaper equivalents:

> "Spot-to-Spot consolidation is a Karpenter feature that replaces existing Spot nodes with cheaper, equivalent Spot instances when available. Karpenter works to actively reduce overprovisioning and thus cluster cost by identifying opportunities to consolidate nodes. For single-node Spot-to-Spot consolidations, Karpenter requires a **minimum of 15 instance types** to ensure sufficient flexibility."
> — [nOps](https://www.nops.io/blog/spot-to-spot-consolidation-in-karpenter-how-to-use-best-practices/)

### Best Practices for Spot Training Workloads

> "Yes, there will be an interruption if you're running a single pod. The solution is to run two or more pods and use a **Pod Disruption Budget** and a **Pod Anti-Affinity** rule."
> — [Reddit/r/kubernetes](https://www.reddit.com/r/kubernetes/comments/118qr6q/spot_instances_with_karpenter/)

---

## Training Job Preemption and Disruption Controls

### Disruption Methods

Karpenter disrupts nodes through automated methods executed in order:
1. **Drift** — NodeClaim values don't match owning NodePool/EC2NodeClass
2. **Consolidation** — Empty or underutilized nodes

### NodePool Disruption Budgets

```yaml
spec:
  disruption:
    consolidationPolicy: WhenEmptyOrUnderutilized
    consolidateAfter: 1m
    budgets:
      - nodes: "10%"  # Max 10% of nodes disrupted at once
```

> "For spot nodes, Karpenter has deletion consolidation enabled by default. When configured, a node may be disrupted via drift even if there are pods with blocking PDBs or the `karpenter.sh/do-not-disrupt` annotation scheduled to it. Budgets will consider nodes that are actively being deleted for any reason, and will only block Karpenter from disrupting nodes voluntarily through drift, emptiness, and consolidation."
> — [Karpenter Disruption Docs](https://karpenter.sh/docs/concepts/disruption/)

### Protecting Training Jobs

**Annotation-based Protection:**
- `karpenter.sh/do-not-evict: true` — Prevents node from being drained
- `karpenter.sh/do-not-consolidate: true` — Prevents node from being consolidated

> **Note:** In Karpenter v1.0, `karpenter.sh/do-not-evict` was deprecated and replaced by `karpenter.sh/do-not-consolidate`.

> "Announcing Karpenter 1.0: The karpenter.sh/do-not-evict annotation was declared as deprecated throughout beta and is dropped in v1. karpenter.sh/do-not-consolidate"
> — [AWS Blog](https://aws.amazon.com/blogs/containers/announcing-karpenter-1-0/)

### Kyverno Policy for Auto-Annotating Jobs

```yaml
# Kyverno policy to auto-add do-not-evict to all Job/CronJob pods
name: add-karpenter-donot-evict
policies.kyverno.io/title: Add Karpenter Do Not Evict
policies.kyverno.io/subject: Pod
spec:
  rules:
    - name: do-not-evict-jobs
      match:
        resources:
          kinds:
            - Job
      mutate:
        patchJson: '[{"op":"add","path":"/spec/template/metadata/annotations/karpenter.sh~1do-not-evict","value":"true"}]'
```

### Expiration for Node Lifetime Control

```yaml
spec:
  template:
    spec:
      expireAfter: 720h  # 30 days max node lifetime
      terminationGracePeriod: 48h
```

> "Nodes begin draining immediately when lifetime exceeded... **Warning:** Nodes with `karpenter.sh/do-not-disrupt` annotation or blocking PDBs can still be terminated if `terminationGracePeriod` is configured."

---

## Use Cases for ML Training

### Karpenter in ML Training Workloads

> "When your ML training job pods (e.g., TensorFlow, PyTorch) request GPUs or large CPUs, Karpenter detects pending pods and **launches new nodes optimized for those resource requests** (like GPU instances) automatically."
> — [Medium/@priyasrivastava18official](https://medium.com/@priyasrivastava18official/karpenter-in-ml-training-workloads-5e4a43cb6f6d)

### CoStar Case Study

> "For CoStar, Karpenter consolidated the Amazon EC2 Spot Instance management into a single, unified auto-scaling approach."
> — [AWS Containers Blog](https://aws.amazon.com/blogs/containers/how-costar-uses-karpenter-to-optimize-their-amazon-eks-resources/)

### Drug Discovery AI Training

> "In this post, we focus on how we used Karpenter on Amazon Elastic Kubernetes Service (Amazon EKS) to scale AI training and inference."
> — [AWS Machine Learning Blog](https://aws.amazon.com/blogs/machine-learning/scale-ai-training-and-inference-for-drug-discovery-through-amazon-eks-and-karpenter/)

### Loka — Building Scalable Infrastructure for Training Specialized LLMs

> "The solution centers on three key technologies working together: **Kubernetes** for orchestration, **Karpenter** for automatic node provisioning, and **Kubeflow** for distributed training jobs. Dynamic GPU nodes, provisioned on-demand by Karpenter, provide elastic training capacity and exist only during active jobs. **Karpenter's autoscaling means GPU nodes only exist during active training, so you're not paying for idle capacity.**"
> — [Loka Blog](https://www.loka.com/blog/building-scalable-infrastructure-for-training-specialized-llms)

### Comparison: Cluster Autoscaler vs Karpenter for ML

| Aspect | Cluster Autoscaler | Karpenter |
|--------|-------------------|-----------|
| GPU Node Setup | Must pre-create GPU node groups | No pre-created node groups needed |
| Scaling | Only within predefined node groups | Dynamically launches based on demand |
| Instance Type Flexibility | Limited to existing node groups | Picks best-fit nodes dynamically |
| Cost Risk | Over-provisioning GPU nodes | No idle nodes kept longer than necessary |
| Speed | Slower (ASG lifecycle) | Faster (~55s vs 3-4min) |
| Pod Wait Time | Higher | Reduced |

---

## Benchmarks and Performance

### Scaling Speed Benchmark

| Metric | Karpenter | Cluster Autoscaler |
|--------|-----------|-------------------|
| Node provisioning | ~55 seconds | 3–4 minutes |
| Retry behavior | Milliseconds | Full scan cycle |
| Consolidation | Proactive | Reactive only |
| Spot replacement | Within 2-min notice window | Not handled natively |

> "Performance difference is real: Node provisioning: Karpenter ~55 seconds vs CA 3-4 minutes consistently. Karpenter retries in milliseconds."
> — [Reddit/RunWithTasrie](https://www.reddit.com/r/RunWithTasrie/comments/1roxbyt/cluster_autoscaler_vs_karpenter_we_tested_both_on/)

### GPU Workload Considerations

> "**Long-running GPU-based ML workloads:** Prefer **Cluster Autoscaler** to leverage pre-defined GPU node groups, avoiding unnecessary pod rescheduling and resource churn."
> — [chkk.io](https://chkk.io/blog/karpenter-vs-cluster-autoscaler)

> "**Cost-optimized Batch & CI Workloads:** Choose **Karpenter** to maximize cost savings via aggressive consolidation with Spot and OnDemand instances."

### Hybrid Approach (Common EKS Blueprint)

> "**Hybrid Approach (Common EKS Blueprint):** Combine **CA-managed node groups** for baseline capacity with **Karpenter** for burst handling, providing the best of both worlds."
> — [chkk.io](https://chkk.io/blog/karpenter-vs-cluster-autoscaler)

---

## NodePool Configuration Reference

```yaml
apiVersion: karpenter.sh/v1
kind: NodePool
metadata:
  name: training-pool
spec:
  template:
    metadata:
      labels:
        billing-team: ml-team
    spec:
      nodeClassRef:
        group: karpenter.k8s.aws
        kind: EC2NodeClass
        name: default
      taints:
        - key: example.com/special-taint
          effect: NoSchedule
      expireAfter: 720h  # 30 days
      terminationGracePeriod: 48h
      requirements:
        - key: "karpenter.k8s.aws/instance-category"
          operator: In
          values: ["c", "m", "r", "g", "p"]
          minValues: 2
        - key: "karpenter.sh/capacity-type"
          operator: In
          values: ["spot", "on-demand"]
        - key: "kubernetes.io/arch"
          operator: In
          values: ["amd64"]
  disruption:
    consolidationPolicy: WhenEmptyOrUnderutilized
    consolidateAfter: 1m
    budgets:
      - nodes: "10%"
      - nodes: "1"  # At least 1 node always available
  limits:
    cpu: "1000"
    memory: 1000Gi
```

### Well-Known Labels

| Label | Example | Description |
|-------|---------|-------------|
| `karpenter.k8s.aws/instance-family` | g4dn | Instance family |
| `karpenter.k8s.aws/instance-category` | c, m, r, g, p | Instance category |
| `karpenter.k8s.aws/instance-gpu-count` | 1, 4, 8 | GPU count |
| `karpenter.k8s.aws/instance-memory` | 32768 | Memory in MiB |
| `karpenter.k8s.aws/instance-pods` | 110 | Supported pod count |
| `karpenter.sh/capacity-type` | spot, on-demand | Capacity type |
| `karpenter.sh/nodepool` | default | NodePool name |

---

## Installation

### Helm Installation

```bash
export KARPENTER_NAMESPACE="karpenter"
export CLUSTER_NAME="my-cluster"
export AWS_ACCOUNT_ID="123456789012"
export AWS_DEFAULT_REGION="us-west-2"
export KARPENTER_VERSION="v1.0.0"

# Install Karpenter via Helm
helm upgrade --install karpenter oci://public.ecr.aws/karpenter/karpenter \
  --version "${KARPENTER_VERSION}" \
  --namespace "${KARPENTER_NAMESPACE}" \
  --create-namespace \
  --set serviceAccount.annotations."eks\.amazonaws\.com/role-arn"="arn:aws:iam::${AWS_ACCOUNT_ID}:role/${CLUSTER_NAME}-karpenter" \
  --set controller.clusterName="${CLUSTER_NAME}" \
  --set controller.region="${AWS_DEFAULT_REGION}"
```

### Terraform Setup Reference

> "The first thing we need to do is create an IAM role for the Karpenter controller and allow it to be assumed by the ServiceAccount."
> — [mikesir87's blog](https://blog.mikesir87.io/2021/12/deploying-karpenter-with-tf/)

---

## Key Quotes for Wiki

> "Karpenter improves the efficiency and cost of running workloads on Kubernetes clusters by: provision[ing] the right nodes right sized for your workloads, replac[ing] multiple node groups with a heterogenous node pool, and improv[ing] cluster utilization."

> "Unlike traditional autoscalers that scale slowly or require pre-provisioned node groups, Karpenter quickly reacts in near real-time, launching nodes as soon as new pods need them."

> "Karpenter's autoscaling means GPU nodes only exist during active training, so you're not paying for idle capacity."

---

## Related Topics

- [Elastic Training on Kubernetes](https://github.com/skai-x/elastic-training) — End-to-end docs on elastic training with Kubernetes
- [TorchElastic on Kubernetes](https://kengz.gitbook.io/blog/ml/distributed-training-with-torchelastic-on-kubernetes) — Distributed training with PyTorch Elastic
- [Distributed Training with Amazon EKS](https://aws.amazon.com/blogs/machine-learning/distributed-training-with-amazon-eks-and-torch-distributed-elastic/) — AWS official blog on Torch Distributed Elastic
- [Karpenter Blueprints](https://github.com/aws-samples/karpenter-blueprints) — Common workload scenarios and configurations

---

## Tags

`#Karpenter` `#Kubernetes` `#Autoscaler` `#GPU` `#Spot-Instances` `#ML-Training` `#EKS` `#NodePool` `#Disruption` `#Cost-Optimization`

---

*Last updated: 2026-04-30*
