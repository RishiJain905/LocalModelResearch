# Breakeven Analysis

## 📌 Overview

Breakeven analysis for AI infrastructure determines the point at which investing in local/on-premise GPU hardware becomes cheaper than renting cloud compute. The breakeven threshold varies dramatically based on **utilization rate, cloud provider pricing, GPU type, staffing costs, and electricity rates**.

> "Our analysis reveals that on-premise deployment are economically viable, with break-even periods typically within a few months for small models, 2 years for medium models and 5 years for larger models..."
> — [Pan & Wang, arXiv:2509.18101v1](https://arxiv.org/html/2509.18101v1) (Carnegie Mellon University, 2025)

Key insight: breakeven is not a single number. It ranges from **0.3 months (small LLM on RTX 5090)** to **never (on-prem vs budget neo-clouds at low utilization)**.

---

## Definitions & Key Concepts

### Levelized Cost of AI (LCOAI)

Proposed by Curcio ([arXiv:2509.02596](https://arxiv.org/pdf/2509.02596)), LCOAI normalizes total amortized CAPEX plus OPEX by valid inference volume:

```
LCOAI = (CAPEX_amortized + Total OPEX) / Number of Valid Inferences
```

Standard unit: USD per thousand inferences ($/1k inferences).

> "At 10 million annual inferences, vendor APIs significantly outperform self-hosted models on cost; however, self-hosting reaches economic parity at 30–40 million annual inferences due to CAPEX amortization."
> — [Curcio, arXiv:2509.02596](https://arxiv.org/pdf/2509.02596)

### Breakeven Utilization Rate

The minimum percentage of time at which owning dedicated GPU infrastructure becomes cheaper per unit of compute than renting from a cloud provider.

| Source | Breakeven Rate | Context |
|--------|---------------|---------|
| Uptime Institute | **33%** cluster-wide average | Dedicated DGX H100 vs cloud averages |
| Remangu / Introl | **60–70%** sustained | On-prem vs hyperscaler on-demand |
| Spheron | **70%** | On-prem vs their aggressively priced on-demand |

### Key Cost Drivers

| Driver | On-Prem Impact | Cloud Impact |
|--------|---------------|--------------|
| **CapEx** | GPU hardware, servers, InfiniBand, racks | Zero |
| **OpEx** | Electricity, colocation, maintenance (10–15%/yr), IT staffing | Hourly rate (bundled) |
| **Staffing** | $225k–$500k/year per 50–100 GPUs | Abstracted; minimal need |
| **Electricity** | $0.06–$0.30/kWh — varies by geography | Bundled in hourly rate |
| **Data Egress** | Varies by ISP/colo | $0.087–$0.12/GB (hyperscalers) |
| **GPU Failure Rate** | 5–10%/year replacement cost | Provider's problem |
| **Financing** | 8–12% over 3 years if financed | N/A |

---

## Academic Research

### Pan & Wang — "A Cost-Benefit Analysis of On-Premise LLM Deployment"
- **URL:** https://arxiv.org/html/2509.18101v1
- **Authors:** Guanzhong Pan, Haibo Wang (Carnegie Mellon University)
- **Date:** 2025

> Hardware comparison table:
> | GPU | Memory | Power | Price (USD) |
> |---|---|---|---|
> | NVIDIA RTX 5090-32GB | 32 GB | 575 W | $2,000 |
> | NVIDIA A100-80GB | 80 GB | 400 W | $15,000 |

> Cost model:
> ```
> C_local(t) = C_hardware + C_electricity * t
> ```

> Break-even at a glance:
> | Scale | Break-Even Range | Hardware Cost |
> |---|---|---|
> | Small (e.g., Qwen3-30B) | 0.3 – 3 months | ~$2,000 |
> | Medium (e.g., Llama-3.3-70B) | 2.3 – 34 months | $15k – $30k |
> | Large (e.g., Qwen3-235B) | 3.5 – 69.3 months | $60k – $240k |

### Curcio — "LCOAI: A Standardized Economic Metric for AI Deployment Costs"
- **URL:** https://arxiv.org/pdf/2509.02596
- **Author:** Eliseo Curcio
- **Date:** 2025

> "LCOAI = Total CAPEX (amortized) + Total OPEX / Number of Valid Inferences"

> Sensitivity table (annual inference volume):
> | Scenario | LCOAI ($/1k inferences at 10M) |
> |---|---|
> | OpenAI GPT-4.1 API | $15.00 |
> | Anthropic Claude Haiku API | $9.80 |
> | Self-hosted LLaMA-2-13B | $24.80 |

### arXiv — Cloud and AI Infrastructure Cost Optimization
- **URL:** https://arxiv.org/html/2307.12479v2

> "GPU compute represents 40-60% of technical budgets for AI-focused organizations."

> "LLM inference costs have decreased by approximately 10x annually since 2021, with equivalent performance now available at 1/1000th the cost compared to 2021."

---

## Industry Reports & Whitepapers

### Lenovo Press LP2368 — On-Premise vs Cloud: GenAI TCO (2026 Edition)
- **URL:** https://lenovopress.lenovo.com/lp2368-on-premise-vs-cloud-generative-ai-total-cost-of-ownership-2026-edition
- **Date:** February 4, 2026

> "Through a rigorous financial comparison... the report demonstrates that on-premises infrastructure achieves a breakeven point in under four months for high-utilization workloads."

> TCO case: Config A (8x H100) vs Azure on-demand ($98.32/hr):
> ```
> 250,141.8 + 6.37x = 98.32x
> 91.95x = 250,141.8
> => x ≈ 2,720 hours (~3.7 months)
> ```

### Uptime Institute — "Sweat Dedicated GPU Clusters to Beat Cloud on Cost"
- **URL:** https://journal.uptimeinstitute.com/sweat-dedicated-gpu-clusters-to-beat-cloud-on-cost/
- **Author:** Dr. Owen Rogers
- **Date:** January 30, 2025

> "The breakeven point is 33% average cluster utilization."
> "At 40% utilization, dedicated infrastructure is 25% cheaper per unit than cloud."
> "At 8% utilization, dedicated infrastructure costs 4x as much per unit as cloud."

| Scenario | Utilization | Dedicated Cost | Cloud Cost | Result |
|---|---|---|---|---|
| Occasional fine-tuning | ~8% | $250/hr | $66/hr | Dedicated ~300% more expensive |
| Regular retraining | ~40% | $50/hr | $66/hr | Dedicated 25% cheaper |

> Asymmetric risk: "10 percentage points OVER the 33% breakeven: Save $16/unit vs. cloud. 10 percentage points UNDER: Pay $30/unit more than cloud."

### SWFTE — Cloud vs On-Prem AI: Complete TCO Analysis 2026
- **URL:** https://www.swfte.com/blog/cloud-vs-onprem-ai-tco-analysis

> "Organizations can achieve 60-70% of cloud costs with on-premise infrastructure at scale."

> 3-year TCO example (10B tokens/month):
> | | Year 1 | Year 2 | Year 3 | Total |
> |---|---|---|---|---|
> | Cloud API (Claude 3.5) | $1.15M | $1.09M | $1.095M | $3.335M |
> | On-Prem (16x A100) | $639K | $359K | $432K | $1.430M |
> | **Savings** | | | | **$1.905M (57%)** |

> Break-even by configuration:
> | Infrastructure | Break-Even Monthly Volume |
> |---|---|
> | 8x L40S ($79K) | ~800M tokens |
> | 16x A100 ($232K) | ~2.5B tokens |
> | 8x H100 ($335K) | ~3.5B tokens |

### Spheron — LLM Inference On-Premise vs GPU Cloud: 2026 Break-Even Analysis
- **URL:** https://www.spheron.network/blog/llm-inference-on-premise-vs-cloud/
- **Date:** April 2026

> "At under 70% GPU utilization, cloud wins on total cost of ownership. At 80%+ sustained utilization, on-prem can win over a 3-year horizon when priced against hyperscalers."

> Cloud cost model:
> ```
> cloud_annual_cost = price_per_gpu_hr × 8 GPUs × 8,760 hr/yr × utilization_rate
> ```

> Spheron pricing (April 2026):
> | GPU | On-Demand ($/hr/GPU) | Spot ($/hr/GPU) |
> |---|---|---|
> | H100 SXM5 | $2.90 | $0.80 |
> | H100 PCIe | $2.01 | N/A |
> | H200 SXM5 | $4.50 | $1.19 |
> | A100 80GB SXM4 | $1.64 | $0.45 |

---

## Blog Posts & Community Analyses

### Alexa V. — "Home Lab vs Cloud GPU: The Real Cost Framework"
- **URL:** https://medium.com/@velinxs/home-lab-vs-cloud-gpu-the-real-cost-framework-f23738891ee8
- **Date:** February 8, 2026

> "Break-even point: ~4–6 hours of daily GPU usage. Below 4 hrs/day: Renting wins. Above 6 hrs/day sustained: Owning starts to make sense."

> Electricity cost for RTX 4090 at $0.16/kWh: $2.11/day, $64/month, $770/year.

> Crossover table (RTX 4090 vs vast.ai at ~$0.18/hr):
> | Daily Use | Monthly Cloud | Yearly |
> |---|---|---|
> | 4 hrs/day | $21.60 | $262 |
> | 8 hrs/day | $43.20 | $525 |
> | 12 hrs/day | $64.80 | $788 |
> | 24/7 | $129.60 | $1,577 |

### RunPod — "How Much Can a GPU Cloud Save You?"
- **URL:** https://www.runpod.io/blog/gpu-cloud-vs-on-prem-cost-savings
- **Author:** James Sandy (Nov 22, 2024)

> "A real-world analysis of a 4× NVIDIA A100 (80GB) workload over 3 years shows a 50.3% cost savings ($124,146) with cloud compared to on-prem."
> "You could rent that same H100 on Runpod for tens of thousands of hours and still not yet be at your break even point."

### Remangu — GPU Infrastructure for ML Training: Cloud vs On-Prem in 2026
- **URL:** https://remangu.com/blog/gpu-infrastructure-ml-training/
- **Author:** Jan Sechovec

> "P5 on-demand pricing runs approximately $98 per hour for a p5.48xlarge (8x H100). That is around $70,000 per month for a single instance running continuously."
> "Even with reserved pricing, cloud costs for continuous utilization are three to four times higher than on-premises over a three-year horizon."
> "Rule of thumb: On-prem wins unambiguously at >70% sustained utilization over 3 years."

### Introl — Hybrid Cloud Strategy for AI
- **URL:** https://introl.com/blog/hybrid-cloud-ai-strategy-gpu-economics-decision-framework
- **Author:** Blake Crosley

> "AWS H100 price cut: Down 44% in June 2025, from ~$7/hr to ~$3.90/hr."
> "Cloud is now favored for utilization below 60–70%; renting is more economical below 12 hrs/day."
> "Lenovo TCO analysis: On-prem pays for itself in 18 months for continuous workloads."

> "Staff—not hardware—is the largest line item... Staff cost ($225k–$300k over 3 years) is the single biggest expense, yet most teams undercount it."
> "This is a full-time role for every 50 to 100 GPUs."

### Civo — GPU Cloud vs Traditional On-Premise
- **URL:** https://www.civo.com/blog/gpu-cloud-vs-traditional-on-premise
- **Date:** April 10, 2026

> "Current on-demand H100 rental: $1.49–$6.98/hour. Early 2025 comparison: AWS up to $7.57/hr, Google Cloud $11.06/hr."
> "On-premise or private cloud infrastructure typically becomes more cost-effective when GPU utilization is high and consistent over time."

---

## Tools & Calculators

| Tool | Type | URL | Notes |
|------|------|-----|-------|
| **Local AI Master — Cloud GPU vs Local Hardware Calculator** | Web-based interactive | https://localaimaster.com/tutorials/cloud-vs-local-calculator | At 80 hrs/month, RTX 4090 vs RunPod: break-even 28.7 months |
| **SLYD — AI Infrastructure TCO Calculator** | Enterprise-grade (3–7 year) | https://slyd.com/resources/tco-calculator | 8-GPU server, 5-year on-prem TCO = $842,050 vs cloud $3.10M |
| **Pan & Wang Interactive Calculator** | Interactive breakeven tool | https://v0-ai-cost-calculator.vercel.app/ | Models 54 deployment scenarios |
| **CostFormation (GitHub)** | Open-source multi-cloud cost calc | https://github.com/gauthamarcot/CostFormation | MIT license. Supports AWS, Azure, GCP pricing APIs |
| **Reddit TCO Simulator (r/LocalLLaMA)** | Community spreadsheet | https://www.reddit.com/r/LocalLLaMA/comments/1r12j7a/i_built_a_tco_simulator_to_find_the_breakeven/ | User-built breakeven analysis tool |

---

## Benchmark Data & Breakeven Points

### Enterprise-Scale Infrastructure (8-GPU Servers)

| Scenario | Cloud Config | On-Prem Config | Breakeven Time | Source |
|---|---|---|---|---|
| 8x H100 vs Azure on-demand ($98.32/hr node) | Azure ND96isr H100 v5 | 8x H100 server ($250K) | **~3.7 months** | Lenovo LP2368 |
| 8x H100 vs AWS on-demand (~$4.10–$6.88/GPU/hr) | AWS p5.48xlarge | 8x H100 server | **50–83% utilization** (~6–18 months) | Spheron / Remangu |
| 8x H100 vs competitive cloud ($2.90/GPU/hr) | Spheron H100 SXM5 | 8x H100 server | **On-prem never wins** | Spheron |
| Continuous utilization (24/7) | Hyperscaler on-demand | 8x H100 + staff + power | **7–12 months** | Introl / MobiDev |
| Continuous utilization | Hyperscaler reserved | 8x H100 + staff + power | **4 months** | Lenovo LP2368 |

### Small / Consumer-Scale (Single GPU)

| GPU | Purchase Cost | Cloud Equiv. Rate | Simple B/E | True TCO B/E | Source |
|---|---|---|---|---|---|
| RTX 4090 | ~$1,700–$2,400 | ~$0.18–$0.74/hr | 11.5–23 months | **20–30 months** | Local AI Master / Alexa V. |
| RTX 4090 (used, hobbyist) | ~$1,200 + $800 system | ~$0.18/hr (vast.ai) | ~4–6 hrs/day usage | ~2 years | Alexa V. Medium |
| A100 80GB | ~$10K–$15K | ~$1.04–$4.10/hr | Varies widely | ~7–18 months | Multiple |

### By Utilization Rate (Uptime Institute Benchmark)

| Average Cluster Utilization | Dedicated vs Cloud Result |
|---|---|
| 8% | Dedicated is **~4x more expensive** |
| 20% | Dedicated is more expensive |
| **33%** | **Breakeven point** |
| 40% | Dedicated is **25% cheaper** |
| 70%+ | Dedicated is significantly cheaper |

### By Model Size / Token Volume (Pan & Wang)

| Model Class | Hardware | Break-Even Range |
|---|---|---|
| Small (~30B params, e.g., Qwen3-30B on RTX 5090) | ~$2,000 | **0.3 – 3 months** |
| Medium (~70B params, e.g., Llama-3.3-70B on 1x A100) | ~$15,000 | **2.3 – 34 months** |
| Large (~235B+ params, e.g., Qwen3-235B on 4x A100) | ~$60,000 | **3.5 – 69 months** |

### By API Token Volume (Swfte / Curcio)

| Monthly Token Volume | Recommendation | Expected Savings |
|---|---|---|
| < 800M tokens | Cloud API / 8x L40S not justified | — |
| ~800M–2.5B tokens | Evaluate on-prem (8x L40S or 16x A100) | Break-even ~6–18 months |
| 10B tokens/month | Strong on-prem case (16x A100) | **57% over 3 years** |
| 30–40M inferences/year | Self-hosted reaches parity with APIs | Curcio LCOAI |

---

## Debates & Conflicting Viewpoints

### On-Premise is Dramatically Cheaper at High Utilization
- **Lenovo (2026):** "Breakeven in under four months for high-utilization workloads... owning the infrastructure yields up to an 18x cost advantage per million tokens."
- **Swfte:** "3-year TCO: on-premise saves $1.9M (57%) for 10B tokens/month."
- **Remangu:** "Continuous utilization: cloud costs are three to four times higher than on-prem over three years."

### Cloud Wins at Any Utilization If Prices Are Competitive
- **Spheron (2026):** "At Spheron's current prices, on-demand cloud costs less than on-prem even at 100% utilization... If your GPU utilization is under 70% or you need capacity this week, cloud is the lower-risk path."
- **RunPod:** "A real-world analysis of a 4× NVIDIA A100 workload over 3 years shows a 50.3% cost savings with cloud compared to on-prem."

### The Utilization Rate Disagreement

| Source | Breakeven Rate | Why the Difference |
|--------|---------------|------------------|
| **Uptime Institute** (Rogers) | **33%** | Compared colo-hosted DGX H100 against averaged cloud providers (including budget providers) |
| **Spheron / Remangu / Introl** | **60–80%** | Compared against their own aggressively priced on-demand or specific hyperscaler rates |

> Asymmetric risk: "10 percentage points OVER the 33% breakeven: Save $16/unit vs. cloud. 10 percentage points UNDER: Pay $30/unit more than cloud."
> — [Uptime Institute](https://journal.uptimeinstitute.com/sweat-dedicated-gpu-clusters-to-beat-cloud-on-cost/)

### Conflicting Staffing Assumptions
- **Lenovo enterprise TCO** assumes existing data center staff.
- **Spheron / Introl** warn:

> "Staff—not hardware—is the largest line item... Staff cost ($225k–$300k over 3 years) is the single biggest expense, yet most teams undercount it. This is a full-time role for every 50 to 100 GPUs."

### "Cloud Costs Are Dropping Faster Than Hardware"
> "The net effect is that cloud API costs are dropping faster than hardware costs, which gradually raises the volume threshold required to justify on-premise investment."
> — [SWFTE](https://www.swfte.com/blog/cloud-vs-onprem-ai-tco-analysis)

This suggests dynamic breakeven: what is true in 2025 may shift by 2026 as frontier API prices fall.

### Depreciation & Obsolescence Risk
> "Rapid hardware obsolescence (e.g., V100→H100 offered 9x perf/$ improvement)... Reserved instances save 40–70% but require 3-year commitments—risky given rapid hardware obsolescence."> — [Introl](https://introl.com/blog/hybrid-cloud-ai-strategy-gpu-economics-decision-framework)

> "GPUs depreciate fast. A $1,200 used 4090 in 2026 might be worth $600 in 2027."> — [Alexa V.](https://medium.com/@velinxs/home-lab-vs-cloud-gpu-the-real-cost-framework-f23738891ee8)

### Electricity as a Decisive Factor
- At **$0.06/kWh** (Pacific NW industrial), on-prem is far more attractive.
- At **$0.25–$0.30/kWh** (California / Europe), annual electricity for one 24/7 GPU reaches $1,200–$1,400, erasing ownership advantages.

---

## Summary Table: Breakeven by GPU Type / Scenario

| GPU / Configuration | Workload Type | Cloud Price Assumption | On-Prem TCO Assumption | Breakeven Point | Source Key |
|---|---|---|---|---|---|
| **RTX 4090 (consumer)** | Hobby / Research | $0.18–$0.74/hr (vast.ai/RunPod) | $2,000 hardware + $64/mo elec | **4–6 hrs/day (~20–30 months true TCO)** | Alexa V.; Local AI Master |
| **A100 80GB (single)** | Fine-tuning / Dev | $1.04–$4.10/hr | ~$15,000 hardware | **7–18 months** (varies by cloud) | Spheron; RunPod |
| **8x H100 SXM5** | Enterprise training/inference | Hyperscaler on-demand ($4.10–$6.88/GPU/hr) | $250K–$350K hardware + staff + power | **3.7–7 months** at high util (~50–83%) | Lenovo; Spheron; Remangu |
| **8x H100 SXM5** | Enterprise training/inference | Budget cloud ($2.01–$2.90/GPU/hr) | $250K–$350K hardware + staff + power | **On-prem never wins on pure cost** | Spheron |
| **8x L40S** | Inference (7B–70B models) | Cloud API / IaaS | $79K hardware + elec | **~800M tokens/month** | Swfte |
| **16x A100** | Large-scale inference | Cloud API ($9/M tokens) | $232K hardware + elec | **~2.5B tokens/month** | Swfte |
| **Small LLM (30B)** | SME inference | API ($3–$15/M tokens) | ~$2,000 (RTX 5090) | **0.3–3 months** | Pan & Wang |
| **Medium LLM (70B)** | Mid-size inference | API ($3–$15/M tokens) | ~$15,000 (1x A100) | **2.3–34 months** | Pan & Wang |
| **Large LLM (235B+)** | Enterprise inference | API ($3–$75/M tokens) | ~$60K–$240K (4–16x A100) | **3.5–69 months** | Pan & Wang |
| **Generic cluster (any GPU)** | Mixed AI workloads | Average cloud on-demand | Dedicated colo-hosted | **33% average utilization** | Rogers / Uptime Institute |

---

## Factor Quick Reference

| Factor | On-Prem Impact | Cloud Impact |
|---|---|---|
| **Utilization Rate** | Dominant driver: must stay above 33–70% | Pay only for active time; no penalty for idleness |
| **GPU Type** | H100/B200 very expensive upfront; RTX very cheap | Higher-end GPUs cost substantially more per hour |
| **Electricity Cost** | $0.06–$0.30/kWh drastically shifts TCO | Included in hourly rate (hidden) |
| **Depreciation Schedule** | 3–5 year enterprise lifecycle; consumer GPUs irrelevant after 2–3 yrs | N/A |
| **Spot Pricing Premium** | N/A | Spot discounts 60–75% make cloud extremely competitive |
| **Data Egress** | Varies by ISP/colo | Hyperscalers: $0.087–$0.12/GB; can add $31k–$43k/yr at 1TB/day |
| **Staffing** | $50k–$500k/year depending on scale | Abstracted; minimal in-house need |
| **Data Gravity** | Co-locate compute with storage to avoid fees | Moving 1PB out of AWS = $92,000 |
| **Lead Time / Scaling** | 2–16 weeks procurement | Minutes to provision |

---

## Source Index

1. [Lenovo Press LP2368](https://lenovopress.lenovo.com/lp2368-on-premise-vs-cloud-generative-ai-total-cost-of-ownership-2026-edition) — On-Premise vs Cloud GenAI TCO (2026)
2. [Uptime Institute](https://journal.uptimeinstitute.com/sweat-dedicated-gpu-clusters-to-beat-cloud-on-cost/) — Sweat Dedicated GPU Clusters (Rogers, 2025)
3. [arXiv:2509.18101v1](https://arxiv.org/html/2509.18101v1) — Pan & Wang, CMU (2025)
4. [arXiv:2509.02596](https://arxiv.org/pdf/2509.02596) — Curcio, LCOAI (2025)
5. [arXiv:2307.12479v2](https://arxiv.org/html/2307.12479v2) — Cloud and AI Infrastructure Cost Optimization
6. [SWFTE](https://www.swfte.com/blog/cloud-vs-onprem-ai-tco-analysis) — Complete TCO Analysis 2026
7. [Spheron](https://www.spheron.network/blog/llm-inference-on-premise-vs-cloud/) — Break-Even Analysis 2026
8. [RunPod](https://www.runpod.io/blog/gpu-cloud-vs-on-prem-cost-savings) — GPU Cloud Savings (Sandy, 2024)
9. [Remangu](https://remangu.com/blog/gpu-infrastructure-ml-training/) — GPU Infrastructure 2026 (Sechovec)
10. [Introl — Hybrid Cloud](https://introl.com/blog/hybrid-cloud-ai-strategy-gpu-economics-decision-framework) — GPU Economics Decision Framework (Crosley)
11. [Introl — Spot](https://introl.com/blog/spot-instances-preemptible-gpus-ai-cost-savings) — Spot Instances & Preemptible GPUs
12. [Civo](https://www.civo.com/blog/gpu-cloud-vs-traditional-on-premise) — GPU Cloud vs On-Premise
13. [Alexa V., Medium](https://medium.com/@velinxs/home-lab-vs-cloud-gpu-the-real-cost-framework-f23738891ee8) — Home Lab vs Cloud GPU Framework
14. [Local AI Master](https://localaimaster.com/tutorials/cloud-vs-local-calculator) — Cloud vs Local Calculator
15. [SLYD](https://slyd.com/resources/tco-calculator) — AI Infrastructure TCO Calculator
16. [CostFormation, GitHub](https://github.com/gauthamarcot/CostFormation) — Open-source multi-cloud cost calculator
17. [v0-ai-cost-calculator](https://v0-ai-cost-calculator.vercel.app/) — Pan & Wang interactive tool

---

## Related Topics

- [Cloud vs Local TCO](./Cloud-vs-Local-TCO.md) — Full TCO comparison tables, hidden costs, and hybrid strategy
- [Utilization Metrics](../Utilization-Metrics.md) *(planned)* — GPU utilization monitoring and optimization
- [Spot Instances](../Spot-Instances.md) *(planned)* — Preemptible pricing strategies and interruption mitigation
- [GPU Depreciation](../GPU-Depreciation.md) *(planned)* — Asset lifecycle optimization for AI hardware
- [Power & Cooling](../Power-Cooling.md) *(planned)* — Data center PUE and electricity cost analysis
- [LCOAI](../LCOAI.md) *(planned)* — Levelized Cost of AI standard metric
