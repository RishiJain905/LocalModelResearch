# Automated Adaption

> Research documentation on automated adaptation strategies for LLMs and neural networks — covering frameworks that automate domain adaptation, budget-aware optimization, and adaptive hyperparameter tuning.

---

## Subtopics

| File | Description |
|------|-------------|
| [AutoAdapt-Framework.md](./AutoAdapt-Framework.md) | Microsoft Research's end-to-end automated framework for LLM domain adaptation — ACG, Planning Agent, AutoRefine. 25% accuracy improvement over SOTA AutoML baselines. |
| [Budget-Aware-Refinement.md](./Budget-Aware-Refinement.md) | Algorithmic strategies that incorporate resource constraints (compute, token, inference budget) into model optimization. Covers 15+ papers from ICLR 2020 to ACL 2026. |

---

## Overview Comparison

| Aspect | AutoAdapt Framework | Budget-Aware Refinement |
|--------|---------------------|------------------------|
| **Core Focus** | End-to-end automated LLM domain adaptation | Resource-constrained model optimization |
| **Primary Institution** | Microsoft Research | Multiple (CMU, Nanjing U, Columbia, etc.) |
| **Key Innovation** | ACG + Planning Agent + AutoRefine | Token/compute budget allocation |
| **Main Use Case** | Domain-specific LLM adaptation (cloud, legal, medical) | Training/inference under budget constraints |
| **Key Result** | 25% accuracy improvement, ~$4, ~30 min overhead | Up to 67% token reduction, 50% training improvement |
| **Year** | 2026 | 2020–2026 |
| **License** | MIT | Various (MIT, Apache, etc.) |

---

## Key Papers

| Paper | Venue | URL |
|-------|-------|-----|
| AutoAdapt: An Automated Domain Adaptation Framework for LLMs | arXiv 2026 | [arXiv:2603.08181](https://arxiv.org/abs/2603.08181) |
| Budgeted Training: Rethinking Deep Neural Network Training Under Resource Constraints | ICLR 2020 | [arXiv:1905.04753](https://arxiv.org/abs/1905.04753) |
| ChipNet: Budget-Aware Pruning with Heaviside Continuous Approximations | ICLR 2021 | [GitHub](https://github.com/transmuteAI/ChipNet) |
| Token-Budget-Aware LLM Reasoning (TALE) | ACL 2025 | [GitHub](https://github.com/GeniusHTX/TALE) |
| Budget-Aware Agentic Routing via Boundary-Guided Training | arXiv 2025 | [arXiv:2602.21227](https://arxiv.org/abs/2602.21227) |
| Budget-Aware Anytime Reasoning with LLM-Synthesized Preference Data | ACL 2026 | [arXiv:2601.11038](https://arxiv.org/abs/2601.11038) |
| BudgetThinker: Empowering Budget-aware LLM Reasoning with Control Tokens | arXiv 2025 | [arXiv:2508.17196](https://arxiv.org/abs/2508.17196) |
| Spend Less, Reason Better: Budget-Aware Value Tree Search for LLM Agents | arXiv 2026 | [arXiv:2603.12634](https://arxiv.org/abs/2603.12634) |

---

## Related Topics

- [Parameter-Efficient Fine-Tuning (LoRA/QLoRA)](../Parameter-Efficient-Fine-Tuning/LoRA-QLoRA.md) — PEFT methods often used in automated adaptation pipelines
- [RAG](../RAG/RAG-Systems.md) — Retrieval-Augmented Generation, one of the strategies AutoAdapt selects from
- [Test-time Scaling](../Test-time-Scaling/Test-time-Scaling.md) — Inference-time compute allocation
- [AutoML](../AutoML/AutoML-Systems.md) — Automated machine learning pipelines

---

*Last updated: 2026-04-26*
