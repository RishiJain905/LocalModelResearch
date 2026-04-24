# Verifiable-trajectories

## 📌 Overview

Techniques for creating verifiable, reward-signal-rich training data — where outputs can be automatically checked for correctness. Covers RL with verifiable rewards (RLVR), code execution verification, and curated math benchmark datasets.

---

## Subtopics

| File | Topic | Type |
|------|-------|------|
| [RLVR.md](./RLVR.md) | Reinforcement Learning with Verifiable Rewards | Training methodology |
| [Code-Verification.md](./Code-Verification.md) | Code-Verification | Execution-based evaluation |
| [Math-gold-standards.md](./Math-gold-standards.md) | Math-Gold-Standards | Benchmark datasets |

---

## Summary Matrix

| Topic | Key Idea | Core Advantage |
|-------|---------|---------------|
| **RLVR** | Binary verifiable rewards (math, code) for RL training | No human labels needed |
| **Code-Verification** | Execute generated code against unit tests | Unambiguous pass/fail signals |
| **Math-Gold-Standards** | Curated verified math datasets | Objective ground-truth for RL |

---

## Key Findings

- **RLVR debate:** Tsinghua (Limit of RLVR, NeurIPS 2025 Best Paper Runner-Up) argues RLVR only improves sampling efficiency, not reasoning capacity — 0% problems solved exclusively by RLVR. Counter: Two-stage theory suggests prolonged training can expand capability boundaries.
- **Code verification:** BigCode project dominates — Evaluation Harness, StarCoder, OctoPack. Self-verification (ReVeal, ICLR 2025) uses iterative generation-verification loops.
- **Math gold standards:** Process supervision (PRM800K) significantly outperforms outcome-only (78.2% vs 72.4% on MATH).

---

## Related Topics

- [datasets-curation](../README.md) — Parent folder overview
