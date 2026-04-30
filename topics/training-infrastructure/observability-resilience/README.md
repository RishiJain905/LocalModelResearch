# observability-resilience

## 📌 Overview

Observability and resilience patterns for distributed ML training on Kubernetes — covering dynamic service routing, fault-tolerant logging, metrics, tracing, and failure recovery for elastic training workloads.

## Subtopics

| File | Description |
|------|-------------|
| [Dynamic-Routing.md](./Dynamic-Routing.md) | Kubernetes Service routing for training jobs — ClusterIP, Headless Services, Istio/Envoy mesh, DNS-based worker discovery, multi-tenant routing, JobSet topology routing |
| [Fault-Tolerant-Logs.md](./Fault-Tolerant-Logs.md) | Fault-tolerant logging for ML training — Fluent Bit, Loki, Elasticsearch, log persistence across job preemption/restart, structured logging, checkpoint-aware logging, PyTorch torchrun fault tolerance |

---

## Related

- [Kubernetes Training Infrastructure](../) — Parent topic
- [Kubernetes](https://kubernetes.io/) — k8s.io
- [Istio](https://istio.io/) — Service mesh
- [Grafana Loki](https://grafana.com/docs/loki/latest/) — Log aggregation
- [Fluent Bit](https://docs.fluentbit.io/) — Log processor

*Last updated: 2026-04-30*
