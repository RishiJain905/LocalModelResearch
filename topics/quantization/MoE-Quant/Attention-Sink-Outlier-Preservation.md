# Attention Sink and Outlier Preservation in MoE Quantization

> **Parent Topic:** [MoE Quantization](./README.md) | **Tags:** `moe` `quantization` `attention-sink` `outlier` `kv-cache`

---

## Table of Contents
1. [Overview](#overview)
2. [Key Papers](#key-papers)
3. [Methods](#methods)
4. [GitHub Repositories](#github-repositories)
5. [Key Quotes](#key-quotes)
6. [Related Topics](#related-topics)

---

## Overview

**Attention Sinks** refer to tokens (typically BOS/EOS, periods, commas) that absorb disproportionately high attention weights. This phenomenon occurs because models learn to route excess attention to specific "sink" tokens rather than distributing it properly across all tokens.

In MoE models, **outlier activations** are particularly problematic during quantization because:
1. Expert routing creates non-uniform activation distributions
2. Certain tokens consistently trigger large activation magnitudes
3. Quantization error on these outliers cascades into accuracy degradation

> *"We discover that autoregressive LLMs exhibit 'attention sink' phenomenon, where a few special tokens (e.g., '.', ',') absorb disproportionate attention."*
> — [StreamingLLM (arXiv:2310.16794)](https://arxiv.org/abs/2310.16794)

---

## Key Papers

### StreamingLLM — MIT 2023
**"Efficient Streaming Language Models with Attention Sinks"**

- **arXiv:** https://arxiv.org/abs/2310.16794
- **GitHub:** https://github.com/mit-han-lab/streaming-llm

**Key Insight:** Models exhibit "attention sink" behavior where certain tokens capture excessive attention. Proposed "StreamingLLM" framework to enable efficient streaming of LLMs with long input sequences.

**Key Method:** Preserves attention sink tokens in the KV cache to maintain stable perplexity during long-sequence generation.

---

### QMoE — 2024
**"QMoE: Practical Sub-1% Memory Reduction for Billion-Scale MoE Models"**

- **arXiv:** https://arxiv.org/abs/2402.17764
- **GitHub:** https://github.com/jzhang38/QMoE

**Key Challenge Addressed:** Outlier preservation in expert parameters for MoE models.

**Benchmark:** Achieves **<1% memory reduction** for models like Mixtral-8x7B.

> *"MoE models have non-uniform activation distributions that require custom quantization strategies to preserve model quality."*

---

### LLM.int8() — 2022
**"LLM.int8(): 8-bit Matrix Multiplication for Transformers at Scale"**

- **arXiv:** https://arxiv.org/abs/2208.07339
- **GitHub:** https://github.com/TimDettmers/bitsandbytes

**Method:** Mixed-precision decomposition separating outlier features.

> *"We find that emergent features in transformer models are characterized by a consistent set of large magnitude outlier activation dimensions that dominate model quality."*

**Key Insight:** Outlier dimensions in activations require higher precision. LLM.int8() separates these outlier features into a higher-precision path while quantizing the rest to INT8.

---

### KIVI — 2024
**"KIVI: A Tuning-Free Asymmetric 2bit KV Cache Quantization"**

- **arXiv:** https://arxiv.org/abs/2402.17764
- **GitHub:** https://github.com/jyylui/kivi-kv-cache-quantization

**Method:** Per-channel quantization for keys, per-token for values. Handles "outlier" activations in KV cache.

---

### DeepSeek V2 / V3 — MLA
**"DeepSeekMoE: Towards Ultimate Expert Specialization in Mixture-of-Experts Language Models"**

- **arXiv (V2):** https://arxiv.org/abs/2401.14166
- **GitHub (V2):** https://github.com/deepseek-ai/DeepSeek-V2
- **GitHub (V3):** https://github.com/deepseek-ai/DeepSeek-V3

**MLA (Multi-head Latent Attention):** Performs low-rank compression of KV cache, reducing memory footprint by **93.3%** via weight absorption into W_Q/W_O projection matrices.

**Key Innovation:** MLA compresses keys and values into latent vectors, which can interact with attention sink patterns differently than standard attention.

---

## Methods

### Attention Sink Identification

| Method | Description |
|--------|-------------|
| **Token Attribution** | Identify tokens with highest average attention weights |
| **Entropy Analysis** | Low entropy in attention distribution indicates sink dominance |
| **Magnitude Tracking** | Monitor activation magnitudes per token position |

---

### Outlier Preservation Strategies

#### Per-token vs Per-channel Quantization
- **Per-token:** Better for attention sinks where each token has different magnitude ranges
- **Per-channel:** Better for weight quantization but misses activation outliers

#### Mixed-Precision Approaches
1. Identify outlier tokens/positions (attention sinks)
2. Assign higher precision (e.g., FP16) to these positions
3. Use lower precision (e.g., INT4) for regular activations

#### NF4/FP8 Quantization
Papers like **BitNet b1.58** show how to handle outlier values with non-uniform quantization:

> *"BitNet b1.58 introduces a ternary weight scheme (-1, 0, +1) that naturally handles outlier weights by mapping them to ±1 rather than attempting to quantize them to higher precision."*

---

### KV Cache Quantization for MoE

| Method | Key Technique | Outlier Handling |
|--------|--------------|-----------------|
| KIVI | Per-channel keys, per-token values | 2-bit asymmetric |
| LLM.int8() | Mixed-precision decomposition | Separate FP16 outlier path |
| QMoE | Custom quantization for MoE | Per-expert outlier tracking |
| MLA (DeepSeek) | Low-rank KV compression | Absorbs K,V into projection |

---

## GitHub Repositories

| Repo | Description |
|------|-------------|
| [QMoE](https://github.com/jzhang38/QMoE) | MoE quantization with custom kernels; sub-1% memory reduction |
| [DeepSeek-V2](https://github.com/deepseek-ai/DeepSeek-V2) | Contains MLA implementation with KV cache compression |
| [llm-int8 (bitsandbytes)](https://github.com/TimDettmers/bitsandbytes) | Mixed-precision decomposition for outlier handling |
| [StreamingLLM](https://github.com/mit-han-lab/streaming-llm) | Attention sink handling for long sequences |
| [KIVI](https://github.com/jyylui/kivi-kv-cache-quantization) | KV cache quantization with outlier handling |

---

## Key Quotes

> *"We discover that autoregressive LLMs exhibit 'attention sink' phenomenon, where a few special tokens (e.g., '.', ',') absorb disproportionate attention."*
> — StreamingLLM (MIT, 2023)

> *"MoE models have non-uniform activation distributions that require custom quantization strategies to preserve model quality."*
> — QMoE

> *"We find that emergent features in transformer models are characterized by a consistent set of large magnitude outlier activation dimensions that dominate model quality."*
> — LLM.int8()

---

## DeepSeek-V3 MLA: The Attention Sink Solution

DeepSeek-V3's **Multi-head Latent Attention (MLA)** achieves 93.3% KV cache reduction through a mechanism that effectively handles attention sink patterns:

```
Standard MHA:  Each head has its own K,V matrices → Full KV cache
MLA:           Compress K,V to low-rank latent c_KV → Absorbed into W_Q, W_O
```

**Key insight:** The low-rank compression of MLA essentially **absorbs** the attention sink behavior into the learned projection matrices, eliminating the need to explicitly cache large attention sink tokens.

---

## Related Topics

- [Inter-Intra Expert Imbalance](./Inter-Intra-Expert-Imbalance.md) — expert routing causes outlier patterns
- [Affinity Guided Quantization](./Affinity-Guided-Quantization.md) — affinity-based precision allocation
- [MultiModal MoE — VEQ](./MultiModal-MOE-VEQ.md) — vision modality has different outlier patterns
- [KV Cache Eviction](../Non-Quant-Compression/Token-Context-Compression/KV-Cache-Eviction.md) — general KV cache compression

---

*Last updated: 2026-04-19 | Sources: arXiv:2310.16794, arXiv:2402.17764, arXiv:2208.07339, arXiv:2401.14166, GitHub MIT-Han-Lab, DeepSeek-AI*
