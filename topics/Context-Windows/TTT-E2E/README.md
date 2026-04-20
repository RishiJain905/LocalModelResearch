# TTT-E2E: Test-Time Training — End-to-End

> **Papers:** [arXiv:2512.23675](https://arxiv.org/abs/2512.23675) · [arXiv:2601.00894](https://arxiv.org/abs/2601.00894)  
> **GitHub:** [test-time-training/e2e](https://github.com/test-time-training/e2e) (592 stars, JAX) · [deveworld/ponderTTT](https://github.com/deveworld/ponderTTT) · [DuNGEOnmassster/efficient-ttt-pytorch](https://github.com/DuNGEOnmassster/efficient-ttt-pytorch)

---

## Overview

Test-Time Training (TTT) is a family of techniques where a model continues learning/adaptating during inference rather than using fixed weights. TTT-E2E specifically refers to approaches that are **End-to-End** — both training-time (via meta-learning) and test-time (via next-token prediction).

This folder covers:
- **TTT-E2E (Stanford)** — Main arXiv paper, official JAX repo
- **PonderTTT** — Adaptive compute allocation via TTT gating
- **TTT-Layers** — Layer-level TTT implementations
- **Memory-as-Weights** — Context compressed into model weights

---

## Key Papers

| Paper | Year | Venue | URL | Key Contribution |
|-------|------|-------|-----|-----------------|
| **End-to-End Test-Time Training for Long Context** | 2025 | arXiv | [2512.23675](https://arxiv.org/abs/2512.23675) | TTT-E2E with meta-learning, 2.7x faster at 128K context |
| **When to Ponder: Adaptive Compute Allocation for Code Generation via Test-Time Training** | 2025 | arXiv | [2601.00894](https://arxiv.org/abs/2601.00894) | PonderTTT — training-free gating using TTT reconstruction loss |

---

## Navigation

- [TTT-E2E (Stanford).md](./TTT-E2E%20(Stanford).md)
- [TTT-Layers.md](./TTT-Layers.md)
- [Memory-as-Weights.md](./Memory-as-Weights.md)

---

*Last updated: 2026-04-20*
