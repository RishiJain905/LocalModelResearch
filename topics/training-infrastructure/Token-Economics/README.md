# Token-Economics

## 📌 Overview

Token Economics under training infrastructure covers the financial framework for training and serving AI models — specifically the cost-per-token dynamics that determine whether owning GPU hardware locally, renting from the cloud, or using managed model APIs is the right economic decision.

> "GPU compute represents the single largest infrastructure cost, typically consuming **40–60% of technical budgets**."> — [arXiv:2307.12479v2](https://arxiv.org/html/2307.12479v2)

> "By introducing the 'Token Economics' framework, we further quantify the efficiency gap, revealing that owning the infrastructure yields up to an **18x cost advantage per million tokens** compared to Model-as-a-Service APIs."
> — [Lenovo Press LP2368](https://lenovopress.lenovo.com/lp2368-on-premise-vs-cloud-generative-ai-total-cost-of-ownership-2026-edition)

The core economic question is not simply "cloud or local?" — it is a **utilization-rate optimization problem**: at what threshold of sustained GPU usage does capital expenditure break even against operating expenditure?

---

## Subtopics

| Subtopic | File | Description |
|----------|------|-------------|
| **Cloud vs Local TCO** | [Cloud-vs-Local-TCO.md](./Cloud-vs-Local-TCO.md) | Total Cost of Ownership comparison of cloud GPU infrastructure (AWS, GCP, Azure, neo-clouds) vs on-premise. Hardware costs, hourly rates, spot pricing, electricity, staffing, and hidden costs. |
| **Breakeven Analysis** | [Breakeven-Analysis.md](./Breakeven-Analysis.md) | Breakeven timelines, utilization thresholds (33–70%), LCOAI metric, breakeven by GPU type (RTX 4090 → 8× H100), and dynamic breakeven calculus. |

---

## Summary Matrix

| Dimension | Cloud (Hyperscaler) | Cloud (Neo-Cloud) | On-Premise |
|-----------|-------------------|-------------------|------------|
| **H100 On-Demand** | $6.88–$14.24/hr | $2.01–$6.16/hr | ~$1.50–$4.00/hr equiv* |
| **Upfront Cost** | $0 | $0 | $79K–$500K+ |
| **Break-even** | N/A | N/A | 4–14 months @ 70%+ util |
| **Breakeven Utilization** | N/A | N/A | 33–70% (source-dependent) |
| **3-Year TCO Example** | $3.34M (10B tokens/mo) | $1.5M–$2.0M (est.) | $1.43M (10B tokens/mo) |
| **Spot Discount** | Up to 44–70% | Up to 59–64% | N/A |
| **Best For** | Enterprise integration, compliance | Cost-optimized AI workloads | High-utilization, data sovereignty, latency |

*Equivalent hourly assuming 3-year depreciation + power + staffing.

---

## Key Data Points at a Glance

| Metric | Value |
|--------|-------|
| GPU share of AI budgets | **40–60%** |
| On-prem break-even (high util) | **< 4 months** (Lenovo) |
| On-prem break-even (cluster avg) | **33%** utilization (Uptime Institute) |
| On-prem break-even (vs neo-cloud) | **70%+** (Spheron) |
| 3-year on-prem savings | **50–85%** across scenarios |
| Small LLM (30B) break-even | **0.3–3 months** on RTX 5090 (~$2K) |
| Large LLM (235B+) break-even | **3.5–69 months** on 4× A100 (~$60K) |
| Cost advantage per million tokens | **Up to 18×** vs model APIs |

---

## Calculators & Tools

| Tool | Type | URL |
|------|------|-----|
| **SLYD AI TCO Calculator** | Enterprise 3–7 year | https://slyd.com/resources/tco-calculator |
| **Local AI Master Calculator** | Hobbyist → Startup | https://localaimaster.com/tutorials/cloud-vs-local-calculator |
| **Pan & Wang Interactive Tool** | 54 LLM scenarios | https://v0-ai-cost-calculator.vercel.app/ |
| **Silicon Analysts Cluster TCO** | Preset-based | https://siliconanalysts.com/tools/cluster-tco |
| **CostFormation (GitHub)** | Multi-cloud IaC + calc | https://github.com/gauthamarcot/CostFormation |

---

## Planned Subtopics

| Subtopic | Status | Notes |
|----------|--------|-------|
| Spot Instances | ⏳ Planned | AWS, GCP, Spheron spot pricing; interruption mitigation |
| GPU Depreciation | ⏳ Planned | Asset lifecycle, component-based depreciation |
| Power & Cooling / PUE | ⏳ Planned | Data center efficiency, electricity cost by region |
| LCOAI | ⏳ Planned | Levelized Cost of AI standardized metric |
| Token Pricing APIs | ⏳ Planned | Per-million-token pricing comparisons |
| Utilization Metrics | ⏳ Planned | GPU monitoring, scheduling, and optimization |
| Hybrid Load Balancing | ⏳ Planned | Strategies to combine on-prem + cloud bursting |
| Data Gravity & Egress | ⏳ Planned | The hidden $0.09–$0.12/GB tax |

---

## Related Topics

- [Fine-Tuning](../../fine-tuning/) — Fine-tuning methods and recipes (cost of adapters vs full fine-tuning)
- [Hardware-Accelerators](../../Hardware-Accelerators/) — GPU/NPU hardware specs and pricing
- [Inference-Serving](../../Inference-Serving/) — Serving infrastructure costs and throughput optimization
- [API-Servers](../../API-Servers/) — API server deployments and per-request economics
- [Benchmarks](../../benchmarks/) — Throughput and cost-per-token benchmarks

---

*Last updated: 2026-04-27*
