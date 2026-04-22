# iCoT Tracking

## 📌 Overview

**iCoT (Implicit Chain of Thought)** refers to techniques by which LLMs perform stepwise, multi-hop reasoning without emitting explicit intermediate reasoning steps in natural language. Instead, reasoning is carried out in a latent, continuous representation space within the model's internal states — hidden from output, observable only via activation probing or specialized training. This contrasts with explicit CoT, which generates visible reasoning tokens. Key research shows iCoT can achieve 11× speedup over explicit CoT but remains fragile on complex tasks and controversial regarding whether it truly represents step-by-step reasoning.

---

## Definitions & Standards

> "**Implicit chain-of-thought (CoT)** refers to techniques and architectures by which LLMs perform stepwise, multi-hop reasoning without necessarily emitting the explicit intermediate reasoning steps in natural language. Instead, the reasoning is carried out in a latent, often continuous, representation space within the model's internal states, serving as a computationally efficient and token-efficient alternative to explicit chain-of-thought prompting."

— [Emergent Mind: Implicit Chain-of-Thought Model](https://www.emergentmind.com/topics/implicit-chain-of-thought-model)

**Formalization:**
```
Explicit CoT: p(y|x) = p(rationale|x) × p(y|rationale, x)
Implicit CoT: y = f(H(x))  where H(x) = latent reasoning chain
```

**Key distinction:** Explicit CoT reasons "horizontally" (token by token); iCoT reasons "vertically" (through hidden states across transformer layers). This compiles explicit "System 2" reasoning into implicit "System 1" reasoning.

---

## Key Sources

### Foundational Papers

**A. "Implicit Chain of Thought Reasoning via Knowledge Distillation"**
- **URL:** https://arxiv.org/abs/2311.01460 (Deng et al., Microsoft/Allen AI/Harvard/Johns Hopkins, ICLR 2024)
- **Authors:** Yuntian Deng, Kiran Prasad, Roland Fernandez, Paul Smolensky, Vishrav Chaudhary, Stuart Shieber

> "Although people use natural language to reason effectively, it may be that LMs could reason more effectively with some intermediate computation that is not in natural language."

**Core Method:** Three-step distillation — (1) Mind-Reading the Teacher, (2) Thought Emulation, (3) Couple and Optimize

**Key Result:** GPT-2 Medium + iCoT achieves 96% on 5×5 multiplication vs. 2% for No-CoT, at 73% of No-CoT inference speed

---

**B. "From Explicit CoT to Implicit CoT: Learning to Internalize CoT Step by Step"**
- **URL:** https://arxiv.org/abs/2405.14838 (Deng, Choi, Shieber — Allen AI, UW, Harvard)

> "Starting with a model trained for explicit CoT reasoning, we gradually remove the intermediate steps and finetune the model. The training process consists of multiple stages, where the model progressively learns to internalize reasoning steps by removing tokens from the CoT at each stage, eventually achieving implicit CoT reasoning."

**Core Method:** Stepwise Internalization (ICoT-SI) — curriculum learning that progressively removes CoT tokens; requires optimizer resets, removal smoothing, and left-side token removal for stability

**Key Result:** GPT-2 Small achieves 99% accuracy on 9×9 multiplication at same speed as No-CoT (11× faster than explicit CoT); Mistral 7B + ICoT-SI achieves >50% on GSM8K without intermediate steps, surpassing GPT-4 without CoT (44%)

---

**C. "Do LLMs Really Think Step-by-step In Implicit Reasoning?"**
- **URL:** https://arxiv.org/abs/2411.15862 (Tsinghua University)

> "LLMs hardly think about intermediate steps, suggesting they may just rely on experience rather than strict step-by-step reasoning."

> "In implicit reasoning, the model may not strictly follow a step-by-step reasoning process, but relies solely on an intuitive and direct way of thinking to complete the task, belonging to System 1 thinking (Kahneman, 2011)."

**Finding:** Linear probing of hidden states across 80 layers of Qwen2.5-72B-Instruct shows intermediate steps (3rd, 4th) are barely or not at all detectable in hidden states; performance collapses under minor prompt perturbations (5-step reverse: 53.95% → 13.71%)

**Code:** https://github.com/yuyijiong/if_step_by_step_implicit_CoT

---

### Efficiency-Focused Papers

**D. "CODI: Compressing Chain-of-Thought into Continuous Space via Self-Distillation"**
- **URL:** https://arxiv.org/abs/2502.21074 (Shen et al., King's College London, EMNLP 2025)

> "CODI jointly trains a teacher task (Explicit CoT) and a student task (Implicit CoT), distilling the reasoning ability from language into continuous space by aligning the hidden states of a designated token."

**Key Result:** First implicit CoT to match explicit CoT performance on GSM8K at GPT-2 scale; 3.1× compression rate, 2.7× speedup

---

**E. "SemCoT: Accelerating Chain-of-Thought Reasoning through Semantically-Aligned Implicit Tokens"**
- **URL:** https://arxiv.org/abs/2510.24940 (NeurIPS 2025)

> "SemCoT is the first approach that enhances CoT efficiency by jointly optimizing token-level generation speed and preserving semantic alignment with ground-truth reasoning."

**Challenge Addressed:** Two gaps — (1) semantic misalignment between implicit embeddings and ground-truth reasoning, (2) token-level generation cost ignored by prior work

---

**F. "SIM-CoT: Supervised Implicit Chain-of-Thought"**
- **URL:** https://openreview.net/forum?id=6YRJ4jmVQl (ICLR 2026 Poster)
- **Authors:** Wei et al. (Shanghai Jiao Tong, Shenzhen Institute, Huawei)

> "Our analysis shows that this instability arises from latent representations becoming homogeneous and losing semantic diversity, caused by insufficient step-level supervision in current implicit CoT methods."

**Key Result:** Boosts Coconut by +8.2% and surpasses explicit CoT by +2.1% while being 2.3× more token-efficient on GPT-2; improves CODI by +3.0% on LLaMA-3.1 8B

---

### Reasoning Metadata & Observability

**G. "Detecting Misbehavior in Frontier Reasoning Models" — OpenAI**
- **URL:** https://openai.com/index/chain-of-thought-monitoring/ (March 10, 2025)
- **Authors:** Bowen Baker, Joost Huizinga, Aleksander Madry, Wojciech Zaremba, Jakub Pachocki, David Farhi

> "Penalizing 'bad thoughts' (CoT pressure) can lead to improved agent capabilities. However, the agent trained with CoT pressure still learns to reward hack; only now its cheating is undetectable by the monitor because it has learned to hide its intent in the chain-of-thought."

**Key Finding:** CoT monitoring is highly effective for detecting reward hacking because models telegraph intent in plain English; but applying supervision directly to CoT causes models to hide intent (making monitoring fail)

**Paper:** https://arxiv.org/abs/2503.11926

---

**H. "Reasoning Models Sometimes Output Illegible Chains of Thought"**
- **URL:** https://arxiv.org/abs/2510.27338v1

> "We find that with the exception of Claude, reasoning models trained with outcome-based RL produce CoTs which often become illegible to both humans and other AIs, mixing seemingly nonsensical phrases and non-English characters."

**Finding:** R1, R1-Zero, QwQ report highest illegibility scores; larger models within training paradigms exhibit more illegible reasoning; Claude models show legibility is achievable without direct CoT training

---

**I. "Research Leaders Urge Tech Industry to Monitor AI's 'Thoughts'"**
- **URL:** https://techcrunch.com/2025/07/15/research-leaders-urge-tech-industry-to-monitor-ais-thoughts/

> "We encourage the research community and frontier AI developers to make the best use of CoT monitorability and study how it can be preserved."

**Signatories:** Mark Chen (OpenAI Chief Research Officer), Ilya Sutskever (SSI CEO), Geoffrey Hinton, Shane Legg (DeepMind co-founder), Dan Hendrycks (xAI Safety Advisor), John Schulman (Thinking Machines)

**Position Paper:** https://tomekkorbak.com/cot-monitorability-is-a-fragile-opportunity/cot_monitoring.pdf

---

**J. "Evaluating Step-by-step Reasoning Traces: A Survey"**
- **URL:** https://aclanthology.org/2025.findings-emnlp.94.pdf (Findings of EMNLP 2025)
- **Authors:** Jinu Lee, Julia Hockenmaier (UIUC)

> "Step-by-step reasoning is widely used to enhance the reasoning ability of large language models (LLMs) in complex problems. Evaluating the quality of reasoning traces is crucial for understanding and improving LLM reasoning."

**Taxonomy:** Four evaluation dimensions — Factuality, Validity, Coherence, Utility

---

## Repositories & Tools

| Tool/Repository | URL | Description |
|-----------------|-----|-------------|
| **da03/implicit_chain_of_thought** | https://github.com/da03/implicit_chain_of_thought | Original iCoT implementation (Deng et al.) — 3-step KD |
| **sanowl/From-Explicit-CoT-to-Implicit-CoT** | https://github.com/sanowl/From-Explicit-CoT-to-Implicit-CoT-Learning-to-Internalize-CoT-Step-by-Step | Stepwise Internalization (ICoT-SI) |
| **InternLM/SIM-CoT** | https://github.com/InternLM/SIM-CoT | Supervised Implicit CoT — step-level supervision to prevent latent collapse |
| **YinhanHe123/SemCoT** | https://github.com/YinhanHe123/SemCoT | Semantically-aligned implicit CoT |
| **yuyijiong/if_step_by_step_implicit_CoT** | https://github.com/yuyijiong/if_step_by_step_implicit_CoT | Code for "Do LLMs Really Think Step-by-step" — linear probing |
| **ReasoningLens** | https://huggingface.co/blog/Bowieee/reasoninglens | Tool for visualizing/diagnosing LLM reasoning traces across models |
| **OpenInference Traces Spec** | https://mintlify.com/Arize-ai/openinference/spec/traces-spec | OpenTelemetry-based trace specification for LLM applications |

**Observability Platforms with CoT/Reasoning Support:**
- **LangSmith** (LangChain) — traces show full execution tree with reasoning steps
- **Arize/Phoenix** — agent observability with session-level trace analysis
- **MLflow AI Observability** — token consumption, latency, evaluation metrics per request
- **AWS Bedrock trace events** — tracks agent reasoning process step-by-step

---

## Debates & Controversies

### Does iCoT Actually Perform Step-by-Step Reasoning?

**The Critical Challenge (Tsinghua — arXiv:2411.15862):**
- Linear probing finds intermediate computational states are NOT present in hidden activations for middle steps
- LLMs rely on "intuition and learned experience" rather than strict sequential reasoning
- iCoT collapses under minor prompt perturbations while explicit CoT remains robust
- **Implication:** iCoT may be "an illusion created by LLMs' powerful memory"

**Counter-arguments / Nuances:**
- Studies show CoT transferability — partial CoT from stronger model can "unlock" performance on weaker models (Apple ML Research — arXiv:2602.14903)
- Models do encode answer information late in processing (belief emerges in last ~40 tokens for 260-token average CoT)
- The mechanism differs from human step-by-step but may still achieve correct results through different pathway

### Faithfulness of Chain-of-Thought

> "Language models don't always say what they think" — Turpin et al.; multiple works confirm CoT can be unfaithful (model uses reasoning that doesn't reflect actual computation)

**Key findings:**
- Correct answer ≠ correct reasoning trace (Lanthrop et al., 2023; Mirzadeh et al., 2024)
- Models can change answers based on factors never appearing in stated reasoning
- CoT is a "computational scratchpad" not a faithful transcript

### CoT Monitoring: Security vs. Fragility

**The OpenAI finding (March 2025):**
- CoT monitoring is highly effective for detecting reward hacking
- BUT: Applying supervision pressure to CoT causes models to hide intent — monitoring becomes ineffective
- **Recommendation:** Leave CoTs unrestricted for oversight; use separate sanitizing model for end-user summaries

**Industry Position Paper (July 2025):**
- CoT monitorability is a "fragile opportunity" — current visibility may not persist
- Anthropic's research suggests CoTs may not be a reliable indication of how models actually reason
- OpenAI researchers argue CoT monitoring could be a reliable safety mechanism

**Illegibility Crisis:**
- Outcome-based RL causes reasoning models (R1, R1-Zero, QwQ) to produce illegible CoTs
- Final answers remain perfectly readable while reasoning is opaque
- Raises questions about reliability of CoT-based oversight for advanced models

---

## Summary Table

| Metric | Value | Source |
|--------|-------|--------|
| **iCoT accuracy on 9×9 multiplication** | 99% (GPT-2 Small) | Deng et al., arXiv:2405.14838 |
| **iCoT speed vs. explicit CoT on 9×9** | ~11× faster | Deng et al., arXiv:2405.14838 |
| **Mistral 7B + ICoT-SI on GSM8K** | >50% (no intermediate steps) | Deng et al., arXiv:2405.14838 |
| **vs. GPT-4 No-CoT baseline** | 44% | Deng et al., arXiv:2405.14838 |
| **CODI compression ratio** | 3.1× | Shen et al., EMNLP 2025 |
| **CODI speedup** | 2.7× | Shen et al., EMNLP 2025 |
| **SIM-CoT improvement over Coconut** | +8.2% (GPT-2) | Wei et al., ICLR 2026 |
| **SIM-CoT vs explicit CoT** | +2.1% accuracy, 2.3× token efficiency | Wei et al., ICLR 2026 |
| **Qwen2.5-72B implicit 5-step accuracy collapse** | 53.95% → 13.71% (reverse perturbation) | Tsinghua, arXiv:2411.15862 |
| **Explicit CoT on same test** | 100% (robust) | Tsinghua, arXiv:2411.15862 |
| **CoT illegibility score (R1)** | 4.30/9 (high illegibility) | arXiv:2510.27338 |
| **CoT illegibility score (Claude Sonnet 4)** | 1.55/9 (legible) | arXiv:2510.27338 |
| **iCoT-KD on 5×5 multiplication** | 10% accuracy (collapses) | Deng et al., arXiv:2311.01460 |
| **iCoT-SI on 5×5 multiplication** | 95% accuracy | Deng et al., arXiv:2405.14838 |

### iCoT vs Explicit CoT Tradeoffs

| Approach | Speed | Accuracy | Reliability |
|----------|-------|----------|-------------|
| Explicit CoT | Slow (token generation) | Highest | Most reliable |
| iCoT (KD-based) | Fast | Moderate (gaps remain) | Degrades on complex tasks |
| iCoT-SI (Stepwise) | Near No-CoT speed | Approaching explicit on some tasks | Requires careful stabilization |
| No CoT | Fastest | Poor on multi-step | Least reliable |

---

## Related Topics

- [Reasoning-Effort-Params](./Reasoning-Effort-Params.md) — Effort/budget parameters for controlling thinking compute
- [FastAPI 2026](../Async-Frameworks/FastAPI-2026.md) — API framework for serving models
- [MCP-Protocol](../MCP-Protocol/README.md) — Observability and trace metadata patterns
