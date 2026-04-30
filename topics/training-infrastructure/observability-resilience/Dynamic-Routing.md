# Dynamic Routing for ML Training Workloads on Kubernetes

## Table of Contents

1. [Overview](#overview)
2. [Kubernetes Service Routing for Training Jobs](#kubernetes-service-routing-for-training-jobs)
3. [Headless Services for Distributed Training Discovery](#headless-services-for-distributed-training-discovery)
4. [Istio/Envoy Service Mesh for Training Workloads](#istioenvoy-service-mesh-for-training-workloads)
5. [DNS-Based Routing for Parameter Server/Worker Communication](#dns-based-routing-for-parameter-serverworker-communication)
6. [Proxy-Based Routing in Training Jobs](#proxy-based-routing-in-training-jobs)
7. [Multi-Tenant Routing](#multi-tenant-routing)
8. [JobSet and Training Topology Routing](#jobset-and-training-topology-routing)
9. [Key Technologies and Repositories](#key-technologies-and-repositories)
10. [References](#references)

---

## Overview

Dynamic routing in Kubernetes is essential for distributed ML training workloads that require flexible service discovery, traffic management, and fault-tolerant communication between parameter servers, workers, and other training components. This document covers the mechanisms, technologies, and patterns for implementing dynamic routing in ML training infrastructure on Kubernetes.

> "Istio's traffic routing rules provide fine-grained control over service-to-service communication, supporting use cases like A/B testing, canary rollouts, and staged rollouts with percentage-based traffic splits."

**Source**: [Istio Traffic Management Documentation](https://istio.io/latest/docs/concepts/traffic-management/)

---

## Kubernetes Service Routing for Training Jobs

### Service Types for Training Workloads

Kubernetes provides several service types that serve different routing purposes for ML training:

| Service Type | Description | Use Case for Training |
|--------------|-------------|----------------------|
| `ClusterIP` | Default internal IP, accessible only within cluster | Worker-to-worker communication within a training job |
| `NodePort` | Exposes service on each Node's IP at static port (30000-32767) | Development/debugging access to training jobs |
| `LoadBalancer` | Provisions external load balancer (cloud provider dependent) | Serving inference endpoints from training cluster |
| `Headless` (`clusterIP: None`) | No cluster IP; DNS returns pod IPs directly | Parameter server and worker discovery in distributed training |

**Source**: [Kubernetes Service Documentation](https://kubernetes.io/docs/concepts/services-networking/service/)

### Training Job Routing with Kubeflow Operators

Kubeflow Training Operators create Kubernetes pods with environment variables for distributed training communication:

> "The Training Operator creates Kubernetes pods with the appropriate environment variables for the `torchrun` CLI to start the distributed PyTorch training job."

**Source**: [Kubeflow Distributed Training Documentation](https://www.kubeflow.org/docs/components/trainer/legacy-v1/reference/distributed-training/)

**Key Environment Variables for Training Routing**:

| Variable | Description |
|----------|-------------|
| `WORLD_SIZE` | Total number of training processes |
| `RANK` | Index of current process |
| `LOCAL_RANK` | Local rank on the node |
| `MASTER_ADDR` | Address of rank 0 |
| `MASTER_PORT` | Port for communication |

**Source**: [Kubeflow PyTorchJob Getting Started](https://www.kubeflow.org/docs/components/trainer/legacy-v1/getting-started/)

### Service Discovery in Training

Kubernetes DNS automatically creates DNS records for every service:

> "When a pod is created, Kubernetes injects DNS configuration into the pod's `/etc/resolv.conf` file. This configuration points to CoreDNS as the nameserver and sets up search domains for short-name resolution."

**Source**: [Kubernetes DNS-Based Service Discovery](https://oneuptime.com/blog/post/2026-02-20-kubernetes-dns-service-discovery/view)

---

## Headless Services for Distributed Training Discovery

### What Are Headless Services?

A Headless Service is created by setting `clusterIP: None`. Unlike standard services that return a single ClusterIP, headless services return individual pod IPs through DNS:

> "CoreDNS detects that the Service is headless and returns multiple A records, one for each pod that matches the Service selector, instead of returning a single ClusterIP."

**Source**: [Kubernetes Headless Service Deep Dive](https://blog.devops.dev/a-deep-dive-into-kubernetes-headless-service-98a45cca68e8)

### Headless Service Use Cases for Training

| Use Case | Description |
|----------|-------------|
| **Parameter Server Discovery** | Workers discover parameter servers via DNS without kube-proxy load balancing |
| **Worker-to-Worker Communication** | Direct pod-to-pod communication for gradient synchronization |
| **Custom Load Balancing** | Applications implement client-side load balancing decisions |
| **Stateful Training Components** | Stable network identities for training components |

> "Headless Services give applications the ability to implement client-side load balancing, instead of relying on Kubernetes to distribute traffic automatically."

**Source**: [Kubernetes Headless Service Deep Dive](https://blog.devops.dev/a-deep-dive-into-kubernetes-headless-service-98a45cca68e8)

### DNS Resolution for Training Jobs

When a client queries the DNS of a Headless Service, it receives the IP addresses of all pods matching the service selector:

> "With a Headless Service, applications can use DNS directly to discover backend pods instead of relying on kube-proxy or a ClusterIP."

**Source**: [Pass4Sure Headless Service Guide](https://www.pass4sure.com/blog/understanding-kubernetes-headless-service-a-detailed-overview-with-practical-examples-2/)

### Example: Headless Service for Distributed Training

```yaml
apiVersion: v1
kind: Service
metadata:
  name: training-workers
spec:
  clusterIP: None  # This makes it headless
  selector:
    app: ml-worker
    training-job: my-distributed-job
  ports:
    - name: grpc
      port: 5000
      targetPort: 5000
```

This produces DNS records that allow workers to discover all peer pods directly.

---

## Istio/Envoy Service Mesh for Training Workloads

### Istio Traffic Management Overview

Istio provides advanced traffic management capabilities through Envoy proxies deployed as sidecars:

> "Istio relies on **Envoy proxies** deployed alongside services to handle all data plane traffic. This allows directing and controlling mesh traffic without modifying services."

**Source**: [Istio Traffic Management](https://istio.io/latest/docs/concepts/traffic-management/)

### Key Istio Resources for Training Routing

| Resource | Purpose |
|----------|---------|
| **Virtual Services** | Route requests to destinations within mesh |
| **Destination Rules** | Configure policies for traffic at destination |
| **Gateways** | Manage inbound/outbound mesh traffic |
| **Service Entries** | Add external services to service registry |
| **Sidecars** | Fine-tune Envoy proxy behavior |

**Source**: [Istio Traffic Management](https://istio.io/latest/docs/concepts/traffic-management/)

### Virtual Service Routing for Training

Virtual services decouple client request destinations from actual workloads:

> "Virtual services, along with destination rules, are the key building blocks of Istio's traffic routing functionality. Each virtual service consists of a set of routing rules that are evaluated in order, letting Istio match each given request to the virtual service to a specific real destination within the mesh."

**Source**: [Istio Traffic Management](https://istio.io/latest/docs/concepts/traffic-management/)

### Load Balancing Options for Training

Istio Destination Rules support multiple load balancing models:

| Method | Description | Training Use Case |
|--------|-------------|-------------------|
| `LEAST_REQUEST` | Default - routes to host with fewest active requests | Worker-to-parameter-server communication |
| `RANDOM` | Forward requests at random | Gradient exchange between workers |
| `ROUND_ROBIN` | Forward to each instance in sequence | Parameter server replication |
| `CONSISTENT_HASH` | Soft session affinity based on HTTP headers/cookies | Stateful training sessions |

**Source**: [Istio Traffic Management](https://istio.io/latest/docs/concepts/traffic-management/)

### Canary Deployments for Training Jobs

Istio enables canary deployments for training workloads:

> "Istio's service mesh provides the control necessary to manage traffic distribution with complete independence from deployment scaling."

**Source**: [Istio Canary Deployments](https://istio.io/latest/blog/2017/0.1-canary/)

### Envoy Sidecar for Training

Envoy proxies intercept traffic through iptables rules:

> "Envoy is an open-source edge and service proxy designed for cloud-native applications. Developed by Lyft and now part of the Cloud Native Computing Foundation (CNCF), Envoy acts as a **sidecar proxy**—a companion container deployed alongside application containers in Kubernetes pods."

**Source**: [Envoy Sidecar Proxy Guide](https://dev.to/naveens16/envoy-sidecar-proxy-mastering-traffic-control-observability-and-autoscaling-in-kubernetes-kcg)

---

## DNS-Based Routing for Parameter Server/Worker Communication

### Parameter Server Architecture

The Parameter Server architecture is a client-server model for distributed training:

> "The Parameter Server architecture is a client-server model that breaks down the training process into two main components: workers and parameter servers."

**Source**: [Parameter Server vs All-Reduce](https://eureka.patsnap.com/article/parameter-server-vs-all-reduce-distributed-training-architecture-tradeoffs)

### DNS-Based Discovery Mechanism

```
Pod [Application Pod] --> |DNS Query: my-service.default.svc.cluster.local| 
Resolver [Pod's /etc/resolv.conf] --> |Forward to| 
CoreDNS [CoreDNS Pod:53] --> |Lookup in| 
Cache [CoreDNS Cache] --> |Watch| 
API [Kubernetes API Server] --> |Service + Endpoint data| 
CoreDNS --> |A Record: pod IPs| 
Pod --> |TCP/HTTP to pod IPs directly| 
Service [Target Service]
```

**Source**: [Kubernetes DNS Service Discovery](https://oneuptime.com/blog/post/2026-02-20-kubernetes-dns-service-discovery/view)

### Headless Services for Parameter Server Discovery

For parameter server training, headless services enable workers to discover all parameter servers directly:

> "A headless Service provides DNS-based discovery, enabling each database Pod to find and connect to its siblings using predictable hostnames and IPs. In such systems, internal replication and synchronization rely on known peer addresses."

**Source**: [Pass4Sure Headless Service Guide](https://www.pass4sure.com/blog/understanding-kubernetes-headless-service-a-detailed-overview-with-practical-examples-2/)

### Worker-to-Worker DNS Routing

DNS-based service discovery allows workers to find peer workers for gradient synchronization:

> "When a client queries the DNS of a Headless Service, it receives the IP addresses of all pods that match the service selector. The client is responsible for deciding which pod to contact."

**Source**: [Kubernetes Headless Service Deep Dive](https://blog.devops.dev/a-deep-dive-into-kubernetes-headless-service-98a45cca68e8)

---

## Proxy-Based Routing in Training Jobs

### Envoy Proxy in Training Workloads

Envoy proxy can be deployed as a sidecar to manage ML training traffic:

> "When the http-client makes outbound calls (to the 'upstream' service), all of the calls go through the Envoy Proxy sidecar."

**Source**: [Red Hat Envoy Microservices Patterns](https://www.redhat.com/en/blog/microservices-patterns-envoy-part-i)

### Service Mesh Benefits for Training

| Feature | Benefit for ML Training |
|---------|------------------------|
| **Circuit Breaking** | Prevents cascading failures during training |
| **Retries/Timeouts** | Handles transient network failures |
| **Traffic Shaping** | Controls gradient exchange rates |
| **Observability** | Metrics on training communication patterns |
| **mTLS** | Secure communication between training components |

**Source**: [Istio Traffic Management](https://istio.io/latest/docs/concepts/traffic-management/)

### Ray on Kubernetes Routing

Ray clusters use Kubernetes services for internal communication:

> "Ray clusters consist of a head node pod and worker node pods. The KubeRay operator manages Ray clusters on Kubernetes."

**Source**: [Ray on Kubernetes Documentation](https://docs.ray.io/en/latest/cluster/kubernetes/index.html)

### KubeRay Service Configuration

| CRD | Use Case |
|-----|----------|
| **RayCluster** | Persistent Ray cluster for long-running training workloads |
| **RayJob** | Batch job submission with single-run or serverless modes |
| **RayService** | Long-running Ray Serve applications with zero-downtime upgrades |

**Source**: [Ray on Kubernetes Documentation](https://docs.ray.io/en/latest/cluster/kubernetes/index.html)

---

## Multi-Tenant Routing

### Multi-Tenancy Models in Kubernetes

Kubernetes supports multiple tenancy models for shared clusters:

| Model | Description |
|-------|-------------|
| **Namespaces-as-a-Service** | Each tenant gets dedicated namespaces |
| **Clusters-as-a-Service** | Each tenant gets dedicated clusters |
| **Control Planes-as-a-Service** | Virtual control planes per tenant with shared worker nodes |

**Source**: [Kubernetes Multi-Tenancy Working Group](https://kubernetes.io/blog/2021/04/15/three-tenancy-models-for-kubernetes/)

### Namespace Isolation for Training

> "A common practice is to **isolate every workload in its own namespace**, even if operated by the same tenant."

**Source**: [Kubernetes Multi-Tenancy Documentation](https://kubernetes.io/docs/concepts/security/multi-tenancy/)

### Network Policies for Tenant Isolation

> "Default behavior: All pods can communicate; all traffic is unencrypted."

**Source**: [Kubernetes Multi-Tenancy Documentation](https://kubernetes.io/docs/concepts/security/multi-tenancy/)

Network policies provide tenant isolation:

```yaml
# Default deny-all policy
# Then allow DNS queries and namespace-specific communication
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
spec:
  podSelector: {}
```

### Service Mesh for Multi-Tenant Training

Service meshes provide advanced multi-tenant routing:

> "Leveraging a service mesh, such as Istio or Linkerd, can provide an additional layer of isolation and control over inter-service communication."

**Source**: [Civo Kubernetes Multi-Tenancy](https://www.civo.com/blog/kubernetes-multi-tenancy-introduction)

### KubeSlice for Multi-Tenant Routing

KubeSlice provides network-level tenant isolation:

> "Multi-tenancy in Kubernetes is a way to deploy multiple workloads in a shared cluster with isolated network traffic, resources, user access, and last but not least control plane access."

**Source**: [Avesha KubeSlice Documentation](https://avesha.io/resources/blog/how-kube-slice-implements-multi-tenancy)

---

## JobSet and Training Topology Routing

### JobSet Overview

JobSet is a Kubernetes-native API for managing distributed ML and HPC jobs:

> "JobSet is a Kubernetes-native API for managing a group of k8s Jobs as a unit. It aims to offer a unified API for deploying HPC (eg, MPI) and AI/ML training workloads."

**Source**: [JobSet GitHub Repository](https://github.com/kubernetes-sigs/jobset)

### Key JobSet Features for Routing

| Feature | Description |
|---------|-------------|
| **ReplicatedJobs** | Declarative replication of Jobs across accelerator islands |
| **Startup Sequencing** | Supports "driver-first" (e.g., Ray) or "worker-first" (e.g., MPI) patterns |
| **Automatic Inter-Pod Communication** | Built-in headless service creation with lifecycle management |
| **Exclusive Topology Placement** | Enforces replicated jobs scheduled to separate network/compute domains |

**Source**: [Scaling ML Training with JobSet](https://blog.abhimanyu-saharan.com/posts/scaling-ml-training-on-kubernetes-with-jobset)

### JobSet Automatic Headless Service Creation

> "Automatic Inter-Pod Communication: Built-in headless service creation with lifecycle management"

**Source**: [JobSet GitHub Repository](https://github.com/kubernetes-sigs/jobset)

### Kueue Integration for Multi-Tenant Job Routing

JobSet integrates with Kueue for workload queuing and prioritization:

> "JobSet integrates with Kueue for job queuing, oversubscription, and multi-tenant workload prioritization—making it suitable for shared HPC or ML clusters."

**Source**: [Scaling ML Training with JobSet](https://blog.abhimanyu-saharan.com/posts/scaling-ml-training-on-kubernetes-with-jobset)

---

## Key Technologies and Repositories

### Kubernetes Service Mesh Technologies

| Technology | Purpose | Repository |
|------------|---------|------------|
| **Istio** | Service mesh with traffic management | https://github.com/istio/istio |
| **Envoy** | L7 proxy and sidecar | https://github.com/envoyproxy/envoy |
| **Linkerd** | Lightweight service mesh | https://github.com/linkerd/linkerd2 |

### Kubernetes Training Operators

| Operator | Purpose | Repository |
|----------|---------|------------|
| **Kubeflow Training Operator** | TFJob, PyTorchJob, MPIJob | https://github.com/kubeflow/training-operator |
| **JobSet** | Distributed job management | https://github.com/kubernetes-sigs/jobset |
| **KubeRay** | Ray cluster on Kubernetes | https://github.com/ray-project/kuberay |
| **Kueue** | Job queueing and batch scheduling | https://github.com/kubernetes-sigs/kueue |

### Official Documentation Links

| Documentation | URL |
|---------------|-----|
| Kubernetes Services | https://kubernetes.io/docs/concepts/services-networking/service/ |
| Istio Traffic Management | https://istio.io/latest/docs/concepts/traffic-management/ |
| Kubernetes Multi-Tenancy | https://kubernetes.io/docs/concepts/security/multi-tenancy/ |
| Kubeflow Training | https://www.kubeflow.org/docs/components/trainer/ |
| Ray on Kubernetes | https://docs.ray.io/en/latest/cluster/kubernetes/index.html |

---

## References

1. Kubernetes Service Documentation - https://kubernetes.io/docs/concepts/services-networking/service/
2. Istio Traffic Management - https://istio.io/latest/docs/concepts/traffic-management/
3. Kubernetes Multi-Tenancy - https://kubernetes.io/docs/concepts/security/multi-tenancy/
4. Kubeflow Training Operator - https://github.com/kubeflow/training-operator
5. Kubernetes Headless Service Deep Dive - https://blog.devops.dev/a-deep-dive-into-kubernetes-headless-service-98a45cca68e8
6. JobSet GitHub Repository - https://github.com/kubernetes-sigs/jobset
7. Scaling ML Training with JobSet - https://blog.abhimanyu-saharan.com/posts/scaling-ml-training-on-kubernetes-with-jobset
8. Ray on Kubernetes - https://docs.ray.io/en/latest/cluster/kubernetes/index.html
9. Kubeflow Distributed Training - https://www.kubeflow.org/docs/components/trainer/legacy-v1/reference/distributed-training/
10. Envoy Sidecar Proxy Guide - https://dev.to/naveens16/envoy-sidecar-proxy-mastering-traffic-control-observability-and-autoscaling-in-kubernetes-kcg
11. Kubernetes DNS Service Discovery - https://oneuptime.com/blog/post/2026-02-20-kubernetes-dns-service-discovery/view
12. Istio Canary Deployments - https://istio.io/latest/blog/2017/0.1-canary/
13. Red Hat Envoy Microservices Patterns - https://www.redhat.com/en/blog/microservices-patterns-envoy-part-i
14. Civo Kubernetes Multi-Tenancy - https://www.civo.com/blog/kubernetes-multi-tenancy-introduction
15. KubeSlice Multi-Tenancy - https://avesha.io/resources/blog/how-kube-slice-implements-multi-tenancy

---

*Last updated: 2026-04-30*