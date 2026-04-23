# RLVR Research Report
# Reinforcement Learning with Verifiable Rewards

Repository: github.com/RishiJain905/LocalModelResearch
Output File: topics/datasets-curation/Verifiable-trajectories/RLVR-Research.md
Date: April 23, 2026

---

## 1. Definitions & Background

### What is RLVR?

Definition: Reinforcement Learning with Verifiable Rewards (RLVR) is a training strategy where language models are improved using RL with rewards that can be automatically verified, rather than subjective human preference labels.

Source: Label Studio blog - URL: https://labelstud.io/blog/reinforcement-learning-from-verifiable-rewards/

Direct Quote: Reinforcement Learning with Verifiable Rewards is among the leading training strategies for injecting learning signals into LLMs, successfully employed by models such as DeepSeek R1 and TulU 3.

Core Definition: At its core, Verifiable Rewards are simple functions that provide a clear-cut, binary ground truth signal - typically a 1 (correct) or 0 (incorrect) - to indicate whether a model output meets a predefined correctness criterion.

---

## 2. Key Sources

### Source 1: DeepSeek-R1 Paper (Nature 2025)

Title: DeepSeek-R1: Incentivizing Reasoning Capability in LLMs via Reinforcement Learning
URL: https://www.nature.com/articles/s41586-025-09422-z
DOI: 10.1038/s41586-025-09422-z
Published: 17 September 2025
Authors: Daya Guo, Dejian Yang, Haowei Zhang, et al. (DeepSeek-AI Team)

Key Findings:
- Pure RL (no SFT) achieved 71 percent on AIME 2024 (R1-Zero), climbing from 15.6 percent baseline
- DeepSeek-R1 achieved 79.8 percent on AIME 2024, 97.3 percent on MATH-500
- GRPO algorithm with rule-based rewards only

Direct Quote: The reasoning abilities of large language models (LLMs) can be incentivized through pure reinforcement learning (RL), eliminating the need for human-labeled reasoning trajectories.

---

## 3. Summary

Key papers collected with direct quotes and URLs.


---

### Source 2: Limit of RLVR (Tsinghua/SJTU - NeurIPS 2025 Best Paper Runner-Up)

Title: Limit of RLVR: Does Reinforcement Learning Really Incentivize Reasoning Capacity in LLMs Beyond the Base Model?
URL: https://limit-of-rlvr.github.io/
arXiv: https://arxiv.org/abs/2504.13837
Code: https://github.com/LeapLabTHU/limit-of-RLVR

Authors: Yang Yue, Zhiqi Chen, Rui Lu, Andrew Zhao, Zhaokai Wang, Shiji Song, Gao Huang (Tsinghua University), PLUS Yang Yue (Shanghai Jiao Tong University)

Awards: NeurIPS 2025 Best Paper Runner-Up | ICML 2025 Workshop AI4Math Best Paper

Core Thesis:
RL fine-tuning enhances sampling efficiency without expanding the reasoning capacity already present in base models.

Key Findings:
- All correct solutions from RL-trained models already exist in the base model distribution
- RLVR enhances **sampling efficiency**, not reasoning capacity
- RL trained models perform WORSE than base models at high pass@k
- 0 percent problems solved exclusively by RLVR

Direct Quote: If RL training truly expands reasoning, we would expect the RL model to solve more such problems than the base. However, we observe the opposite.

Direct Quote 2: Crucially, all correct solutions from RL-trained models already exist in the base model distribution, proving RLVR enhances sampling efficiency, not reasoning capacity, while inadvertently shrinking the solution space.

---

### Source 3: GRPO Paper (DeepSeekMath)

Title: Reinforcement Learning with Verifiable Rewards: GRPO Effective Loss, Dynamics, and Success Amplification
URL: https://arxiv.org/abs/2503.06639
arXiv ID: 2503.06639

Key Quote: Group Relative Policy Optimization (GRPO) was introduced and used recently for promoting reasoning in LLMs under verifiable (binary) rewards.

---

### Source 4: TulU 3 (AI2)

URL: https://github.com/orgs/allenai/packages/container/package/open-instruct

AI2 used RLVR in TulU 3 to surpass DeepSeek V3 and GPT-4o on key benchmarks.

---

### Source 5: The Debate on RLVR Reasoning Capability Boundary (arXiv 2510.04028)

Title: The Debate on RLVR Reasoning Capability Boundary: Shrinkage, Expansion, or Both? A Two-Stage Dynamic View
URL: https://arxiv.org/html/2510.04028v1

Authors: (To verify from paper)

Core Thesis: Resolves the RLVR shrinkage vs. expansion debate through a two-stage dynamic view:
1. Stage 1 (Exploitation): Early training narrows capability boundary
2. Stage 2 (Exploration): Prolonged training can expand boundary

Direct Quote: Under the Pass at k metric, raising the probability of at least one correct action above 1/k corresponds to an expansion of the reasoning capability boundary.

Direct Quote 2: Premature termination of training explains why some studies observe only shrinkage.

---

## 3. Methodology

### How RLVR Works

1. **Group Sampling:** Generate multiple candidate responses for each prompt
2. **Scoring:** Apply rule-based reward functions (math match, code execution)
3. **GRPO Update:** Increase probability of above-average responses, decrease below-average

Source: DataCamp
URL: https://www.datacamp.com/blog/what-is-grpo-group-relative-policy-optimization

Direct Quote: GRPO fine-tunes the model directly based on the output of the reward functions, without the need to collect preference data.

---

### Verifiable Reward Types

**1. Mathematical Correctness:**
Binary 1 (correct) or 0 (incorrect) based on string match against ground truth.

