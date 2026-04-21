# Sub-Quadratic & Hybrid Runtimes (SSMs)

> **Papers:** [Mamba arXiv:2312.00752](https://arxiv.org/abs/2312.00752) · [Mamba-2 arXiv:2405.21060](https://arxiv.org/abs/2405.21060) · [RetNet arXiv:2307.08621](https://arxiv.org/abs/2307.08621)  
> **GitHub:** [state-spaces/mamba](https://github.com/state-spaces/mamba) · [HazyResearch/H3](https://github.com/HazyResearch/H3) · [fla-org/flash-linear-attention](https://github.com/fla-org/flash-linear-attention)  
> **Blogs:** [Tri Dao: Mamba-2 Part I](https://tridao.me/blog/2024/mamba2-part1-model/) · [Together AI: Mamba-3](https://www.together.ai/blog/mamba-3)

---

## Overview

> *"Mamba enjoys fast inference (5× higher throughput than Transformers) and linear scaling in sequence length."* — [Mamba Paper](https://arxiv.org/abs/2312.00752)

Sub-quadratic runtimes reduce the O(N²) attention complexity that dominates Transformer inference cost. State Space Models (SSMs), linear recurrents, and hybrid architectures achieve O(N) or O(N log N) inference cost while maintaining competitive quality.

---

## Core SSM Architectures

### 1. Mamba — Selective State Spaces

> **Paper:** [Mamba: Linear-Time Sequence Modeling with Selective State Spaces](https://arxiv.org/abs/2312.00752) — Albert Gu, Tri Dao (Dec 2023)  
> **GitHub:** [state-spaces/mamba](https://github.com/state-spaces/mamba) — 17.8k stars

> *"Mamba-3B model outperforms Transformers of the same size and matches Transformers twice its size."*

| Aspect | Details |
|--------|---------|
| **Complexity** | O(N) inference |
| **Throughput** | 5× higher than Transformers |
| **Key innovation** | Selective SSMs with input-dependent gating |
| **Architecture** | Hardware-aware parallel algorithm; no attention/MLP blocks |

**Architecture highlights:**
- Input-dependent gating (selective state spaces)
- Hardware-aware parallel algorithm in CUDA
- No attention or MLP blocks needed

### 2. Mamba-2 — State Space Duality (SSD)

> **Paper:** [Transformers are SSMs: Generalized Models and Efficient Algorithms Through Structured State Space Duality](https://arxiv.org/abs/2405.21060) — ICML 2024  
> **Blogs:** [Part I: The Model](https://tridao.me/blog/2024/mamba2-part1-model/) · [Part II: The Theory](https://goombalab.github.io/blog/2024/mamba2-part2-theory/)

> *"A state-space model with a scalar-times-identity state matrix is equivalent to a masked self-attention with a 1-semiseparable causal mask."*

| Aspect | Details |
|--------|---------|
| **Key insight** | SSMs and attention are mathematically equivalent under structured matrices |
| **Contribution** | Unified SSM/attention theory; improved training efficiency |

### 3. Mamba-3

> **Paper:** [Mamba-3: Improved Sequence Modeling using State Space Principles](https://arxiv.org/abs/2603.15569) — March 2026  
> **Blog:** [Together AI: Mamba-3](https://www.together.ai/blog/mamba-3)

> *"Decode latency is competitive across recurrent models... recurrent mixers scale more gently with context length than vLLM Llama-3.2-1B."*

---

## Linear Recurrent Models

### xLSTM — Extended LSTM

> **Paper:** [xLSTM: Extended Long Short-Term Memory](https://arxiv.org/abs/2405.04517)  
> **GitHub:** [NX-AI/xlstm](https://github.com/NX-AI/xlstm)

> *"Exponential gating and modified memory structures boost xLSTM capabilities to perform favorably when compared to state-of-the-art Transformers and State Space Models."*

| Metric | Value |
|--------|-------|
| **Parameters** | 409M |
| **SlimPajama perplexity** | 13.43 (vs Mamba 13.70, Llama 14.25) |
| **Key innovation** | Exponential gating + modified memory structures |

### RWKV — Reinventing RNNs for the Transformer Era

> **Paper:** [RWKV: Reinventing RNNs for the Transformer Era](https://arxiv.org/abs/2307.08621) — Microsoft Research  
> **GitHub:** [weigao266/Awesome-Efficient-Arch](https://github.com/weigao266/Awesome-Efficient-Arch)

> *"Combines the efficient parallelizable training of transformers with the efficient inference of RNNs... constant computational and memory complexity during inference."*

| Aspect | Details |
|--------|---------|
| **Training** | Parallelizable (like Transformer) |
| **Inference** | O(1) memory and compute |
| **Latest** | RWKV-X hybrid architecture |

### RetNet — Retentive Network

> **Paper:** [Retentive Network: A Successor to Transformer for Large Language Models](https://arxiv.org/abs/2307.08621) — Microsoft

> *"RetNet achieves low-cost inference (3.4×/15.6×/8.4× better throughput/latency/memory), parallel training, and favorable scaling curves."*

| Representation | Complexity |
|---------------|------------|
| **Parallel** | O(N) training |
| **Recurrent** | O(1) inference |
| **Chunkwise** | O(N log N) for long contexts |

---

## Hybrid Transformer-SSM Architectures

### Jamba — AI21 Labs

> **Paper:** [JAMBA: Hybrid Transformer-Mamba Language Models](https://openreview.net/pdf?id=JFPaD7lpBD)  
> **Blog:** [AI21: Rise of Hybrid LLMs](https://www.ai21.com/blog/rise-of-hybrid-llms/)  
> **HuggingFace:** [ai21labs](https://huggingface.co/ai21labs)

> *"First large-scale hybrid Transformer–Mamba–MoE language model... 3 times higher throughput on long contexts."*

| Model | Context | Notes |
|-------|---------|-------|
| Jamba-1.5-Mini | 256K | Publicly available |
| Jamba-1.5-Large | 256K | Publicly available |

### H3 — Hungry Hungry Hippos

> **Paper:** [Hungry Hungry Hippos: Towards Language Modeling with State Space Models](https://arxiv.org/abs/2212.14052)  
> **GitHub:** [HazyResearch/H3](https://github.com/HazyResearch/H3)  
> **Blog:** [From HiPPO to H3](https://openreview.net/forum?id=-n8d1eueJ) — ICLR 2023

> *"Hybrid H3-attention language models generate text 2.4× faster than Transformers at 2.7B parameters."*

---

## Gated Linear Attention & DeltaNet

### Gated Linear Attention (GLA)

> **Paper:** [Gated Linear Attention Transformers with Hardware-Efficient Training](https://arxiv.org/abs/2312.06635)

> *"GLA Transformer performs competitively against LLaMA-architecture Transformer as well recent linear-time-inference baselines such as RetNet and Mamba."*

### Gated DeltaNet

> **Blog:** [Sebastian Raschka: Gated DeltaNet](https://sebastianraschka.com/llms-from-scratch/ch04/08_deltanet/)  
> **Paper:** [Improving Mamba2 with Delta Rule](https://jankautz.com/publications/GatedDeltaNet_ICLR25.pdf)

> *"Gated DeltaNet is one of the clearest examples... practical compromise: replace most full-attention layers with a recurrent memory update, but keep some full-attention layers."*

**Used in production:**
- **Qwen3-Next**
- **Kimi Linear** — 3:1 hybrid ratio of Gated DeltaNet to attention

---

## Sub-Quadratic Attention Alternatives

### Hyena Operator

> **Paper:** [Hyena hierarchy: towards larger convolutional language models](https://dl.acm.org/doi/10.5555/3618408.3619572)

> *"Subquadratic drop-in replacement for attention constructed by interleaving implicitly parametrized long convolutions and data-controlled gating."*

### Hierarchically Pruned Attention (HiP)

> **Paper:** [HiP: Training-Free Sub-quadratic Cost Transformer](https://arxiv.org/html/2406.09827v1)

> *"Reduces quadratic time complexity to O(T log T)... 2.7× speedup in prefill, 16.5× speedup in decode at 32k context."*

---

## Benchmark Comparisons

| Architecture | Inference Complexity | Key Advantage |
|-------------|---------------------|--------------|
| **Mamba** | O(N) | 5× throughput vs Transformers |
| **Mamba-2/SSD** | O(N) with duality | Unified SSM/attention view |
| **RWKV** | O(1) | Parallel training, RNN inference |
| **RetNet** | O(1) | Triple representation flexibility |
| **xLSTM** | O(N) | Extended LSTM with exponential gating |
| **Gated DeltaNet** | O(1) KV cache | 3:1 hybrid with attention (Qwen3-Next, Kimi) |
| **H3** | O(N log N) | Hybrid SSM-attention, 2.4× faster generation |

> *"SSMs achieved sequence lengths of 220K tokens within 24GB memory limits — approximately 3× longer than Transformers."* — [arXiv:2601.01237v1](https://arxiv.org/html/2601.01237v1)

---

## GitHub Repos

| Repo | Description |
|------|-------------|
| [state-spaces/mamba](https://github.com/state-spaces/mamba) | Official Mamba implementation (17.8k stars) |
| [state-spaces/s4](https://github.com/state-spaces/s4) | S4, HiPPO, SaShiMi, DSS, S4D, S4ND |
| [HazyResearch/H3](https://github.com/HazyResearch/H3) | H3 — Hybrid SSM-attention language model |
| [NX-AI/xlstm](https://github.com/NX-AI/xlstm) | Extended LSTM with exponential gating |
| [fla-org/flash-linear-attention](https://github.com/fla-org/flash-linear-attention) | Flash Linear Attention — integrate attention layers into linear models |
| [weigao266/Awesome-Efficient-Arch](https://github.com/weigao266/Awesome-Efficient-Arch) | Comprehensive list: RWKV, linear RNNs, efficient architectures |
| [rasbt/LLMs-from-scratch](https://github.com/rasbt/LLMs-from-scratch) | Sebastian Raschka: Gated DeltaNet implementation |
| [yyyujintang/Awesome-Mamba-Papers](https://github.com/yyyujintang/Awesome-Mamba-Papers) | Curated Mamba SSM papers |
| [gauravfs-14/awesome-mamba](https://github.com/gauravfs-14/awesome-mamba) | S4 and structured state space models |

---

## Blog Posts & Technical Reports

| Title | Source | URL |
|-------|--------|-----|
| **State Space Duality (Mamba-2) Part I** | Tri Dao | [Link](https://tridao.me/blog/2024/mamba2-part1-model/) |
| **State Space Duality (Mamba-2) Part II** | Goomba Lab | [Link](https://goombalab.github.io/blog/2024/mamba2-part2-theory/) |
| **From HiPPO to H3** | HazyResearch (OpenReview) | [Link](https://openreview.net/forum?id=-n8d1eueJ) |
| **Attention was never enough** | AI21 Labs | [Link](https://www.ai21.com/blog/rise-of-hybrid-llms/) |
| **Mamba-3 from Together AI** | Together AI | [Link](https://www.together.ai/blog/mamba-3) |
| **Mamba vs Transformers: Efficiency, Scale, and the Future** | Medium | [Link](https://michielh.medium.com/mamba-vs-transformers-efficiency-scale-and-the-future-of-ai-d7a8dedb4018) |
| **The Annotated S4** | SRush | [Link](https://srush.github.io/annotated-s4/) |
| **Debunking claims about subquadratic attention** | LessWrong | [Link](https://www.lesswrong.com/posts/kpSXeMcthtHgnwMx3/debunking-claims-about-subquadratic-attention) |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Problem solved** | O(N²) attention bottleneck during LLM inference |
| **Key innovation** | Selective state spaces, linear recurrents, SSM-attention duality |
| **Best for** | Long context inference, high-throughput serving, memory-constrained部署 |

---

*Last updated: 2026-04-21*
