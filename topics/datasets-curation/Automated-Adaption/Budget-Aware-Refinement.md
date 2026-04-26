# Budget-Aware Refinement

> **📌 Overview**
> **Budget-Aware Refinement** refers to algorithmic strategies that explicitly incorporate resource constraints — computational budget, token budget, inference cost, or training time — into model optimization. Foundational work dates to ICLR 2020 (Budgeted Training). Recent work (2024–2026) focuses heavily on LLM reasoning and agentic systems where token/compute budgets are critical. See also: [AutoAdapt Framework](./AutoAdapt-Framework.md) which includes AutoRefine as a component.

---

## Definitions

### Core Definition

> *"What is the best performance one can achieve given this data and algorithm within the allowed budget?"*

**Source:** [Budgeted Training: Rethinking Deep Neural Network Training Under Resource Constraints](https://arxiv.org/abs/1905.04753) — ICLR 2020

### Related Terminology

| Term | Description |
|------|-------------|
| **Budgeted Training** | Training neural networks under non-asymptotic, resource-constrained regimes |
| **Token Budget Reasoning** | Allocating reasoning tokens efficiently in LLMs |
| **Cost-Aware Optimization** | Methods that balance accuracy gains against computational cost |
| **Compute Budget Adaptation** | Adjusting model behavior based on available inference/training resources |
| **Anytime Algorithms** | Methods producing incrementally better solutions over time |

---

## Key Sources (Papers & Reports)

### 1. Budgeted Training (ICLR 2020)

| Field | Value |
|-------|-------|
| **Title** | Budgeted Training: Rethinking Deep Neural Network Training Under Resource Constraints |
| **URL** | [https://arxiv.org/abs/1905.04753](https://arxiv.org/abs/1905.04753) |
| **Venue** | ICLR 2020 |
| **Authors** | Mengtian Li (CMU), Ersin Yumer (Uber ATG), Deva Ramanan (CMU & Argo AI) |
| **Project Page** | [http://www.cs.cmu.edu/~mengtial/proj/budgetnn/](http://www.cs.cmu.edu/~mengtial/proj/budgetnn/) |

> *"We introduce a formal setting for studying training under the non-asymptotic, resource-constrained regime, i.e., budgeted training. We analyze the problem through the lens of learning rate schedule design."*

**Key Contribution:** Linear learning rate schedule (β_p = 1 - p) that decays to zero by end of budget, achieving budgeted convergence. The schedule is parameter-free and outperforms step decay by up to **+50% at 1% budget on ImageNet**.

---

### 2. ChipNet: Budget-Aware Pruning (ICLR 2021)

| Field | Value |
|-------|-------|
| **Title** | ChipNet: Budget-Aware Pruning with Heaviside Continuous Approximations |
| **URL** | [https://arxiv.org/pdf/2102.07156](https://arxiv.org/pdf/2102.07156) |
| **Venue** | ICLR 2021 |
| **Authors** | Rishabh Tiwari, Udbhav Bamba, Arnav Chavan, Deepak Gupta (transmuteAI) |
| **GitHub** | [https://github.com/transmuteAI/ChipNet](https://github.com/transmuteAI/ChipNet) |
| **OpenReview** | [https://openreview.net/forum?id=xCxXwTzx4L1](https://openreview.net/forum?id=xCxXwTzx4L1) |

> *"ChipNet: Budget-Aware Pruning with Heaviside Continuous Approximations. Structured pruning methods are among the effective strategies for extracting small resource-efficient convolutional neural networks from larger models."*

**Key Contribution:** Budget-aware structured pruning using continuous approximations of the Heaviside function, allowing differentiable pruning masks that respect computational budgets.

---

### 3. TALE: Token-Budget-Aware LLM Reasoning (ACL 2025)

| Field | Value |
|-------|-------|
| **Title** | Token-Budget-Aware LLM Reasoning |
| **URL** | [https://arxiv.org/abs/2412.18547](https://arxiv.org/abs/2412.18547) |
| **Venue** | ACL 2025 Findings (pages 24842–24855) |
| **Authors** | Tingxu Han, Zhenting Wang, Chunrong Fang, Shiyu Zhao, Shiqing Ma, Zhenyu Chen (Nanjing University, Rutgers, UMass Amherst) |
| **Paper PDF** | [https://aclanthology.org/2025.findings-acl.1274.pdf](https://aclanthology.org/2025.findings-acl.1274.pdf) |
| **GitHub** | [https://github.com/GeniusHTX/TALE](https://github.com/GeniusHTX/TALE) |

> *"Token-Budget-Aware LLM Reasoning. Reasoning is crucial for LLMs to perform complex tasks, but methods like Chain-of-Thought (CoT) reasoning often lead to significant token overhead and increased costs."*

**Key Results:** TALE framework reduces CoT token usage by **67%** with **<3% accuracy loss**. Two implementations: TALE-EP (zero-shot estimator) and TALE-PT (post-training with SFT/DPO).

---

### 4. Budget-Aware Agentic Routing (ICLR 2025 under review)

| Field | Value |
|-------|-------|
| **Title** | Budget-Aware Agentic Routing via Boundary-Guided Training |
| **URL** | [https://arxiv.org/abs/2602.21227](https://arxiv.org/abs/2602.21227) |

> *"We propose Budget-Aware Agentic Routing, which selects between a cheap and an expensive model at each step to optimize the cost--success efficiency frontier."*

**Key Methods:** BoSFT (Boundary-Guided Supervised Fine-Tuning) and BoPO (Boundary-Guided Policy Optimization) for routing between small and large models.

---

### 5. Inference-Aware Optimization (arXiv 2025)

| Field | Value |
|-------|-------|
| **Title** | Think Smarter not Harder: Adaptive Reasoning with Inference Aware Optimization |
| **URL** | [https://arxiv.org/abs/2501.17974](https://arxiv.org/abs/2501.17974) |
| **Citations** | 28 |

> *"In this work, we propose a way to allow models to be aware of inference budgets by formulating it as utility maximization with respect to an inference budget."*

---

### 6. Budget-Aware Anytime Reasoning (ACL 2026)

| Field | Value |
|-------|-------|
| **Title** | Budget-Aware Anytime Reasoning with LLM-Synthesized Preference Data |
| **URL** | [https://arxiv.org/abs/2601.11038](https://arxiv.org/abs/2601.11038) |
| **Venue** | ACL 2026 Findings |
| **Authors** | Xuanming Zhang et al. (Columbia University, Oracle AI) |

> *"Even partial solutions can provide immediate utility (e.g., a feasible but incomplete trip plan), while additional computation can further refine them."*

**Key Method:** Preference Data Prompting (PDP) for improving intermediate reasoning quality under token budgets.

---

### 7. BudgetThinker (arXiv 2025)

| Field | Value |
|-------|-------|
| **Title** | BudgetThinker: Empowering Budget-aware LLM Reasoning with Control Tokens |
| **URL** | [https://arxiv.org/abs/2508.17196](https://arxiv.org/abs/2508.17196) |

> *"This paper introduces BudgetThinker, a novel framework designed to empower LLMs with budget-aware reasoning, enabling precise control over the reasoning length."*

**Key Method:** Dynamic control token insertion with two-stage training (SFT + curriculum RL).

---

### 8. BAVT: Budget-Aware Value Tree Search (arXiv 2026)

| Field | Value |
|-------|-------|
| **Title** | Spend Less, Reason Better: Budget-Aware Value Tree Search for LLM Agents |
| **URL** | [https://arxiv.org/abs/2603.12634](https://arxiv.org/abs/2603.12634) |

> *"Budget-Aware Value Tree Search for LLM Agents"*

**Source:** [arXiv HTML](https://arxiv.org/html/2603.12634)

> *"Across all four multi-hop datasets, the proposed BAVT framework consistently outperforms the parallel sampling."*

---

### 9. Budget-Aware Test-time Scaling

| Field | Value |
|-------|-------|
| **Title** | Budget-Aware Test-time Scaling via Discriminative Verification |
| **URL** | [https://huggingface.co/papers/2510.14913](https://huggingface.co/papers/2510.14913) |

> *"A hybrid approach combining discriminative verification with self-consistency outperforms generative verification in test-time scaling for large language models, achieving higher accuracy within a fixed compute budget."*

---

### 10. Budget-Aware Tool-Use (arXiv 2025)

| Field | Value |
|-------|-------|
| **Title** | Budget-Aware Tool-Use Enables Effective Agent Scaling |
| **URL** | [https://arxiv.org/html/2511.17006v1](https://arxiv.org/html/2511.17006v1) |

> *"We provide the first systematic study of budget-constrained tool-use agents by formalizing agent test-time scaling with explicit tool-call budgets."*

---

## Repositories & Tools

| Resource | URL | Description |
|----------|-----|-------------|
| **ChipNet** | [github.com/transmuteAI/ChipNet](https://github.com/transmuteAI/ChipNet) | Budget-aware structured pruning with Heaviside continuous approximations (MIT) |
| **TALE** | [github.com/GeniusHTX/TALE](https://github.com/GeniusHTX/TALE) | Token-budget-aware LLM reasoning (ACL 2025) |
| **Budget RNNs** | [github.com/tejaskannan/budget-rnn](https://github.com/tejaskannan/budget-rnn) | RNN inference under energy budgets on low-power sensors |
| **BudgetMem** | [github.com/ViktorAxelsen/BudgetMem](https://github.com/ViktorAxelsen/BudgetMem) | Query-aware budget-tier memory computation |
| **Budget-Aware Adapters** | [github.com/rodrigoberriel/budget-aware-adapters](https://github.com/rodrigoberriel/budget-aware-adapters) | ICCV 2019 — Domain-specific models with adjustable parameter budgets |
| **Structured Pruning + Budget-Aware Reg** | [github.com/A-LinCui/Structured-Pruning-of-Neural-Networks-with-Budget-Aware-Regularization](https://github.com/A-LinCui/Structured-Pruning-of-Neural-Networkswith-Budget-Aware-Regularization) | Pretrain, prune, finetune ResNet on CIFAR-10 |
| **OptiLLM** | [github.com/algorithmicsuperintelligence/optillm](https://github.com/algorithmicsuperintelligence/optillm) | OpenAI API-compatible proxy with 20+ techniques for LLM accuracy and efficiency |

---

## Benchmarks

| Benchmark | Domain | Description |
|-----------|--------|-------------|
| **NaturalPlan (Trip)** | Structured planning | Constraint Satisfaction Rate (CSR) metric |
| **AIME 2024** | Math reasoning | Multi-step competition math problems |
| **GPQA-Diamond** | Scientific QA | Expert-level question answering |
| **GSM8K** | Math reasoning | Grade school math problems |
| **MathBench** | Math reasoning | Arithmetic through college-level |
| **CIFAR-10/100** | Image classification | Standard vision benchmark |
| **ImageNet** | Image classification | Large-scale visual recognition |
| **COCO Detection/Segmentation** | Object detection | Detection and segmentation tasks |
| **Cityscapes** | Semantic segmentation | Urban scene understanding |
| **Kinetics** | Video classification | Human action recognition |
| **SciWorld** | Scientific reasoning | Multi-step scientific tasks |
| **ALFWorld** | Embodied instruction | Instruction following |
| **AppWorld** | Interactive coding | 457 API functions |

---

## Use Cases

### 1. Training-Time Budget Allocation
- **Problem:** Training on limited compute budgets (e.g., few epochs, constrained GPU hours)
- **Solution:** Linear learning rate schedule that decays to zero by end of budget
- **Result:** +50% improvement over step decay at 1% budget on ImageNet
- **Reference:** [Li et al., ICLR 2020](https://arxiv.org/abs/1905.04753)

### 2. Token-Efficient LLM Reasoning
- **Problem:** Chain-of-Thought reasoning wastes tokens on redundant steps
- **Solution:** TALE framework dynamically allocates token budgets based on problem complexity
- **Result:** 67% token reduction, 59% cost reduction, <3% accuracy loss
- **Reference:** [Han et al., ACL 2025](https://github.com/GeniusHTX/TALE)

### 3. Agentic Model Routing
- **Problem:** Allocating expensive model calls in multi-step agent workflows
- **Solution:** Boundary-guided training (BoSFT + BoPO) for cost-success efficiency
- **Result:** >63% success at $0.125 vs $0.35 for always-large baseline
- **Reference:** [Anonymous, ICLR 2025](https://arxiv.org/abs/2602.21227)

### 4. Structured Model Pruning
- **Problem:** Compressing neural networks while respecting computational budgets
- **Solution:** ChipNet with differentiable Heaviside approximations for budget-aware masks
- **Reference:** [Tiwari et al., ICLR 2021](https://github.com/transmuteAI/ChipNet)

### 5. Anytime Reasoning Systems
- **Problem:** Producing useful partial solutions within strict token/latency budgets
- **Solution:** Preference Data Prompting (PDP) for inference-time self-improvement
- **Result:** Improves both intermediate and final solution quality
- **Reference:** [Zhang et al., ACL 2026](https://arxiv.org/abs/2601.11038)

### 6. KV-Cache Budget Optimization
- **Problem:** Memory constraints in LLM inference
- **Solution:** SqueezeAttention with layer-wise and sequence-wise budget allocation
- **Result:** 30–70% memory reduction, up to 2.2× throughput improvement
- **Reference:** [OpenReview 2025](https://openreview.net/forum?id=9HK2rHNAhd)

### 7. Tool-Use Agent Scaling
- **Problem:** Efficient tool-call budget allocation for agents
- **Solution:** Budget-forcing strategy with explicit tool budgets
- **Result:** 24.6% on BrowseComp vs baseline
- **Reference:** [arXiv 2511.17006](https://arxiv.org/html/2511.17006v1)

---

## Summary Table

| Paper / Resource | Venue | Year | Focus | URL |
|------------------|-------|------|-------|-----|
| Budgeted Training | ICLR | 2020 | Training schedule | [arXiv](https://arxiv.org/abs/1905.04753) |
| ChipNet | ICLR | 2021 | Pruning | [GitHub](https://github.com/transmuteAI/ChipNet) |
| TALE | ACL | 2025 | Token budgets | [GitHub](https://github.com/GeniusHTX/TALE) |
| Budget-Aware Agentic Routing | arXiv | 2025 | Model routing | [arXiv](https://arxiv.org/abs/2602.21227) |
| Inference-Aware Optimization | arXiv | 2025 | Budget formulation | [arXiv](https://arxiv.org/abs/2501.17974) |
| Budget-Aware Anytime Reasoning | ACL | 2026 | Anytime reasoning | [arXiv](https://arxiv.org/abs/2601.11038) |
| BudgetThinker | arXiv | 2025 | Control tokens | [arXiv](https://arxiv.org/abs/2508.17196) |
| BAVT | arXiv | 2026 | Value tree search | [arXiv](https://arxiv.org/abs/2603.12634) |
| Budget-Aware Feature Selection | RL Journal | 2025 | RL feature selection | [PDF](https://openreview.net/pdf/d4acb51043380b9161f2877dfd90cf322c2b67c7.pdf) |
| Budget-Aware Dynamic Inference | BMVC | 2024 | Spatially adaptive | [PDF](https://bmva-archive.org.uk/bmvc/2024/papers/Paper_853/paper.pdf) |
| Budget-Aware Test-time Scaling | HF Papers | 2025 | Discriminative verification | [HF](https://huggingface.co/papers/2510.14913) |
| SqueezeAttention | OpenReview | 2025 | KV-cache | [OpenReview](https://openreview.net/forum?id=9HK2rHNAhd) |
| BECOME | ACM | 2025 | Model extraction | [ACM](https://dl.acm.org/doi/abs/10.1145/3746252.3761032) |
| Budget-Aware Tool-Use | arXiv | 2025 | Agent scaling | [arXiv](https://arxiv.org/html/2511.17006v1) |
| EvolCAF | arXiv | 2024 | Bayesian optimization | [arXiv](https://arxiv.org/abs/2404.16906) |

---

## Notes

- The term "refinement" in this context typically refers to **post-training refinement under budget constraints** rather than general model refinement
- Most recent work (2024–2026) focuses on **LLM reasoning** and **agentic systems** where token/compute budgets are critical
- Earlier work (2019–2022) focused more on **vision models** and **structured pruning**
- Budget awareness can be introduced at **training time** (schedule design) or **inference time** (runtime adaptation)

---

## Related Topics

- [AutoAdapt Framework](./AutoAdapt-Framework.md) — Includes AutoRefine (budget-aware HPO) as a core component
- [LoRA / QLoRA](../Parameter-Efficient-Fine-Tuning/LoRA-QLoRA.md) — Parameter-efficient methods with budget considerations
- [Test-time Scaling](../Test-time-Scaling/Test-time-Scaling.md) — Allocating compute at inference time
- [Model Routing](../Inference-Efficiency/Model-Routing.md) — Dynamic selection between models of different sizes