**2. Code Execution:**
- +1: all tests pass
- -1: any test fails
- -0.2: no valid code generated

**3. Format/Instruction Rewards:**
String matching for keywords, structured output templates.

---

### Connection to Process Reward Models (PRM)

PRM = Process Reward Model, assigns scores to individual reasoning steps rather than just final outcomes.

Source: Michael Brenndoerfer blog
URL: https://mbrenndoerfer.com/writing/process-reward-models-prm-outcome-math-training-limitations

Direct Quote: A process reward model (PRM) is a model that assigns a score to individual steps in a reasoning chain, rather than evaluating only the final outcome.

Key Distinction:
- RLVR (ORM): Outcome-based only (final answer correct/incorrect)
- PRM: Step-level evaluation enabling better credit assignment

Source: Stop Summation paper (NeurIPS 2025)
URL: https://openreview.net/forum?id=3Sxby0hH1q

Direct Quote: We find the primary cause of reward hacking induced by PRM is that: the canonical summation-form credit assignment in reinforcement learning (RL), i.e. cumulative gamma-decayed future rewards, causes the LLM to hack steps with high rewards.

Solution proposed: Min-form credit assignment (PURE approach) - defines value function as minimum future rewards rather than sum.

---


## 4. Benchmark Results

| Benchmark | Score | Source |
|-----------|-------|--------|
| AIME 2024 Pass at 1 (R1-Zero) | 71 percent (from 15.6 percent baseline) | Nature |
| AIME 2024 Pass at 1 (R1) | 79.8 percent | Nature |
| MATH-500 (R1) | 97.3 percent | Galileo AI |
| ARC-AGI-1 (R1-Zero) | 14 percent | ARC Prize |
| ARC-AGI-1 (R1) | 15.8 percent | ARC Prize |

---

## 5. Debates & Controversies

### Sampling Efficiency vs. True Reasoning

**Tsinghua Position:** RLVR only improves sampling efficiency. All correct solutions already exist in base model. RL does not create new reasoning.

**Direct Quote:** Crucially, all correct solutions from RL-trained models already exist in the base model distribution, proving RLVR enhances sampling efficiency, not reasoning capacity.

**Counterargument:** Production models (o1, o3) show genuine capability improvements at scale.

### The 0 percent Claim

Tsinghua paper: 0 percent problems solved exclusively by RLVR - every problem RL model can solve, base model could also solve with enough samples.

### NeurIPS 2025 Recognition

Limit of RLVR received perfect score (6,6,6,6) at NeurIPS 2025, named Best Paper Runner-Up.

URL: https://x.com/jiqizhixin/status/1987710546674856051

### Two-Stage Dynamic View

A response paper argues the debate is premature:
- Stage 1 (Exploitation): Early training narrows boundary (over-reinforcing high-probability solutions)
- Stage 2 (Exploration): Prolonged training can expand boundary (redirecting to low-probability high-potential tokens)

Key Quote: Premature termination of training explains why some studies observe only shrinkage.

---

## 6. Summary Table / Data Points

| Metric | Value | Source |
|--------|-------|--------|
| DeepSeek-R1-Zero AIME 2024 | 71 percent (from 15.6 percent baseline) | Nature 2025 |
| DeepSeek-R1 AIME 2024 | 79.8 percent | Nature 2025 |
| DeepSeek-R1 MATH-500 | 97.3 percent | Galileo AI |
| RLVR Core Claim | Binary (0/1) verifiable rewards | Label Studio |
| GRPO | No labeled data required | DataCamp |
| Limit of RLVR Core | 0 percent problems solved exclusively by RLVR | Tsinghua arXiv |
| NeurIPS 2025 | Best Paper Runner-Up (perfect 6,6,6,6) | NeurIPS awards |

---

## Key Papers Table

| Paper | URL | Date |
|-------|-----|------|
| DeepSeek-R1 (Nature) | https://www.nature.com/articles/s41586-025-09422-z | Sep 2025 |
| Limit of RLVR | https://arxiv.org/abs/2504.13837 | Apr 2025 |
| GRPO | https://arxiv.org/abs/2503.06639 | Mar 2025 |
| RLVR Debate (Two-Stage) | https://arxiv.org/html/2510.04028v1 | Oct 2025 |
| Stop Summation (PURE) | https://openreview.net/forum?id=3Sxby0hH1q | NeurIPS 2025 |

---

## Bibliography

1. https://labelstud.io/blog/reinforcement-learning-from-verifiable-rewards/
2. https://www.datacamp.com/blog/what-is-grpo-group-relative-policy-optimization
3. https://www.nature.com/articles/s41586-025-09422-z
4. https://arxiv.org/abs/2501.12948
5. https://limit-of-rlvr.github.io/
6. https://arxiv.org/abs/2504.13837
7. https://github.com/LeapLabTHU/limit-of-RLVR
8. https://arxiv.org/abs/2503.06639
9. https://arxiv.org/html/2510.04028v1
10. https://arcprize.org/blog/r1-zero-r1-results-analysis
11. https://galileo.ai/blog/deepseek-reinforcement-learning
12. https://mbrenndoerfer.com/writing/process-reward-models-prm-outcome-math-training-limitations
13. https://fireworks.ai/blog/reinforcement-learning-with-verifiable-reward
14. https://github.com/Mohammadjafari80/GSM8K-RLVR
15. https://www.lesswrong.com/posts/s3NaETDujoxj4GbEm/tsinghua-paper-does-rl-really-incentivize-reasoning-capacity
16. https://x.com/jiqizhixin/status/1987710546674856051
17. https://openreview.net/forum?id=3Sxby0hH1q
