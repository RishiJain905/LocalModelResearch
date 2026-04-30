# Fault-Tolerant Logs: Logging Infrastructure for ML Training Resilience

## Table of Contents

1. [Overview](#overview)
2. [Official Documentation & Repositories](#official-documentation--repositories)
3. [Distributed Logging for Kubernetes Training Jobs](#distributed-logging-for-kubernetes-training-jobs)
   - [Fluentd](#fluentd)
   - [Fluent Bit](#fluent-bit)
   - [Grafana Loki](#grafana-loki)
   - [Elasticsearch (ELK Stack)](#elasticsearch-elk-stack)
4. [Log Persistence Across Job Preemption/Restart](#log-persistence-across-job-preemptionrestart)
5. [Structured Logging for Training Metrics](#structured-logging-for-training-metrics)
6. [Checkpoint-Aware Logging](#checkpoint-aware-logging)
7. [Log Aggregation for Fault Analysis in Distributed Training](#log-aggregation-for-fault-analysis-in-distributed-training)
8. [Comparison Tables](#comparison-tables)
9. [Additional Resources](#additional-resources)

---

## Overview

> "Logs, metrics, and traces are the three distinct signals that form the foundation of observability in Kubernetes."

**Source:** [Kubernetes Observability: Logs, Metrics, and Traces](https://medium.com/@niteshthakur498/kubernetes-observability-logs-metrics-and-traces-see-inside-your-cluster-6b4decb6e8ca)

Fault-tolerant logging for ML training on Kubernetes addresses the challenge of preserving training diagnostics across job preemptions, node failures, and elastic scaling events. In distributed training environments (PyTorch DDP, TensorFlow PS, Horovod), logs from multiple workers must be aggregated and correlated to reconstruct training state for fault analysis.

**Key Requirements:**
- **Log persistence** across pod preemption/restart (especially with Spot/Preemptible instances)
- **Structured logging** for training metrics (loss curves, gradients, learning rates)
- **Checkpoint-aware logging** that correlates log events with saved model states
- **Distributed log aggregation** from multiple training workers/nodes
- **Fault analysis capabilities** to reconstruct failure sequences

---

## Official Documentation & Repositories

### Fluent Bit

| Resource | URL |
|----------|-----|
| Official Documentation | https://docs.fluentbit.io/ |
| GitHub Repository | https://github.com/fluent/fluent-bit |
| CNCF Status | Graduated (CNCF sub-project under Fluentd) |
| Backpressure Documentation | https://docs.fluentbit.io/manual/administration/backpressure |
| Buffering & Storage Configuration | https://docs.fluentbit.io/manual/administration/buffering-and-storage |

### Fluentd

| Resource | URL |
|----------|-----|
| Official Documentation | https://www.fluentd.org/documentation |
| GitHub Repository | https://github.com/fluent/fluentd |
| CNCF Status | Graduated |

### Fluent Operator (Kubernetes Operator)

| Resource | URL |
|----------|-----|
| GitHub Repository | https://github.com/fluent/fluent-operator |
| Description | "This guide provisions a logging pipeline including the Fluent Bit DaemonSet and its log input/filter/output configurations to collect Kubernetes logs including container logs, host logs, and kernel logs." |

### Grafana Loki

| Resource | URL |
|----------|-----|
| Official Documentation | https://grafana.com/docs/loki/latest/ |
| GitHub Repository | https://github.com/grafana/loki |
| CNCF Status | Graduated |
| Loki Overview | https://grafana.com/docs/loki/latest/get-started/overview/ |
| LogQL Query Language | https://grafana.com/docs/loki/latest/query/ |

### Logging Operator (Kube Logging)

| Resource | URL |
|----------|-----|
| Official Documentation | https://kube-logging.github.io/ |
| GitHub Repository | https://github.com/kube-logging/logging-operator |
| CNCF Status | Sandbox (accepted September 19, 2023) |

### Elasticsearch

| Resource | URL |
|----------|-----|
| Official Documentation | https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html |
| Elasticsearch Kubernetes Logging | https://coralogix.com/blog/kubernetes-logging-with-elasticsearch-fluentd-and-kibana/ |

---

## Distributed Logging for Kubernetes Training Jobs

### Fluentd

> "Fluentd is software that's frequently used to collect and aggregate logging. Hosted at https://www.fluentd.org, like prometheus it is open source software."

**Source:** [Packt Subscription - Logging and Tracing](https://subscription.packtpub.com/book/cloud-and-networking/9781788834759/8/ch08lvl1sec66/viewing-logs-using-kibana)

**Architecture for Kubernetes Training Jobs:**
```
Kubernetes Nodes → Fluentd DaemonSet → [Elasticsearch / Loki / S3]
```

**Key Characteristics:**
- Written in Ruby
- Rich plugin ecosystem for Kubernetes log collection
- Environment variable configuration for Elasticsearch hosts
- Can be deployed as DaemonSet collecting from `/var/log` and `/var/lib/docker/containers`

**Example Fluentd DaemonSet Configuration:**
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
  namespace: kube-system
spec:
  template:
    spec:
      containers:
      - name: fluentd
        image: fluent/fluentd-kubernetes-daemonset:v1.11.5-debian-elasticsearch7-1.1
        env:
        - name: FLUENT_ELASTICSEARCH_HOST
          value: "elastic01.demo.local"
        - name: FLUENT_ELASTICSEARCH_PORT
          value: "9200"
```

**Source:** [Fluent Bit Blog - Logging Fluentd with Kubernetes](https://fluentbit.io/blog/2021/01/25/logging-fluentd-with-kubernetes/)

---

### Fluent Bit

> "Fluent Bit is a fast, lightweight logs and metrics agent. It is a CNCF graduated sub-project under the umbrella of Fluentd. Fluent Bit is licensed under the terms of the Apache License v2.0."

**Source:** [Grafana Loki Documentation - Fluent Bit](https://grafana.com/docs/loki/latest/send-data/fluentbit/)

**Why Fluent Bit for ML Training:**

| Aspect | Benefit |
|--------|---------|
| Memory footprint | Small (~10MB), suitable for GPU nodes |
| Performance | 10-40x greater log processing than Fluentd |
| Kubernetes enrichment | Native namespace, pod, labels support |
| Storage options | Memory-based or filesystem-based buffering |
| OpenTelemetry | Full support for logs, metrics, traces |

**Source:** [CNCF Blog - Fluentd to Fluent Bit Migration Guide](https://www.cncf.io/blog/2025/10/01/fluentd-to-fluent-bit-a-migration-guide/)

**Filesystem-Based Buffering for Training Job Persistence:**

> "Guarantees data safety while controlling memory usage. Memory and filesystem buffering are **not mutually exclusive**."

> "When threshold is reached, instead of pausing the plugin, new data goes to Chunks `down` in filesystem—**no data loss**, controlled memory usage."

**Source:** [Chronosphere - Avoiding Data Loss and Backpressure Problems with Fluent Bit](https://chronosphere.io/learn/avoiding-data-loss-and-backpressure-problems-with-fluent-bit/)

**Configuration Example:**
```ini
[SERVICE]
    flush                       1
    log_level                   info
    storage.path                /var/log/flb-storage/
    storage.sync                normal
    storage.checksum            off
    storage.max_chunks_up       5

[INPUT]
    Name              tail
    Path              /var/log/containers/*.log
    storage.type      filesystem
```

**Persistent Storage with Amazon EFS:**

> "In this blog, I cover how to persist logs from your Amazon Elastic Kubernetes Service (Amazon EKS) containers on highly durable, highly available, elastic, and scalable storage."

**Source:** [AWS Blog - Persistent Storage for Container Logging Using Fluent Bit and Amazon EFS](https://aws.amazon.com/blogs/storage/persistent-storage-for-container-logging-using-fluent-bit-and-amazon-efs/)

---

### Grafana Loki

> "Loki is a **horizontally-scalable, highly-available, multi-tenant log aggregation system** inspired by Prometheus."

**Source:** [Grafana Loki Documentation - Overview](https://grafana.com/docs/loki/latest/get-started/overview/)

**Core Philosophy:**

> "Loki does **not** index the contents of logs—only metadata (labels) about your logs."

> "A log stream is a set of logs which share the same labels. Labels help Loki to find a log stream within your data store, so having a quality set of labels is key to efficient query execution."

**Architecture:**
```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Agent     │────▶│    Loki     │────▶│   Grafana   │
│ (Alloy/     │     │   Server    │     │  (Query &   │
│  Promtail)  │     │             │     │   Display)  │
└─────────────┘     └─────────────┘     └─────────────┘
```

**Key Advantages for ML Training:**

| Feature | Description |
|---------|-------------|
| Cost-efficient storage | Label-only indexing, compressed chunks |
| Horizontal scalability | Runs from Raspberry Pi to petabytes/day |
| PromQL-inspired query | LogQL enables metric extraction from logs |
| Multi-tenancy | Multiple training jobs share Loki instance |
| Storage backends | S3, GCS, filesystem for Kubernetes PVs |

**LogQL for Training Metrics:**

> "LogQL is directly inspired by PromQL. Every query starts with a stream selector — label filters."

**Source:** [Medium - Master Kubernetes Logging: Grafana Loki + Alloy](https://medium.com/@mdsraihaniqbal1999/master-kubernetes-logging-grafana-loki-alloy-made-simple-72695589ae05)

**Example LogQL Query for Training Errors:**
```logql
{service="training-job"} |= "ERROR" |= "gradient" | json | line_format "{{.timestamp}} {{.message}}"
```

---

### Elasticsearch (ELK Stack)

> "Loki vs Elasticsearch: The choice of **Loki vs Elasticsearch for Kubernetes logs** directly impacts your storage costs, diagnosis time, and the skills required from your teams."

**Source:** [SFEIR Institute - Loki vs Elasticsearch](https://institute.sfeir.com/en/kubernetes-training/loki-vs-elasticsearch-logs-kubernetes/)

**Comparison for ML Training Workloads:**

| Aspect | Elasticsearch | Loki |
|--------|---------------|------|
| Indexing | Full-text indexing | Label-only indexing |
| Storage cost | Higher (full index) | Lower (object storage) |
| Query flexibility | Full-text search | Label-based + LogQL |
| Infrastructure | Heavy (Java heap) | Lightweight |
| Best for | Deep log analysis | High-volume training logs |

**ILM (Index Lifecycle Management) for Kubernetes Logs:**

> "This guide covers how to design and implement ILM policies specifically tailored for Kubernetes log data."

**Source:** [OneUptime - Elasticsearch ILM Kubernetes Logs](https://oneuptime.com/blog/post/2026-02-09-elasticsearch-ilm-kubernetes-logs/view)

**Fluent Bit to Elasticsearch Integration:**
```yaml
[OUTPUT]
    Name              elasticsearch
    Match             *
    Host              elasticsearch.logging
    Port              9200
    Index             kubernetes-logs-%Y%m%d
    HTTP_User         elastic
    HTTP_Passwd       ${ELASTIC_PASSWORD}
    tls               On
   ，阿里云.log
```

---

## Log Persistence Across Job Preemption/Restart

### The Challenge

> "One of the most effective methods for retaining Kubernetes job logs beyond the lifecycle of a pod is to utilize persistent storage."

**Source:** [WafaTech - Best Practices for Managing Kubernetes Job Logs](https://wafatech.sa/blog/devops/kubernetes/best-practices-for-managing-kubernetes-job-logs/)

### Filesystem-Based Buffering (Fluent Bit)

> "The problems that result from backpressure include high memory usage, service downtime, and data loss, all of which we would like to avoid."

**Source:** [Chronosphere - Avoiding Data Loss and Backpressure Problems with Fluent Bit](https://chronosphere.io/learn/avoiding-data-loss-and-backpressure-problems-with-fluent-bit/)

**Memory vs Filesystem Buffering Comparison:**

| Approach | Memory Control | Data Safety | Trade-off |
|----------|---------------|-------------|-----------|
| Default (memory) | ❌ | ❌ | Fastest, but risk of OOM |
| `mem_buf_limit` | ✅ | ❌ | May lose data |
| Filesystem buffering | ✅ | ✅ | Best solution for training jobs |

### Kubernetes Volumes for Training Logs

**EmptyDir Volume:**
> "An emptyDir volume recreates when containers restart or crash."

**Source:** [Medium - Kubernetes Volume Basics: emptyDir and PersistentVolume](https://www.alibabacloud.com/blog/kubernetes-volume-basics-emptydir-and-persistentvolume_594834)

**Use Case for Training Jobs:**
- Store intermediate training artifacts
- Buffer logs before Fluent Bit collection
- Temporary storage for checkpoint metadata

**PersistentVolumeClaim for Training Logs:**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: training-logs-pvc
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: efs-sc
  resources:
    requests:
      storage: 10Gi
```

### Spot/Preemptible Instance Handling

> "Combined with spot instances, we cut the cost for GPUs from ¥16.21/hour to ¥1.62/hour, reducing the overall cost for the training job by nearly 70%."

**Source:** [Kubeflow Blog - Elastic Training with MPI Operator](https://blog.kubeflow.org/elastic%20training/operators/2021/03/15/elastic-training.html)

**Logging Considerations for Spot Instances:**
- Fluent Bit filesystem buffering prevents log loss during node termination
- Stateful log storage (EFS, NFS) enables log retention across preemptions
- Log aggregation must handle worker node churn

---

## Structured Logging for Training Metrics

### MLflow for Distributed Training

> "Using TensorBoard when you need MLflow — or vice versa — means fighting your tools instead of training your models."

**Source:** [MLOps Lab - MLflow vs TensorBoard](https://www.mlopslab.org/mlflow-vs-tensorboard-which-experiment-tracker-should-you-use-in-2026/)

**Key Findings for Distributed Training:**
- MLflow tracks parameters, metrics, code versions, data versions, and trained model artifacts
- Distributed training with PyTorch/TensorFlow requires special handling to avoid duplicate logging

**Source:** [GitHub MLflow Issue #5798](https://github.com/mlflow/mlflow/issues/5798)

> "I would like to know if there is any documentation or example for logging mlflow metrics with distributed training using tensorflow or pytorch."

### PyTorch Lightning Structured Logging

> "To use a logger, simply pass it into the Trainer. Lightning uses TensorBoard by default."

**Source:** [PyTorch Lightning Documentation](https://pytorch-lightning.readthedocs.io/en/1.1.8/logging.html)

**Supported Loggers:**
- TensorBoard (default)
- MLflow
- Comet
- Neptune
- Weights & Biases (wandb)

### NVIDIA NeMo Framework Logging

> "The `NeMoLogger` class is responsible for setting the logging directories for NeMo runs."

**Source:** [NVIDIA NeMo Framework User Guide](https://docs.nvidia.com/nemo-framework/user-guide/24.07/nemo-2.0/features/logging.html)

**Core Classes:**
- `Logger` - Sets logging directories and configures trainer loggers
- `ModelCheckpoint` - Manages checkpointing directory and resume logic

### PyTorch Metrics Logging

> "Optimizer step optimizer.step() # --- Logging steps --- # Accumulate loss (use .item() to get Python number) running_loss += loss.item() * inputs.size(0)"

**Source:** [APX Maven - Log Metrics in PyTorch Training](https://apxml.com/courses/getting-started-with-pytorch/chapter-8-monitoring-debugging-models/logging-metrics-training-evaluation)

**Best Practices for Distributed Training Logging:**
- Use `torch.distributed.reduce` instead of `all_reduce` for logging aggregated metrics
- Rank 0 process typically handles logging to avoid duplicates

**Source:** [PyTorch Discussion - Best Practices of Logging in Distributed Training](https://discuss.pytorch.org/t/what-is-the-best-practices-of-logging-in-distributed-training/68286)

---

## Checkpoint-Aware Logging

### TorchElastic Fault Tolerance

> "PyTorch offers a utility called torchrun that provides fault-tolerance and elastic training. When a failure occurs, torchrun logs the errors."

**Source:** [PyTorch Tutorials - Fault-tolerant Distributed Training with torchrun](https://docs.pytorch.org/tutorials/beginner/ddp_series_fault_tolerance.html)

**Key Features:**
- Automatic restart on node failure
- Dynamic worker membership changes
- Preserves training state through checkpoint coordination

### Kubernetes Training Operator Checkpoint Integration

> "The TorchElastic Controller for Kubernetes reduces distributed training time and cost by limiting idle cluster resources and training on Amazon EC."

**Source:** [AWS Blog - Fault Tolerant Distributed ML Training with TorchElastic Controller](https://aws.amazon.com/blogs/containers/fault-tolerant-distributed-machine-learning-training-with-the-torchelastic-controller-for-kubernetes/)

**Checkpoint-Aware Logging Requirements:**
- Log event timestamps must correlate with checkpoint iterations
- Training metrics should include checkpoint epoch/step number
- Failure analysis requires log correlation with last successful checkpoint

### Kubeflow Trainer v2 Fault Handling

> "KF Trainer v2 improves handling these faults by supporting **Kubernetes PodFailurePolicy**, which allows users to **define specific rules** for handling different types of failures, such as restarting the job after temporary node issues or terminating the job after critical errors."

**Source:** [Kubeflow Blog - Democratizing AI Model Training on Kubernetes](https://blog.kubeflow.org/trainer/intro/)

**PodFailurePolicy Integration:**
- Restart after temporary node issues
- Terminate after critical errors
- Configurable failure handling per training job

---

## Log Aggregation for Fault Analysis in Distributed Training

### Architecture Components

```
┌─────────────────────────────────────────────────────────────────┐
│                    Training Job Pods                            │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐      │
│  │ Worker 0 │  │ Worker 1 │  │ Worker 2 │  │ Worker N │      │
│  │  (Rank 0)│  │  (Rank 1)│  │  (Rank 2)│  │ (Rank N) │      │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘      │
│       │             │             │             │              │
│       └─────────────┴─────────────┴─────────────┘              │
│                         │                                      │
│              Fluent Bit Sidecar/DaemonSet                       │
│                         │                                      │
└─────────────────────────┼───────────────────────────────────────┘
                          │
                    ┌─────▼─────┐
                    │   Loki    │
                    │  (or ES)  │
                    └─────┬─────┘
                          │
                    ┌─────▼─────┐
                    │  Grafana  │
                    │  (Query)  │
                    └───────────┘
```

### Kubeflow Training Job Fault Analysis

**PyTorchJob Example with Logging:**
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
```

**Key Environment Variables for Fault Analysis:**
| Variable | Description |
|----------|-------------|
| `WORLD_SIZE` | Total number of training processes |
| `RANK` | Index of current process |
| `LOCAL_RANK` | Local rank on the node |
| `MASTER_ADDR` | Address of rank 0 |
| `MASTER_PORT` | Port for communication |

**Source:** [Kubeflow Documentation - Getting Started with PyTorchJob](https://www.kubeflow.org/docs/components/trainer/legacy-v1/getting-started/)

### Log Stream Selection for Training Jobs

**Loki Label-Based Filtering:**
```logql
{service="pytorch-job", namespace="ml-training", job_name="distributed-bert"}
  |= "ERROR"
  |= "gradient Nan"
  | json
  | line_format "Rank {{.rank}} Epoch {{.epoch}}: {{.message}}"
```

**Key Labels for Training Job Identification:**
- `namespace` - Kubernetes namespace
- `job_name` - Training job name
- `replica_type` - Master/Worker distinction
- `rank` - Process rank in distributed training
- `framework` - PyTorch/TensorFlow/Horovod

### LogQL for Training Fault Analysis

**Metric Extraction from Logs:**
```logql
sum_over_time(
  {service="training"} |= "loss" | json | unwrap loss[5m]
)
```

**Error Rate Calculation:**
```logql
sum(rate({service="training"} |= "ERROR"[5m])) by (job_name)
```

**Source:** [Grafana Loki - Metric Queries](https://grafana.com/docs/loki/latest/query/metric_queries/)

---

## Comparison Tables

### Log Collectors Comparison

| Feature | Fluent Bit | Fluentd | Loki (Agent) | Elasticsearch |
|---------|-----------|---------|--------------|---------------|
| **Memory footprint** | ~10MB | ~40-80MB | ~50MB | Heavy (ES cluster) |
| **CNCF status** | Graduated | Graduated | Graduated | Independent |
| **Kubernetes native** | ✅ DaemonSet | ✅ DaemonSet | ✅ Promtail/Alloy | ✅ Operator |
| **Structured logging** | ✅ JSON parser | ✅ JSON parser | ✅ Pipeline stages | ✅ Ingest pipeline |
| **Filesystem buffering** | ✅ | ❌ | N/A | N/A |
| **ML training use case** | Edge/GPU nodes | Aggregator | Cost-effective aggregation | Deep analysis |

**Source:** [CNCF Blog - Logstash, Fluentd, Fluent Bit, or Vector](https://www.cncf.io/blog/2022/02/10/logstash-fluentd-fluent-bit-or-vector-how-to-choose-the-right-open-source-log-collector/)

### Log Storage Solutions for ML Training

| Solution | Best For | Scalability | Cost | Query Capability |
|----------|----------|-------------|------|-------------------|
| **Loki** | High-volume training logs | Petabytes/day | Low (object storage) | LogQL + metrics |
| **Elasticsearch** | Deep fault analysis | TB/day | High (full index) | Full-text search |
| **CloudWatch Logs** | AWS-native training | Managed | Medium | SQL-like queries |
| **Google Cloud Logging** | GKE-native training | Managed | Medium | Log Explorer |

### Kubernetes Volume Options for Training Logs

| Volume Type | Persistence | Use Case |
|-------------|--------------|----------|
| **emptyDir** | Ephemeral (pod lifetime) | Temporary buffering |
| **hostPath** | Node-persistent | Debug logs |
| **PersistentVolumeClaim** | Persistent across pods | Long-term log archival |
| **EFS/NFS** | Shared across nodes | Distributed training logs |
| **Object Storage (S3/GCS)** | Cloud-native persistent | Centralized log storage |

---

## Additional Resources

### Official Documentation Links

| Resource | URL |
|----------|-----|
| Fluent Bit Documentation | https://docs.fluentbit.io/ |
| Fluent Operator GitHub | https://github.com/fluent/fluent-operator |
| Grafana Loki Overview | https://grafana.com/docs/loki/latest/get-started/overview/ |
| LogQL Reference | https://grafana.com/docs/loki/latest/query/ |
| Logging Operator (Kube Logging) | https://kube-logging.github.io/ |
| PyTorch Fault Tolerance Tutorial | https://docs.pytorch.org/tutorials/beginner/ddp_series_fault_tolerance.html |
| Kubeflow Training Operator | https://github.com/kubeflow/training-operator |
| Kubeflow Trainer v2 | https://github.com/kubeflow/trainer |

### Blog Posts and Technical Reports

| Title | Source |
|-------|--------|
| [Fault-tolerant Distributed ML Training with TorchElastic Controller](https://aws.amazon.com/blogs/containers/fault-tolerant-distributed-machine-learning-training-with-the-torchelastic-controller-for-kubernetes/) | AWS |
| [Fluentd to Fluent Bit: A Migration Guide](https://www.cncf.io/blog/2025/10/01/fluentd-to-fluent-bit-a-migration-guide/) | CNCF |
| [Elastic Training with MPI Operator](https://blog.kubeflow.org/elastic%20training/operators/2021/03/15/elastic-training.html) | Kubeflow Blog |
| [Logging Operator (Kube Logging) - CNCF](https://www.cncf.io/projects/logging-operator-kube-logging/) | CNCF |
| [Avoiding Data Loss and Backpressure Problems with Fluent Bit](https://chronosphere.io/learn/avoiding-data-loss-and-backpressure-problems-with-fluent-bit/) | Chronosphere |
| [Loki vs Elasticsearch for Kubernetes](https://institute.sfeir.com/en/kubernetes-training/loki-vs-elasticsearch-logs-kubernetes/) | SFEIR Institute |
| [Kubernetes Logging Strategy: Production-Grade Log Aggregation 2026](https://hostperl.com/blog/kubernetes-logging-strategy-production-grade-log-aggregation-2026) | Hostperl |

### GitHub Repositories

| Repository | Description |
|------------|-------------|
| [fluent/fluent-bit](https://github.com/fluent/fluent-bit) | Fluent Bit - Log collector agent |
| [fluent/fluentd](https://github.com/fluent/fluentd) | Fluentd - Log collector aggregator |
| [fluent/fluent-operator](https://github.com/fluent/fluent-operator) | Kubernetes operator for Fluent Bit/Fluentd |
| [grafana/loki](https://github.com/grafana/loki) | Loki - Log aggregation system |
| [kube-logging/logging-operator](https://github.com/kube-logging/logging-operator) | CNCF Logging Operator |
| [kubeflow/training-operator](https://github.com/kubeflow/training-operator) | Kubeflow Training CRDs |
| [lenisha/TorchElastic_Lab](https://github.com/lenisha/TorchElastic_Lab) | TorchElastic on Kubernetes Lab |

---

*Last updated: 2026-04-30*
