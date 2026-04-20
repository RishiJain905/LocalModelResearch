# P-EAGLE: The Shift — From Sequential to Parallel Drafting

> **Source:** [arXiv:2602.01469](https://arxiv.org/abs/2602.01469) · [AWS Blog](https://aws.amazon.com/blogs/machine-learning/p-eagle-faster-llm-inference-with-parallel-speculative-decoding-in-vllm/)  
> **Subtopic of:** [P-EAGLE](./README.md)

---

## The Core Problem: Autoregressive Drafting Bottleneck

Standard EAGLE drafts tokens **sequentially** — each draft token depends on the previous one, requiring K forward passes for K speculated tokens:

```
Target generates token t_i
→ Draft model generates draft token d_1 (1 forward pass)
→ Draft model generates draft token d_2 (1 forward pass, depends on d_1)
→ Draft model generates draft token d_3 (1 forward pass, depends on d_2)
...
→ Draft model generates draft token d_K (K sequential forward passes)
→ Target model verifies all K tokens in a single batched forward pass
```

> *"EAGLE achieves 2–3× speedups over standard autoregressive decoding and is widely deployed (vLLM, SGLang, TensorRT-LLM). However, to produce K draft tokens, it requires **K sequential forward passes** through the draft model. As drafter models improve at long outputs, this overhead becomes significant—drafter latency scales **linearly with speculation depth**, constraining how aggressively you can speculate."*  
> — [AWS Blog](https://aws.amazon.com/blogs/machine-learning/p-eagle-faster-llm-inference-with-parallel-speculative-decoding-in-vllm/)

**Consequence:** EAGLE-3 peaks at **K=3** — beyond this, drafter overhead dominates and speedup decreases.

---

## P-EAGLE's Solution: Parallel Draft Generation

P-EAGLE generates **all K draft tokens in a single forward pass**:

```
Target generates token t_i
→ P-EAGLE drafter generates ALL K draft tokens (1 forward pass)
→ Target model verifies all K tokens in a single batched forward pass
```

### Why It Works: Two Learnable Placeholders

At MTP (Multi-Token Prediction) positions 2–K, the previous hidden state is unknown because tokens haven't been generated yet. P-EAGLE introduces two trainable parameters to substitute for missing information:

| Parameter | Shape | Role |
|---|---|---|
| **`h_shared`** | `[1, 1, 3×hidden_size]` | Shared hidden state for all MTP positions — substitutes for missing hidden vectors |
| **`mask_token_embedding`** | Standard embedding dim | Substitutes for unknown previous tokens |

### Parallel Input Construction

| Position Type | Input | Notes |
|---|---|---|
| Prompt positions | `emb(p)` + `h_prompt` (shifted by one) | Same convention as autoregressive EAGLE |
| Position 1 (NTP) | `emb(new)` + `h_context` | Identical to standard autoregressive EAGLE |
| Positions 2–K (MTP) | `emb(mask)` + `h_shared` | Learnable parameters as neutral placeholders |

All positions pass through **N transformer layers** (P-EAGLE uses **4 layers** vs. EAGLE-3's 1 layer), then through the LM head to predict draft tokens `t₁...t_K` in a **single forward pass**.

---

## Why 4 Layers Instead of 1?

From ablation studies on LLaMA 3.1 8B target (K=5):

| Layers | HumanEval | MT-Bench | Δ% |
|---|---|---|---|
| 1 | 2.69 | 2.41 | — |
| 2 | 3.58 | 2.76 | +33.1% / +14.5% |
| **4** | **3.92** | **3.04** | **+45.7% / +26.1%** |

> *"Using 4 transformer layers in P-EAGLE (vs EAGLE-3's 1 layer) yields +46% acceptance length improvement. Beyond 4 layers, returns diminish."*  
> — [arXiv:2602.01469](https://arxiv.org/abs/2602.01469)

---

## Theoretical Justification: Attention Injectivity

**Theorem B.3 (from paper):** Attention score function is injective in relative position for almost all query-key pairs.

> *"This means explicit depth encoding or position augmentation is representationally redundant—RoPE already encodes absolute position from which prediction depth is computable."*  
> — [arXiv:2602.01469](https://arxiv.org/abs/2602.01469)

**Empirical confirmation:**
> *"The regularized variant's learnable α decays from 0.1 → 0.029 (71% decrease), confirming the model learns to minimize context injection."*  
> — [arXiv:2602.01469](https://arxiv.org/abs/2602.01469)

**Implication:** Explicitly injecting context that attention already provides is harmful. The learnable shared hidden state works because attention already encodes sufficient positional information.

---

## Hidden State Design Ablation (GPT-OSS 20B, K=5)

| Strategy | Acceptance Length | Δ% |
|---|---|---|
| **Baseline (learnable shared)** | **3.16** | — |
| + depth-specific encoding | 2.85 | −9.8% |
| + NTP hidden + depth encoding | 2.68 | −15.2% |
| + NTP hidden only | 2.81 | −11.1% |
| + regularized NTP hidden | 2.94 | −7.0% |

> *Adding explicit depth or context encoding **hurts** performance — confirms the injectivity theorem.*  
> — [arXiv:2602.01469](https://arxiv.org/abs/2602.01469)

---

## P-EAGLE vs Autoregressive EAGLE-3

| Aspect | Autoregressive EAGLE-3 | P-EAGLE |
|---|---|---|
| Forward passes for K tokens | K sequential | **1 parallel** |
| Model depth | 1 transformer layer | **4 transformer layers** |
| Hidden state at MTP positions | Computed sequentially | Shared learnable `h_shared` |
| Optimal K | 3 (beyond this overhead dominates) | **7** |
| Speedup ceiling | Limited by sequential overhead | **1.55×–1.69× higher** |

---

## Comparison with Related Methods

### PARD (ICLR 2026, arXiv:2504.18583)

| Aspect | PARD | P-EAGLE |
|---|---|---|
| Draft model | Target-independent (one model for all targets) | Target-dependent (specific draft model per target) |
| Training | COD sampling | Amortized mask construction |
| Reported speedup | Up to 4.08× | 1.10×–1.69× |
| Context scaling | OOM at 8K | **Handles 20K** |

> *"The performance gains of PARD are attributed to its parallel token prediction capability during the draft phase, which reduces the number of forward passes required by the draft model from K to 1."*  
> — [PARD Paper](https://arxiv.org/abs/2504.18583)

### ParallelSpec (Xiao et al., 2024)

> *"ParallelSpec proposed parallel drafting with a single transformer layer, but omits critical implementation details—notably whether and how target model hidden states are utilized—and does not address the memory scaling challenges that arise from extended training sequences."*  
> — [arXiv:2602.01469](https://arxiv.org/abs/2602.01469)

---

## 📚 References

- [arXiv:2602.01469](https://arxiv.org/abs/2602.01469) — primary paper
- [AWS ML Blog](https://aws.amazon.com/blogs/machine-learning/p-eagle-faster-llm-inference-with-parallel-speculative-decoding-in-vllm/)
- [PARD (arXiv:2504.18583)](https://arxiv.org/abs/2504.18583)
- [vLLM Speculators RFC](https://github.com/vllm-project/speculators/issues/292)
