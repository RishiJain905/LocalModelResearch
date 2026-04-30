# Kubeflow ML Platform on Kubernetes

## Table of Contents

1. [Overview](#overview)
2. [Official Documentation & Repositories](#official-documentation--repositories)
3. [Training Operators (TFJob, PyTorchJob, MPIJob)](#training-operators-tfjob-pytorchjob-mpijob)
4. [Kubeflow Trainer v2 (TrainJob API)](#kubeflow-trainer-v2-trainjob-api)
5. [Distributed Training on Kubernetes](#distributed-training-on-kubernetes)
6. [Elastic Training Patterns](#elastic-training-patterns)
7. [KServe Inference Serving](#kserve-inference-serving)
8. [Katib Hyperparameter Tuning](#katib-hyperparameter-tuning)
9. [Kubeflow Pipelines](#kubeflow-pipelines)
10. [Use Cases & Production Deployments](#use-cases--production-deployments)

---

## Overview

> "Kubeflow is the foundation of tools for AI Platforms on Kubernetes. AI platform teams can build on top of Kubeflow by using each project independently or deploying the entire AI reference platform to meet their specific needs."

**Source**: [Kubeflow GitHub](https://github.com/kubeflow/kubeflow)

Kubeflow is an open-source toolkit for making Machine Learning (ML) on Kubernetes easy, portable, and scalable. It provides a comprehensive ecosystem covering the entire AI lifecycle:

| Stage | Kubeflow Project |
|-------|------------------|
| Data Preparation | Kubeflow Spark Operator |
| Model Development | Kubeflow Notebooks |
| Model Training | Kubeflow Trainer (TrainJob, PyTorchJob, TFJob, MPIJob) |
| Model Optimization | Kubeflow Katib (AutoML, HPO) |
| Model Serving | KServe |
| Pipeline Orchestration | Kubeflow Pipelines |
| ML Metadata | Kubeflow Model Registry |

**Source**: [Kubeflow Architecture Documentation](https://www.kubeflow.org/docs/started/architecture/)

---

## Official Documentation & Repositories

### Primary Resources

| Resource | URL |
|----------|-----|
| Official Documentation | https://www.kubeflow.org/docs/ |
| Main GitHub Repository | https://github.com/kubeflow/kubeflow |
| Kubeflow Pipelines | https://github.com/kubeflow/pipelines |
| Kubeflow Training Operator | https://github.com/kubeflow/training-operator |
| Kubeflow Trainer | https://github.com/kubeflow/trainer |
| Katib (Hyperparameter Tuning) | https://github.com/kubeflow/katib |
| MPI Operator | https://github.com/kubeflow/mpi-operator |
| KServe | https://github.com/kserve/kserve |

### Key Version Information

> "Kubeflow 1.9 Release, Delivered: July 2024"

**Source**: [Kubeflow Roadmap](https://github.com/kubeflow/kubeflow/blob/master/ROADMAP.md)

### Documentation Sections

- **Getting Started**: https://www.kubeflow.org/docs/
- **Kubeflow Trainer**: https://www.kubeflow.org/docs/components/trainer/
- **Kubeflow Pipelines**: https://www.kubeflow.org/docs/components/pipelines/overview/
- **KServe**: https://www.kubeflow.org/docs/components/kserve/
- **Katib**: https://www.kubeflow.org/docs/components/katib/overview/
- **Notebooks**: https://www.kubeflow.org/docs/components/notebooks/overview/

---

## Training Operators (TFJob, PyTorchJob, MPIJob)

### Overview

Kubeflow Training Operator provides Kubernetes Custom Resource Definitions (CRDs) for distributed ML training:

> "Kubeflow provides several operators for distributed training including the TF operator, PyTorch operator, and MPI operator. The TF and PyTorch operators run distributed training jobs using the corresponding frameworks while the MPI operator allows for framework-agnostic distributed training."

**Source**: [Kubeflow Distributed Training and HPO Slides](https://www.slideshare.net/slideshow/kubeflow-distributed-training-and-hpo/236777175)

### TFJob (TensorFlow Training)

> "The `tf-operator` is the Kubeflow implementation of TFJobs."

**Source**: [Kubeflow Slideshare](https://www.slideshare.net/slideshow/kubeflow-distributed-training-and-hpo/236777175)

TFJob is a Kubernetes custom resource to run TensorFlow training jobs. The Training Operator creates Kubernetes pods with appropriate environment variables for `TF_CONFIG` to start distributed TensorFlow training jobs.

**Key Features**:
- Parameter server (PS) distributed training
- Support for TensorFlow distributed strategies
- Automatic `TF_CONFIG` environment variable setup

**API Version**: `kubeflow.org/v1`

**Source**: [PyTorch Training Documentation](https://www.kubeflow.org/docs/components/trainer/legacy-v1/user-guides/pytorch/)

### PyTorchJob (PyTorch Training)

> "`PyTorchJob` is a Kubernetes custom resource to run PyTorch training jobs on Kubernetes."

**Source**: [Kubeflow PyTorch Training Guide](https://www.kubeflow.org/docs/components/trainer/legacy-v1/user-guides/pytorch/)

**Key Features**:
- PyTorch Distributed Data Parallel (DDP) support
- Automatic `WORLD_SIZE` and `RANK` environment variables
- Ring all-reduce algorithm support
- Integration with `torchrun` CLI

**Example PyTorchJob**:
```yaml
apiVersion: kubeflow.org/v1
kind: PyTorchJob
metadata:
  name: pytorch-tcp-dist-mnist
spec:
  pytorchReplicaSpecs:
    Master:
      replicas: 1
      restartPolicy: OnFailure
      template:
        spec:
          containers:
          - name: pytorch
            image: gcr.io/kubeflow-ci/pytorch-dist-mnist_test:1.0
            ports:
            - containerPort: 23456
              name: pytorchjob-port
    Worker:
      replicas: 3
      restartPolicy: OnFailure
      template:
        spec:
          containers:
          - name: pytorch
            image: gcr.io/kubeflow-ci/pytorch-dist-mnist_test:1.0
```

**Source**: [Kubeflow PyTorchJob Example](https://www.kubeflow.org/docs/components/trainer/legacy-v1/user-guides/pytorch/)

### MPIJob (MPI-based Distributed Training)

> "The MPI Operator allows for running allreduce-style distributed training on Kubernetes. Unlike other operators, such as the TF Operator and the Pytorch Operator, the MPI Operator allows for framework-agnostic distributed training."

**Source**: [Kubeflow MPI Training Guide](https://www.kubeflow.org/docs/components/trainer/legacy-v1/user-guides/mpi/)

**Key Features**:
- Framework-agnostic (works with TensorFlow, PyTorch, etc.)
- Horovod integration for data-parallel training
- Gang-scheduling via Volcano or scheduler-plugins
- Supports spot/preemptible instances for cost optimization

**MPIJob v2beta1 Example**:
```yaml
apiVersion: kubeflow.org/v2beta1
kind: MPIJob
metadata:
  name: tensorflow-benchmarks
spec:
  slotsPerWorker: 1
  runPolicy:
    cleanPodPolicy: Running
    schedulingPolicy:
      minAvailable: 10
  mpiReplicaSpecs:
    Launcher:
      replicas: 1
      template:
        spec:
          containers:
          - name: mpi-launcher
            image: mpioperator/tensorflow-benchmarks:latest
    Worker:
      replicas: 2
      template:
        spec:
          containers:
          - name: mpi-worker
            image: mpioperator/tensorflow-benchmarks:latest
            resources:
              limits:
                nvidia.com/gpu: 2
```

---

## Kubeflow Trainer v2 (TrainJob API)

### Overview

> "Kubeflow Trainer v2 (KF Trainer) was created to hide this complexity, by abstracting Kubernetes from AI Practitioners and providing the easiest, most scalable way to run distributed PyTorch jobs."

**Source**: [Kubeflow Trainer v2 Blog](https://blog.kubeflow.org/trainer/intro/)

**Evolution Timeline**:
- **2017**: Kubeflow introduced TFJob to orchestrate TensorFlow training on Kubernetes
- **2021**: Multiple operators consolidated into unified Training Operator v1
- **2025**: Kubeflow Trainer v2 released with unified TrainJob API

### TrainJob API (v2alpha1)

> "Kubeflow Trainer v2 introduces new APIs that replace the older, framework-specific CRDs such as `PyTorchJob`, `TFJob`, and `MPIJob`."

**Source**: [Migrating to Kubeflow Trainer v2](https://www.kubeflow.org/docs/components/trainer/operator-guides/migration/)

**Key Design Principles**:
- Single `TrainJob` API for all ML frameworks
- Separation of concerns: Platform Administrators define `TrainingRuntimes`, AI Practitioners use `TrainJob`
- Kubernetes-native improvements (JobSet, Kueue, Indexed Jobs, PodFailurePolicy)

### Core CRDs

| CRD | Scope | Purpose |
|-----|-------|---------|
| `TrainingRuntime` | Namespace | Infrastructure details (image, failure policy, gang-scheduling) |
| `ClusterTrainingRuntime` | Cluster | Same as TrainingRuntime but cluster-wide |
| `TrainJob` | Namespace | Training job configuration (code, dataset, runtime reference) |

**Source**: [Kubeflow Trainer v2 Blog](https://blog.kubeflow.org/trainer/intro/)

### Python SDK

> "The SDK handles all Kubernetes API interactions, eliminating direct Kubernetes API interaction for AI Practitioners."

**Source**: [Kubeflow Trainer v2 Blog](https://blog.kubeflow.org/trainer/intro/)

```python
from kubeflow.trainer import TrainerClient

client = TrainerClient()

def my_train_func():
    import os
    import torch
    import torch.distributed as dist
    from torch.utils.data import DataLoader, DistributedSampler
    
    backend = "nccl" if torch.cuda.is_available() else "gloo"
    local_rank = int(os.getenv("LOCAL_RANK", 0))
    dist.init_process_group(backend=backend)
    
    model = YourModel()
    dataset = YourDataset()
    train_loader = DataLoader(dataset, sampler=DistributedSampler(dataset))
    # Training loop...
    dist.barrier()
    if dist.get_rank() == 0:
        print("Training is finished")
    dist.destroy_process_group()

job_name = client.train(
    runtime=client.get_runtime("torch-distributed"),
    trainer=CustomTrainer(
        func=my_train_func,
        num_nodes=5,
        resources_per_node={"gpu": 2},
    ),
)
```

### Built-in Trainers for LLM Fine-tuning

| Trainer Type | Description |
|--------------|-------------|
| `BuiltinTrainer` | Pre-configured runtimes for common use cases |
| `TorchTune` | PyTorch native LLM fine-tuning integration |

**Source**: [Kubeflow Trainer v2 Blog](https://blog.kubeflow.org/trainer/intro/)

### Latest Releases

> "Kubeflow Trainer v2.2: JAX & XGBoost Runtimes, Flux for HPC Support, and TrainJob progress and metrics observability."

**Source**: [Kubeflow Trainer v2.2 Release Blog](https://blog.kubeflow.org/kubeflow-trainer-v2.2-release/)

---

## Distributed Training on Kubernetes

### PyTorch Distributed Training

> "The Training Operator creates Kubernetes pods with the appropriate environment variables for the `torchrun` CLI to start the distributed PyTorch training job."

**Source**: [Distributed Training Documentation](https://www.kubeflow.org/docs/components/trainer/legacy-v1/reference/distributed-training/)

**Mechanism**:
1. User writes training code using native PyTorch Distributed APIs
2. User creates PyTorchJob with required number of workers and GPUs using Training Operator Python SDK
3. Training Operator creates Kubernetes pods with environment variables for `torchrun`
4. PyTorch DDP performs ring all-reduce for gradient synchronization

**Supported Strategies**:
- PyTorch Distributed Data Parallel (DDP)
- PyTorch FSDP (Fully Sharded Data Parallel)

### TensorFlow Distributed Training

> "The Training Operator creates the TensorFlow parameter server (PS) and workers for PS distributed training."

**Source**: [Distributed Training Documentation](https://www.kubeflow.org/docs/components/trainer/legacy-v1/reference/distributed-training/)

**Mechanism**:
1. User writes training code using native TensorFlow Distributed APIs
2. User creates TFJob with required number of PSs, workers, and GPUs
3. Training Operator sets appropriate environment variables for `TF_CONFIG`
4. Parameter server splits training data and averages model weights

**Supported Strategies**:
- Parameter Server training
- Multi-workerMirroredStrategy
- CollectiveAllReduceStrategy

### Training Operator Architecture

> "The Training Operator will automatically set `WORLD_SIZE` and `RANK` for the appropriate PyTorchJob worker to perform PyTorch Distributed Data Parallel (DDP)."

**Source**: [Getting Started with PyTorchJob](https://www.kubeflow.org/docs/components/trainer/legacy-v1/getting-started/)

**Key Environment Variables Set by Training Operator**:

| Variable | Description |
|----------|-------------|
| `WORLD_SIZE` | Total number of training processes |
| `RANK` | Index of current process |
| `LOCAL_RANK` | Local rank on the node |
| `MASTER_ADDR` | Address of rank 0 |
| `MASTER_PORT` | Port for communication |

---

## Elastic Training Patterns

### Overview

> "Elastic Training breaks the traditional fixed resource configuration paradigm by enabling dynamic changes to the number of instances participating in distributed training."

**Source**: [Elastic Training with MPI Operator](https://blog.kubeflow.org/elastic%20training/operators/2021/03/15/elastic-training.html)

### Key Benefits

| Benefit | Description |
|---------|-------------|
| **Fault Tolerance** | Any worker can fail as long as at least one survives; training can recover without restarting |
| **Resource Utilization** | Cluster can reduce replicas of lower-priority training workloads during resource stress |
| **Cost Reduction** | Enables use of spot/preemptible instances with significantly lower costs |

**Source**: [Elastic Training Blog](https://blog.kubeflow.org/elastic%20training/operators/2021/03/15/elastic-training.html)

### Cost Impact Data

> "Combined with spot instances, we cut the cost for GPUs from ¥16.21/hour to ¥1.62/hour, reducing the overall cost for the training job by nearly 70%."

**Source**: [Elastic Training with MPI Operator](https://blog.kubeflow.org/elastic%20training/operators/2021/03/15/elastic-training.html)

| Metric | Standard Instance | With Spot + Elastic |
|--------|-------------------|---------------------|
| GPU Cost | ¥16.21/hour | ¥1.62/hour |
| Overall Cost Reduction | - | ~70% |
| Training Speed Improvement | - | 5-10x under same budget |

### Elastic Horovod Architecture

**Horovod v0.20.0+** provides built-in elastic training support:

> "Elastic Horovod provides built-in elastic training support for Horovod. All collective operations coordinated in `hvd.elastic.run`. State synchronized between workers before training. Worker failure/add events trigger reset on all workers."

**Source**: [Elastic Training Blog](https://blog.kubeflow.org/elastic%20training/operators/2021/03/15/elastic-training.html)

### Kubernetes Integration with MPI-Operator

**Original MPI-Operator Issues for Elastic Training**:

| Issue | Problem |
|-------|---------|
| No `discover_hosts.sh` | Launcher pod lacks host discovery mechanism |
| Pods not deleted on scale-down | Worker pods remain when replicas reduced |
| Role not updated on scale-up | Launcher can't create processes on new pods |

**Solution: Dynamic Host Discovery**:

```
MPIJob Controller
    ↓ (podLister lists worker pods)
Filter: status.phase == Running
    ↓
Encode into discover_hosts.sh
    ↓
Update ConfigMap
    ↓
Propagate to launcher pod at /etc/mpi/
```

**Source**: [Elastic Training Blog](https://blog.kubeflow.org/elastic%20training/operators/2021/03/15/elastic-training.html)

### Elastic Training Requirements

1. **Horovod v0.20.0+** with elastic support
2. **`discover_hosts.sh` script** for real-time host detection
3. **Training code wrapped in `hvd.elastic.run`** decorator
4. **State object (`hvd.elastic.State`)** for synchronizing variables

### Production Cost Optimization

> "We reduced our ML training costs by 78%! Setting up Kubeflow with Multitenancy, Google Auth, Spot Instances and Node Group Scale Down to Zero."

**Source**: [How we Reduced ML Training Costs by 78%](https://blog.gofynd.com/how-we-reduced-our-ml-training-costs-by-78-a33805cb00cf)

---

## KServe Inference Serving

### Overview

> "KServe is an open-source project that enables serverless inferencing on Kubernetes. KServe provides performant, high abstraction interfaces for common machine learning (ML) frameworks like TensorFlow, XGBoost, scikit-learn, PyTorch, and ONNX to solve production model serving use cases."

**Source**: [KServe Introduction](https://www.kubeflow.org/docs/components/kserve/introduction/)

### Key Features

- **Kubernetes Custom Resource Definition** for serving ML models
- **Autoscaling**: GPU autoscaling, scale to zero, canary rollouts
- **Model abstraction**: Encapsulates complexity of networking, health checking, server configuration
- **Prediction, pre-processing, post-processing, and explainability** out of the box

**Source**: [KServe Introduction](https://www.kubeflow.org/docs/components/kserve/introduction/)

### KServe Architecture

KServe provides:
1. **InferenceService CRD**: Core Kubernetes custom resource for ML model serving
2. **Transformer**: Pre-processing and post-processing
3. **Predictor**: Framework-specific model servers
4. **Explainer**: Model explanation and interpretability

### Installation with Kubeflow

> "Kubeflow provides Kustomize installation files in the `kubeflow/manifests` repository with each Kubeflow release. For the officially tested and supported installation method, refer to the KServe test workflow in the Kubeflow manifests repository."

**Source**: [KServe Introduction](https://www.kubeflow.org/docs/components/kserve/introduction/)

### Name History

> "KFServing was renamed to KServe in September 2021, when the `kubeflow/kfserving` GitHub repository was transferred to the independent KServe GitHub organization."

**Source**: [KServe Introduction](https://www.kubeflow.org/docs/components/kserve/introduction/)

### Serving LLMs with KServe

> "Learn how to deploy and serve large language models at scale using KitOps for packaging, Kubeflow for orchestration, and KServe for production-grade inference on Kubernetes."

**Source**: [Serving LLMs at Scale with KitOps, Kubeflow, and KServe](https://jozu.com/blog/serving-llms-at-scale-with-kitops-kubeflow-and-kserve/)

---

## Katib Hyperparameter Tuning

### Overview

> "Katib is a Kubernetes-native project for automated machine learning (AutoML). Katib supports hyperparameter tuning, early stopping and neural architecture search (NAS)."

**Source**: [Katib Overview](https://www.kubeflow.org/docs/components/katib/overview/)

### Key Features

- **Framework-agnostic**: Supports TensorFlow, MXNet, PyTorch, XGBoost, and others
- **Rich optimization algorithms**: Bayesian optimization, Random Search, CMA-ES, Hyperband, ENAS, DARTS
- **Integration with Kubeflow Training Operator**: Can optimize hyperparameters for large PyTorchJob models
- **Extensible and portable**: Runs Kubernetes containers for hyperparameter tuning

**Source**: [Katib Overview](https://www.kubeflow.org/docs/components/katib/overview/)

### Katib Core Concepts

| Concept | Description |
|---------|-------------|
| **Experiment** | Single tuning run with objective, search space, and search algorithm |
| **Suggestion** | Proposed hyperparameter values from the tuning process |
| **Trial** | One evaluation iteration corresponding to a worker job |
| **Worker** | Kubernetes resource that evaluates a Trial |

**Source**: [Katib Architecture](https://www.kubeflow.org/docs/components/katib/reference/architecture/)

### Supported Algorithms

| Algorithm | Type |
|-----------|------|
| Bayesian optimization | Hyperparameter tuning |
| Tree of Parzen Estimators (TPE) | Hyperparameter tuning |
| Random Search | Hyperparameter tuning |
| CMA-ES | Hyperparameter tuning |
| Hyperband | Early stopping + tuning |
| ENAS | Neural Architecture Search |
| DARTS | Differentiable Architecture Search |

**Source**: [Katib Overview](https://www.kubeflow.org/docs/components/katib/overview/)

### Integration with Training Jobs

> "Katib is integrated with Kubeflow Training Operator jobs such as PyTorchJob, which allows to optimize hyperparameters for large models of any size."

**Source**: [Katib Overview](https://www.kubeflow.org/docs/components/katib/overview/)

---

## Kubeflow Pipelines

### Overview

> "Kubeflow Pipelines (KFP) is a platform for building and deploying portable and scalable machine learning (ML) workflows using containers on Kubernetes-based systems."

**Source**: [Kubeflow Pipelines Overview](https://www.kubeflow.org/docs/components/pipelines/overview/)

### Key Capabilities

- **Pipeline authoring**: Use KFP Python SDK to define components and pipelines
- **Compilation**: Pipelines compile to intermediate representation YAML
- **Execution**: Submit pipelines to KFP-conformant backend (open source or Google Cloud Vertex AI Pipelines)
- **Artifact management**: Track and visualize pipeline definitions, runs, experiments, and ML artifacts

**Source**: [Kubeflow Pipelines Overview](https://www.kubeflow.org/docs/components/pipelines/overview/)

### Architecture

> "A pipeline declares the logical structure for executing components together as a machine learning (ML) workflow in a Kubernetes cluster; this includes defining execution order and conditions, as well as configuring parameter passing and data flow."

**Source**: [Pipeline Concepts](https://www.kubeflow.org/docs/components/pipelines/concepts/pipeline/)

When a pipeline is executed, the Kubeflow Pipelines backend converts the pipeline into instructions to launch Kubernetes Pods to carry out the steps.

### Pipeline Benefits

- Clear isolation between components
- Can be scheduled to run periodically
- Can run with different input parameters
- Versioning support
- Parallelization support
- Non-blocking GPU access

**Source**: [Kubeflow Pipelines Slides](https://danielholmberg.fi/assets/slides/kubeflow_ittf21.pdf)

---

## Use Cases & Production Deployments

### Spotify

> "As we built these new Machine Learning systems, we started to hit a point where our engineers spent more of their time maintaining data and backend systems in support of the ML-specific code than iterating on the model itself. We realized we needed to standardize best practices and build tooling to bridge the gaps between data, backend, and ML: we needed a Machine Learning platform."

**Source**: [Spotify Engineering Blog](https://engineering.atspotify.com/2019/12/13/the-winding-road-to-better-machine-learning-infrastructure-through-tensorflow-extended-and-kubeflow)

**Key Outcomes**:
- ML engineers focus on designing and analyzing experiments instead of building infrastructure
- Faster time from prototyping to production
- Integration with TensorFlow Extended (TFX) and Kubeflow in their "Paved Road for ML systems"

### CERN

**Use**: On-premises GPUs (Nvidia V100s and T4s) for distributed training, hyperparameter optimization, and model serving.

**Kubeflow Components Used**:
- Jupyter Notebooks for experimentation
- ML Pipelines for workflow automation
- Katib for hyperparameter optimization
- KALE for converting Notebooks to Pipelines
- Distributed Training for model training
- Model Serving for inference

**Source**: [CERN Kubeflow Presentation](https://danielholmberg.fi/assets/slides/kubeflow_ittf21.pdf)

### Kubernetes at Scale (Spotify Case Study)

> "A small percentage of our fleet has been migrated to Kubernetes, and some of the things that we've heard from our internal teams are that they have less of a need to focus on manual capacity provisioning and more time to focus on delivering features for Spotify."

**Source**: [Kubernetes Case Studies - Spotify](https://kubernetes.io/case-studies/spotify/)

**Results**:
- CPU utilization improved 2-3x on average with Kubernetes bin-packing and multi-tenancy
- Services that take 10 million requests per second benefit from autoscaling
- New service creation reduced from ~1 hour to seconds/minutes

### Enterprise Cost Optimization

> "We reduced our ML training costs by 78%! Setting up Kubeflow with Multitenancy, Google Auth, Spot Instances and Node Group Scale Down to Zero."

**Source**: [How we Reduced ML Training Costs by 78%](https://blog.gofynd.com/how-we-reduced-our-ml-training-costs-by-78-a33805cb00cf)

---

## Additional Resources

### Technical Reports & Blog Posts

| Title | Source |
|-------|--------|
| [Elastic Training with MPI Operator and Practice](https://blog.kubeflow.org/elastic%20training/operators/2021/03/15/elastic-training.html) | Kubeflow Blog |
| [Democratizing AI Model Training on Kubernetes: Introducing Kubeflow Trainer V2](https://blog.kubeflow.org/trainer/intro/) | Kubeflow Blog |
| [Kubeflow Trainer v2.2: JAX & XGBoost Runtimes](https://blog.kubeflow.org/kubeflow-trainer-v2.2-release/) | Kubeflow Blog |
| [The Winding Road to Better ML Infrastructure Through TFX and Kubeflow](https://engineering.atspotify.com/2019/12/13/the-winding-road-to-better-machine-learning-infrastructure-through-tensorflow-extended-and-kubeflow) | Spotify Engineering |
| [Cost Efficient Distributed Training with Elastic Horovod](https://medium.com/data-science/cost-efficient-distributed-training-with-elastic-horovod-and-amazon-ec2-spot-instances-599ea35c0700) | Medium |

### Community Resources

| Resource | URL |
|----------|-----|
| Kubeflow Slack | #kubeflow-trainer channel on CNCF Slack |
| Kubeflow GitHub Issues | https://github.com/kubeflow/kubeflow/issues |
| Training Working Group Meetings | AutoML and Training WG |
| Kubeflow Pipelines Community Meeting | Every other Wed 10-11AM (PST) |

### Related CNCF Projects

| Project | Purpose |
|---------|---------|
| Kueue | Kubernetes-native batch scheduler |
| Volcano | Batch scheduling system for Kubernetes |
| JobSet | Kubernetes API for batch jobs |
| Istio | Service mesh (note: can interfere with training operators in user namespaces) |

---

## Notes for Deployment

### Istio Sidecar Injection Issue

> "`PyTorchJob` **doesn't work in a user namespace by default** due to Istio automatic sidecar injection."

**Solution**: Add annotation `sidecar.istio.io/inject: "false"` to disable sidecar injection.

**Source**: [PyTorchJob Documentation](https://www.kubeflow.org/docs/components/trainer/legacy-v1/user-guides/pytorch/)

### Gang Scheduling

For distributed training jobs requiring all workers to be scheduled together, configure gang-scheduling via:
- **Volcano** scheduler
- **scheduler-plugins**

Configure `minAvailable` in `schedulingPolicy` to ensure all replicas are scheduled atomically.

### Kueue Integration

> "Kubeflow Trainer v2 includes initial support for Kueue through Pod Integration, which allows individual training pods to be queued when resources are busy."

**Source**: [Kubeflow Trainer v2 Blog](https://blog.kubeflow.org/trainer/intro/)