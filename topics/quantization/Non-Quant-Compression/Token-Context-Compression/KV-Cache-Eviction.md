# KV-Cache Eviction & Compression

> **Key Papers:** [H₂O NeurIPS 2023](https://arxiv.org/abs/2306.14048) · [StreamingLLM ICLR 2024](https://arxiv.org/abs/2309.17453) · [SnapKV NeurIPS 2024](https://arxiv.org/abs/2404.14469) · [GEAR arXiv:2403.05527](https://arxiv.org/abs/2403.05527) · [PyramidKV arXiv:2406.02069](https://arxiv.org/abs/2406.02069)  
> **GitHub:** [FMInference/H2O](https://github.com/FMInference/H2O) · [mit-han-lab/streaming-llm](https://github.com/mit-han-lab/streaming-llm) · [NVIDIA/kvpress](https://github.com/NVIDIA/kvpress) · [opengear-project/GEAR](https://github.com/opengear-project/GEAR)  
> **Blogs:** [MartinLwx — LLM Inference Optimization: KV Cache](https://martinlwx.github.io/en/llm-inference-optimization-kv-cache/) · [vLLM PagedAttention](https://docs.vllm.ai/en/latest/design/paged_attention/)

---

## Overview

KV cache scales linearly with sequence length and batch size, creating severe memory bottlenecks for long-context LLM deployment. KV-cache eviction/compression methods decide which tokens to keep in memory during autoregressive generation.

| Aspect | Details |
|--------|---------|
| **Problem** | KV cache memory scales O(batch × seq_len) |
| **Approach** | Evict, quantize, or compress KV entries |
| **Key Methods** | Heavy-hitter eviction, attention-sink retention, layer-adaptive budgets |
| **Speedup** | Up to 29× throughput improvement (H₂O) |
| **Compression Ratios** | Up to 32× at 128k sequence length (SnapKV) |

---

## Key Methods

### 1. H₂O — Heavy-Hitter Oracle (NeurIPS 2023)

> **Paper:** [arXiv:2306.14048](https://arxiv.org/abs/2306.14048) · **GitHub:** [FMInference/H2O](https://github.com/FMInference/H2O)

**Core Concept:**
> *"A small portion of tokens contributes most of the value when computing attention scores. We call these tokens Heavy Hitters (H₂)."*

Heavy Hitters emerge naturally and strongly correlate with frequent co-occurrence of tokens.

**Method:** Formulates KV cache eviction as a **dynamic submodular problem** with theoretical guarantees. Dynamically retains balance of **recent tokens** and **H₂ tokens**.

**Scoring Function:** Based on past attention values (cache information sum).

**Results with 20% heavy hitters:**

| System | Throughput Improvement |
|--------|----------------------|
| vs. DeepSpeed Zero-Inference | **29×** |
| vs. Hugging Face Accelerate | **29×** |
| vs. FlexGen | **3×** |
| Latency reduction | **1.9×** |

---

### 2. StreamingLLM — Attention Sinks (ICLR 2024)

> **Paper:** [arXiv:2309.17453](https://arxiv.org/abs/2309.17453) · **GitHub:** [mit-han-lab/streaming-llm](https://github.com/mit-han-lab/streaming-llm)

**Key Discovery:**
> *"We introduce StreamingLLM, an efficient framework that enables LLMs trained with a finite length attention window to generalize to infinite sequence lengths without any fine-tuning."*

**Observation:** Keeping KV of **initial tokens** (attention sinks) largely recovers window attention performance.

**Method:**
- Retains only **4 initial tokens** (attention sinks) + **recent tokens**
- KV cache divided into: attention sinks + streaming tokens
- Can process up to **4 million tokens** with stable performance

```python
# StreamingLLM retains attention sinks + recent tokens
KV_cache = [initial_tokens] + [recent_tokens]  # Discard intermediate
```

---

### 3. SnapKV — Automatic KV Selection (NeurIPS 2024)

> **Paper:** [arXiv:2404.14469](https://arxiv.org/abs/2404.14469) · **Blog:** [The Salt](https://thesalt.substack.com/p/compress-the-kv-cache-with-snapkv)

**Key Insight:**
> *"Each attention head consistently focuses on specific prompt attention features during generation."*

SnapKV automatically compresses KV caches by selecting clustered important KV positions **per attention head**.

**Method:**
1. **Observation window** (e.g., 16 tokens) profiles attention patterns
2. **Max pooling** (kernel size 5) identifies crucial attention features per head
3. Selects top-KV positions per attention head based on observed importance

**Results:**

| Metric | Improvement |
|--------|-------------|
| Generation speed | **3.6×** increase |
| Memory efficiency | **8.2×** enhancement |
| Max context (A100-80GB) | **380K tokens** |
| Compression ratio | **32×** at 128k sequence length |

> *"SnapKV can process up to 380K context tokens on a single A100-80GB GPU...exhibiting only a negligible accuracy drop in the Needle-in-a-Haystack test."*

---

### 4. GEAR — Near-Lossless KV Cache Compression (arXiv:2403.05527)

> **Paper:** [arXiv:2403.05527](https://arxiv.org/abs/2403.05527) · **GitHub:** [opengear-project/GEAR](https://github.com/opengear-project/GEAR)

**Key Quote:**
> *"We propose GEAR, an efficient KV cache compression framework that achieves near-lossless high-ratio compression."*

**Three-Stage Compression:**

| Stage | Method | Purpose |
|-------|--------|---------|
| **Quantization** | Ultra-low precision for similar-magnitude entries | Primary compression |
| **Low-Rank Approximation** | Low-rank matrix for quantization error | Capture global patterns |
| **Sparse Recovery** | Sparse matrix for outlier errors | Handle exceptions |

**Key Differentiator:** Does NOT need to preserve any first or last tokens uncompressed (unlike other low-bit compression algorithms).

**Results:**
| Metric | Improvement |
|--------|-------------|
| Compression ratio | **5.02×** with 4-bit quantization |
| Throughput | **~5×** improvement |
| Memory (vs FP16) | **41%** reduction |

---

### 5. PyramidKV — Dynamic Layer-Adaptive Budgets (arXiv:2406.02069)

> **Paper:** [arXiv:2406.02069](https://arxiv.org/abs/2406.02069)

**Key Quote:**
> *"PyramidKV consists of two steps: (1) Dynamically allocating different KV cache sizes/budgets across different layers; and (2) Selecting important KV vectors in each attention head for caching."*

**Observation:** Attention patterns vary from layer to layer — pyramidal information funneling.

**Method:**
- Different budgets per layer (pyramid shape)
- Leverages pyramidal information funneling phenomenon
- Significantly outperforms baselines on few-shot learning tasks

---

### 6. FastGen — Adaptive Hybrid Eviction (Answer.ai)

> **Code:** [AnswerDotAI/cold-compress](https://github.com/AnswerDotAI/cold-compress) (formerly [machilusZ/FastGen](https://github.com/machilusZ/FastGen))

**Paper:** *"Model Tells You What to Discard: Adaptive KV Cache Compression for LLMs"*

**Method:**
- Profiles attention matrix at end of prefill stage
- Picks **hybrid portfolio of eviction strategies** per attention head per instance
- Strategies: special tokens, local context, top-k selection

---

### 7. LookaheadKV — Predicting Future Attention (ICLR 2026)

> **Paper:** [arXiv:2603.10899](https://arxiv.org/abs/2603.10899) · **OpenReview:** [Link](https://openreview.net/forum?id=RVLMGPXt2i)

**Key Quote:**
> *"We propose LookaheadKV, a lightweight framework that improves KV cache eviction in transformers by using parameter-efficient modules to predict future attention patterns."*

**Method:**
- Lightweight eviction framework using surrogate future response
- **No explicit draft generation required**
- Augments LLM with parameter-efficient modules

---

### 8. PagedAttention (vLLM)

> **Docs:** [vLLM PagedAttention](https://docs.vllm.ai/en/latest/design/paged_attention/)

**Key Concept:**
> *"PagedAttention divides the KV cache into fixed-size blocks, allowing non-contiguous storage in GPU memory."*

**Method:**
- KV blocks (pages) — fixed size measured in tokens per block
- Global free block pool (doubly linked list)
- Block tables mapping logical → physical blocks
- Reduces both internal and external fragmentation

---

## NVIDIA kvpress — Unified Framework

> **GitHub:** [NVIDIA/kvpress](https://github.com/NVIDIA/kvpress) · **Paper:** [arXiv:2510.00636](https://arxiv.org/abs/2510.00636v1) · **HF Space:** [nvidia/kvpress](https://huggingface.co/spaces/nvidia/kvpress)

20+ compression methods implemented in one toolkit:

| Category | Methods |
|----------|---------|
| **Scorer-based** | RandomPress, KNormPress, SnapKVPress, ExpectedAttentionPress, StreamingLLMPress, TOVAPress, ObservedAttentionPress, QFilterPress, PyramidKVPress, LagKVPress, KeyDiffPress, LeverageScorePress, KVzapPress, DMSPress, FastKVzipPress |
| **Wrapper** | AdaKVPress, PerLayerCompressionPress, ComposedPress, ChunkKVPress, DecodingPress, PrefillDecodingPress |
| **Special** | ThinKPress, SimLayerKVPress, DuoAttentionPress, FinchPress, KVzipPress, KVComposePress |

```python
from kvpress import KNormPress
press = KNormPress(compression_ratio=0.5)
answers = pipe(context, questions=questions, press=press)
```

---

## Papers Table

| Paper | Year | Venue | URL | Key Contribution |
|-------|------|-------|-----|-----------------|
| **H₂O** | 2023 | NeurIPS 2023 | [arXiv](https://arxiv.org/abs/2306.14048) | Heavy-hitter eviction, 29× throughput |
| **StreamingLLM** | 2023 | ICLR 2024 | [arXiv](https://arxiv.org/abs/2309.17453) | Attention sinks, 4M token streaming |
| **GEAR** | 2024 | — | [arXiv](https://arxiv.org/abs/2403.05527) | Near-lossless 3-stage compression |
| **SnapKV** | 2024 | NeurIPS 2024 | [arXiv](https://arxiv.org/abs/2404.14469) | Per-head automatic selection, 32× ratio |
| **PyramidKV** | 2024 | — | [arXiv](https://arxiv.org/abs/2406.02069) | Layer-adaptive budget allocation |
| **FastGen** | — | — | [GitHub](https://github.com/AnswerDotAI/cold-compress) | Hybrid eviction portfolio |
| **LookaheadKV** | 2026 | ICLR 2026 | [arXiv](https://arxiv.org/abs/2603.10899) | Predicts future attention patterns |
| **SAGE-KV** | 2025 | — | [arXiv:2503.08879](https://arxiv.org/abs/2503.08879) | Self-attention guided eviction |
| **LAVA** | 2025 | — | [arXiv:2509.09754](https://arxiv.org/abs/2509.09754) | Layer-wise dynamic eviction |
| **IntelLLM** | 2025 | ICLR 2025 | [OpenReview](https://openreview.net/forum?id=4QWPCTLq20) | Hint-based compression |
| **KVzap** | 2026 | — | [arXiv:2601.07891](https://arxiv.org/abs/2601.07891) | Fast adaptive pruning, GQA-compatible |

---

## GitHub Repos

| Repo | Description |
|------|-------------|
| [FMInference/H2O](https://github.com/FMInference/H2O) | Heavy-Hitter Oracle (NeurIPS 2023) |
| [mit-han-lab/streaming-llm](https://github.com/mit-han-lab/streaming-llm) | StreamingLLM (ICLR 2024) |
| [opengear-project/GEAR](https://github.com/opengear-project/GEAR) | Near-lossless 3-stage compression |
| [NVIDIA/kvpress](https://github.com/NVIDIA/kvpress) | Unified toolkit with 20+ methods |
| [AnswerDotAI/cold-compress](https://github.com/AnswerDotAI/cold-compress) | FastGen adaptive compression |
| [mit-han-lab/streaming-llm](https://github.com/mit-han-lab/streaming-llm) | StreamingLLM |
| [Zefan-Cai/Awesome-LLM-KV-Cache](https://github.com/Zefan-Cai/Awesome-LLM-KV-Cache) | Curated KV cache resources |
| [October2001/Awesome-KV-Cache-Compression](https://github.com/October2001/Awesome-KV-Cache-Compression) | 679 stars — comprehensive list |

---

## Blog Posts & Technical Reports

| Title | Source | URL |
|-------|--------|-----|
| **LLM inference optimization: KV Cache** | MartinLwx | [Link](https://martinlwx.github.io/en/llm-inference-optimization-kv-cache/) |
| **KV caching, a deeper look** | Medium | [Link](https://medium.com/@plienhar/llm-inference-series-4-kv-caching-a-deeper-look-4ba9a77746c8) |
| **PagedAttention from First Principles** | Hamza Elshafie | [Link](https://hamzaelshafie.bearblog.dev/paged-attention-from-first-principles-a-view-inside-vllm/) |
| **Compress the KV Cache with SnapKV** | The Salt | [Link](https://thesalt.substack.com/p/compress-the-kv-cache-with-snapkv) |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Problem solved** | KV cache memory bottleneck for long-context LLM inference |
| **Key innovation** | Heavy-hitter retention (H₂O), attention sinks (StreamingLLM), per-head selection (SnapKV), near-lossless 3-stage (GEAR) |
| **Speedup** | Up to 29× throughput (H₂O); 3.6× generation speed (SnapKV) |
| **Compression ratio** | Up to 32× at 128k (SnapKV); 5× with GEAR + 4-bit quantization |
| **Best for** | Long-context scenarios; streaming generation; memory-constrained deployment |

---

*Last updated: 2026-04-19*
