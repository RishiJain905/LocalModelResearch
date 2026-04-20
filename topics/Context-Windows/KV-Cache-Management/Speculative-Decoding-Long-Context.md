# Speculative Decoding for Long Context

> **Key Papers:** [SpecInfer (ASPLOS 2024)](https://arxiv.org/abs/2305.09781) · [Medusa (arXiv:2401.10774)](https://arxiv.org/abs/2401.10774) · [EAGLE-3 (arXiv:2503.01840)](https://arxiv.org/abs/2503.01840) · [TriForce (COLM 2024)](https://arxiv.org/abs/2404.11912) · [LongSpec (ACL 2025)](https://arxiv.org/abs/2502.17421)
> **GitHub:** ⭐2,724 [FasterDecoding/Medusa](https://github.com/FasterDecoding/Medusa) · ⭐2,279 [SafeAILab/EAGLE](https://github.com/SafeAILab/EAGLE) · ⭐789 [sgl-project/SpecForge](https://github.com/sgl-project/SpecForge) · ⭐279 [Infini-AI-Lab/TriForce](https://github.com/Infini-AI-Lab/TriForce)
> **Blogs:** [NVIDIA — Intro to Speculative Decoding](https://developer.nvidia.com/blog/an-introduction-to-speculative-decoding-for-reducing-latency-in-ai-inference/) · [Together AI — MagicDec](https://www.together.ai/blog/speculative-decoding-for-high-throughput-long-context-inference) · [AWS — P-EAGLE](https://aws.amazon.com/blogs/machine-learning/p-eagle-faster-llm-inference-with-parallel-speculative-decoding-in-vllm/)

---

## Overview

> *"The fundamental bottleneck for long-context generation is KV cache loading — as context grows, the memory bandwidth required to load KV cache during verification dominates, making decoding memory-bound."* — MagicDec, ICLR 2025

Speculative decoding (SD) accelerates LLM inference with a **draft-then-verify** paradigm: a small draft model generates candidate tokens, and the target model verifies them in parallel. For long-context scenarios, SD becomes particularly effective because the **KV cache loading bottleneck** (not compute) becomes the limiting factor — verification parallelizes perfectly over the draft tokens.

---

## Speculative Decoding Fundamentals

### Core Papers

| Paper | Year | Venue | Key Contribution |
|-------|------|-------|-----------------|
| **Speculative Decoding** (Leviathan et al.) | 2023 | Google | Original speculative sampling paper |
| **SpecInfer** | 2024 | ASPLOS | Tree-based token verification, collectively boost-tuned SSMs |

> *"Using a smaller 60M parameter T5-small as the guessing mechanism, we get ~3× improvement in speed."* — [Google Research Blog](https://research.google/blog/looking-back-at-speculative-decoding/)

---

## Draft Model Architectures

### Medusa — Multiple Decoding Heads

> **Paper:** [arXiv:2401.10774](https://arxiv.org/abs/2401.10774) · **GitHub:** ⭐2,724 [FasterDecoding/Medusa](https://github.com/FasterDecoding/Medusa)

Adds multiple decoding heads to predict future tokens in parallel via tree-based attention. **No separate draft model needed.** 2.2×–3.6× speedup.

- **Medusa-1:** Freezes original model, trains only new heads
- **Medusa-2:** Full-model training with special recipe
- Integrated into TensorRT-LLM, HuggingFace TGI, Alibaba RTP-LLM

**Hydra (extension):** Sequentially-dependent draft heads with decoder layer for better context aggregation (ICML 2024).

### EAGLE — Feature-Level Speculative Decoding

> **Paper:** [arXiv:2401.15077](https://arxiv.org/abs/2401.15077) (ICML 2024) · **GitHub:** ⭐2,279 [SafeAILab/EAGLE](https://github.com/SafeAILab/EAGLE)

Operates at **feature level** (second-top-layer features), addresses sampling uncertainty. 3× faster than vanilla, 2× faster than Lookahead, 1.6× faster than Medusa.

| Version | Paper | Key Innovation | Speedup |
|---------|-------|---------------|---------|
| **EAGLE-1** | ICML 2024 | Feature-level drafting | 3× vanilla |
| **EAGLE-2** | EMNLP 2024 | Dynamic draft tree via confidence scores | ~4× on 13B |
| **EAGLE-3** | NeurIPS 2025 | Training-Time Test (TTT), direct token prediction | ~5.6× on 13B |
| **P-EAGLE** | AWS Blog Mar 2026 | Non-autoregressive parallel drafting | Removes AR bottleneck |

> *"EAGLE-3: Scaling up Inference Acceleration via Training-Time Test"* — replaces top-layer features with multi-layer feature fusion via TTT procedure. ~5.6× speedup on 13B, up to 6.5× on reasoning models.

Integrated into: **vLLM, SGLang, NVIDIA TensorRT-LLM, NeMo Framework, AMD ROCm, AWS NeuronX, Intel Extension for Transformers, PaddleNLP, MLC-LLM**.

### SpecForge — EAGLE-3 Training Framework

> **Paper:** [arXiv:2603.18567](https://arxiv.org/abs/2603.18567) · **GitHub:** ⭐789 [sgl-project/SpecForge](https://github.com/sgl-project/SpecForge) · **Blog:** [LMSYS](https://lmsys.org/blog/2025-07-25-spec-forge/)

Open-source EAGLE-3 draft model training framework integrated with SGLang. Supports FSDP, tensor parallelism, Training-Time Test procedure.

### Cascade Speculative Drafting

> **Paper:** [arXiv:2312.11462](https://arxiv.org/abs/2312.11462) (NeurIPS 2024) · **GitHub:** ⭐33 [lfsszd/CS-Drafting](https://github.com/lfsszd/CS-Drafting)

Two cascade types:
- **Vertical:** small draft → medium draft → target (multi-model hierarchy)
- **Horizontal:** single model drafts multiple tokens in parallel

### Sequoia — Hardware-Aware Trees

> **Paper:** [arXiv:2402.12374](https://arxiv.org/abs/2402.12374)

Dynamic programming to find **optimal token tree structure** given hardware characteristics. Serving LLaMA2-70B on a single RTX 4090 with average TBT of 0.57s — **8× faster than DeepSpeed-Zero-Inference**.

---

## Long-Context SD Adaptations

### Key Insight: Why SD Works for Long Context

> *"Counterintuitively, in the long-context regime, a larger batch size results in decoding being *more* memory bound, instead of the other way around."* — [MagicDec Blog](https://www.together.ai/blog/speculative-decoding-for-high-throughput-long-context-inference)

| Context Batch Regime | Bottleneck | SD Effectiveness |
|---|---|---|
| Short context, small batch | Model parameters (compute-bound) | Moderate |
| Short context, large batch | Model parameters (compute-bound) | Limited |
| **Long context, any batch** | **KV cache loading (memory-bound)** | **Effective** |

### TriForce — Hierarchical SD for Long Sequences

> **Paper:** [arXiv:2404.11912](https://arxiv.org/abs/2404.11912) · **Venue:** COLM 2024 · **GitHub:** ⭐279 [Infini-AI-Lab/TriForce](https://github.com/Infini-AI-Lab/TriForce)

Training-free, hierarchical SD for long sequences. Uses retrieval-based sparse KV cache for draft model + StreamingLLM for target model. **2.2× speedup** on single A100 for 128K context.

> *"Natural divergence between draft model with StreamingLLM/sliding-window and full KV cache increases with context length."*

### MagicDec — High-Throughput Long-Context

> **Paper:** [arXiv:2408.11049](https://arxiv.org/abs/2408.11049) · **Venue:** ICLR 2025 · **Blog:** [Together AI](https://www.together.ai/blog/speculative-decoding-for-high-throughput-long-context-inference)

Counterintuitive finding: for large batch + long context, decoding becomes memory-bound due to KV cache loading. Uses **self-speculation with StreamingLLM sliding window** for draft model (fixed draft KV cache size). Up to **2× speedup** on 8×A100.

### LongSpec — Lossless Long-Context SD

> **Paper:** [arXiv:2502.17421](https://arxiv.org/abs/2502.17421) · **Venue:** ACL 2025 Main · **GitHub:** ⭐82 [sail-sg/LongSpec](https://github.com/sail-sg/LongSpec)

Three innovations:
1. **Memory-efficient architecture:** Sliding-window self-attention (window=512) + cross-attention reusing target's KV cache → constant-sized draft KV cache
2. **Anchor-offset position indices:** Reserve tokens [0,1,2,3] as attention sinks + large random offsets → mitigates short-context training / long-context inference mismatch. Reaches same loss **3.93× faster**.
3. **Hybrid tree attention:** 75% latency reduction via split cached/speculative verification

**Results:** Up to **3.26× speedup** over FlashAttention baseline on long-context tasks. Up to **7×** over HuggingFace.

### SpecExtend — Drop-in Enhancement for Long Sequences

> **Paper:** [arXiv:2505.20776](https://arxiv.org/abs/2505.20776) · **GitHub:** ⭐5 [jycha98/SpecExtend](https://github.com/jycha98/SpecExtend)

Training-free. Two components: (1) FlashAttention + Hybrid Tree Attention for draft/target, (2) **Cross-model Retrieval (CMR)** — uses target's attention scores to dynamically select relevant KV cache chunks for draft model. Improves draft accuracy by up to **2.55×**.

> *"Draft accuracy degradation on long inputs cannot be attributed solely to position extrapolation — it's largely due to loss of semantically relevant context."*

### SpecPV — Partial Verification

> **Paper:** [arXiv:2512.02337](https://arxiv.org/abs/2512.02337)

Uses partial KV states for fast verification in long-context generation, with periodic full verification to eliminate accumulated errors. Short context → full verification; long context → partial verification.

### TokenSwift — Ultra-Long 100K Tokens

> **Paper:** [arXiv:2502.18890](https://arxiv.org/abs/2502.18890) · **Venue:** ICML 2025 · **GitHub:** ⭐123 [bigai-nlco/TokenSwift](https://github.com/bigai-nlco/TokenSwift)

Addresses three challenges for 100K+ token generation: (1) frequent model weight reloading → Medusa-style draft heads with partial KV cache, (2) repetitive generation → sliding window token penalty, (3) dynamic temperature adjustment.

### QuantSpec — Hierarchical Quantized KV Cache

> **Paper:** [arXiv:2502.10424](https://arxiv.org/abs/2502.10424) · **Venue:** ICML 2025

Self-speculative decoding using hierarchical 4-bit quantized KV caches. Custom CUDA kernels achieving **2.88× speedup** over FP16 attention.

---

## Recent 2025–2026 Advances

| Paper | Venue/Source | Key Innovation |
|-------|-------------|----------------|
| **P-EAGLE** | AWS Blog Mar 2026 | Non-autoregressive parallel drafting, removes AR bottleneck |
| **SpecForge** | LMSYS Jul 2025 | EAGLE-3 training framework for SGLang |
| **CARD** | arXiv:2508.04462 | Cache-assisted parallel SD, up to 4.83× |
| **When Drafts Evolve** | ICLR 2026 Workshop | Online learning for SD (DPO gradient, Hydra online) |
| **Adaptive SD with RL** | ICLR 2026 | RL to dynamically determine draft tree depth |
| **SVIP** | EMNLP 2025 | Draft model adaptively determines sequence length based on entropy |

---

## Papers Table

| Paper | Year | Venue | URL | Key Contribution |
|-------|------|-------|-----|------------------|
| **SpecInfer** | 2024 | ASPLOS | [arXiv:2305.09781](https://arxiv.org/abs/2305.09781) | Tree-based verification, boost-tuned SSMs |
| **Medusa** | 2024 | arXiv | [arXiv:2401.10774](https://arxiv.org/abs/2401.10774) | Multi-head parallel prediction, no draft model |
| **EAGLE-1** | 2024 | ICML | [arXiv:2401.15077](https://arxiv.org/abs/2401.15077) | Feature-level drafting |
| **EAGLE-2** | 2024 | EMNLP | — | Dynamic draft tree |
| **EAGLE-3** | 2025 | NeurIPS | [arXiv:2503.01840](https://arxiv.org/abs/2503.01840) | Training-Time Test, ~5.6× speedup |
| **TriForce** | 2024 | COLM | [arXiv:2404.11912](https://arxiv.org/abs/2404.11912) | Hierarchical SD, 2.2× at 128K |
| **MagicDec** | 2025 | ICLR | [arXiv:2408.11049](https://arxiv.org/abs/2408.11049) | Self-speculation + StreamingLLM, 2× at large batch |
| **LongSpec** | 2025 | ACL | [arXiv:2502.17421](https://arxiv.org/abs/2502.17421) | Constant draft KV, anchor-offset, hybrid tree, 3.26× |
| **SpecExtend** | 2025 | arXiv | [arXiv:2505.20776](https://arxiv.org/abs/2505.20776) | Cross-model KV retrieval for draft, 2.55× accuracy |
| **TokenSwift** | 2025 | ICML | [arXiv:2502.18890](https://arxiv.org/abs/2502.18890) | 100K+ token generation |
| **QuantSpec** | 2025 | ICML | [arXiv:2502.10424](https://arxiv.org/abs/2502.10424) | Hierarchical quantized KV cache, 2.88× |
| **Sequoia** | 2024 | arXiv | [arXiv:2402.12374](https://arxiv.org/abs/2402.12374) | Hardware-aware tree structure |
| **CS-Drafting** | 2024 | NeurIPS | [arXiv:2312.11462](https://arxiv.org/abs/2312.11462) | Vertical + horizontal cascades |

---

## GitHub Repos

| Repo | ⭐ | Description |
|------|-----|-------------|
| [FasterDecoding/Medusa](https://github.com/FasterDecoding/Medusa) | 2,724 | Multi-head SD — integrated into TensorRT-LLM, TGI |
| [SafeAILab/EAGLE](https://github.com/SafeAILab/EAGLE) | 2,279 | EAGLE-1/2/3 — integrated into vLLM, SGLang, TRT-LLM |
| [sgl-project/SpecForge](https://github.com/sgl-project/SpecForge) | 789 | EAGLE-3 training framework for SGLang |
| [Infini-AI-Lab/TriForce](https://github.com/Infini-AI-Lab/TriForce) | 279 | COLM 2024 — hierarchical SD for long sequences |
| [bigai-nlco/TokenSwift](https://github.com/bigai-nlco/TokenSwift) | 123 | ICML 2025 — ultra-long 100K+ token generation |
| [lfsszd/CS-Drafting](https://github.com/lfsszd/CS-Drafting) | 33 | NeurIPS 2024 — cascade SD |
| [jycha98/SpecExtend](https://github.com/jycha98/SpecExtend) | 5 | Training-free drop-in SD enhancement |
| [flexflow/flexflow-serve](https://github.com/flexflow/flexflow-serve) | 79 | SpecInfer implementation |

---

## Blog Posts

| Title | Source | URL |
|-------|--------|-----|
| **Intro to Speculative Decoding** | NVIDIA Developer Blog | [Link](https://developer.nvidia.com/blog/an-introduction-to-speculative-decoding-for-reducing-latency-in-ai-inference/) |
| **MagicDec** | Together AI Blog | [Link](https://www.together.ai/blog/speculative-decoding-for-high-throughput-long-context-inference) |
| **P-EAGLE** | AWS ML Blog | [Link](https://aws.amazon.com/blogs/machine-learning/p-eagle-faster-llm-inference-with-parallel-speculative-decoding-in-vllm/) |
| **SpecForge** | LMSYS Blog | [Link](https://lmsys.org/blog/2025-07-25-spec-forge/) |
| **EAGLE-3 Speculative Decoding on vLLM** | Red Hat Developer | [Link](https://developers.redhat.com/articles/2026/04/16/performance-improvements-speculative-decoding-vllm-gpt-oss) |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Core idea** | Draft-then-verify: small model drafts, target verifies in parallel |
| **Best draft methods** | EAGLE-3 (5.6×), Medusa (2.2–3.6×), Cascade (multi-model hierarchy) |
| **Long-context breakthrough** | KV cache loading is memory-bound — verification parallelizes perfectly |
| **Best for long context** | LongSpec (3.26×, constant draft KV), TriForce (2.2× at 128K), MagicDec (2× at large batch) |
| **Key long-context innovations** | Anchor-offset positions, hybrid tree attention, sliding-window drafts, partial verification |
| **Best integrated** | EAGLE-3 in vLLM + SGLang + TensorRT-LLM |
| **2026 advances** | P-EAGLE (parallel drafting), SpecForge (training framework), RL-adaptive trees |

---

*Last updated: 2026-04-20*