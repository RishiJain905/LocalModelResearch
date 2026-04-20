# Mamba-3 Results & Benchmarks

> **Source:** [arXiv:2603.15569](https://arxiv.org/abs/2603.15569) · [Together AI Blog](https://www.together.ai/blog/mamba-3) · [OpenReview](https://openreview.net/forum?id=HwCvaJOiCj)  
> **Subtopic of:** [Mamba-3](./README.md)

---

## Language Modeling (FineWeb-Edu, 100B tokens)

### 1.5B Scale

| Model | FW-Edu Perplexity ↓ | LAMB Perplexity ↓ | Avg. Downstream Acc ↑ |
|---|---|---|---|
| Transformer | 10.51 | 11.1 | 55.4 |
| Gated DeltaNet | 10.45 | 10.9 | 55.8 |
| Mamba-2 | 10.47 | 12.0 | 55.7 |
| Mamba-3 SISO | 10.35 | 10.9 | 56.4 |
| **Mamba-3 MIMO (R=4)** | **10.24** | **10.2** | **57.6** |

- Mamba-3 SISO: **+0.6 pp** over Gated DeltaNet (next best recurrent)
- Mamba-3 MIMO: **+1.8 pp** over Gated DeltaNet total (+1.2 pp over SISO)
- Mamba-3 MIMO: **+2.2 pp** over Transformer

### 820M Scale

| Model | FW-Edu PPL | LAMB PPL | Avg Acc |
|---|---|---|---|
| Transformer | 11.42 | 15.0 | 52.8 |
| Gated DeltaNet | 11.39 | 12.7 | 53.9 |
| Mamba-2 | 11.35 | 13.8 | 53.4 |
| Mamba-3 SISO | 11.23 | 12.9 | 54.4 |
| **Mamba-3 MIMO** | **11.11** | **11.8** | **55.3** |

---

## State Tracking (Formal Language Tasks, 440M scale)

| Model | Parity ↑ | Arithmetic w/o Brackets ↑ | Arithmetic w/ Brackets ↑ |
|---|---|---|---|
| **Mamba-3** | **100.00** | **98.51** | **87.75** |
| Mamba-3 (w/ Std. RoPE) | 1.56 | 20.70 | 2.62 |
| Mamba-3 (w/o RoPE) | 2.27 | 1.49 | 0.72 |
| Mamba-2 | 0.90 | 47.81 | 0.88 |
| Gated DeltaNet [-1,1] | 100.00 | 99.25 | 93.50 |

> *"Mamba-2 scored **0.9%** on parity → Mamba-3 achieves **100%**."*  
> — [arXiv:2603.15569](https://arxiv.org/abs/2603.15569)

---

## Inference Latency

### 1.5B Models — Prefill + Decode Latency (H100-SXM 80GB, batch=128)

| Model | n=512 | n=1024 | n=2048 | n=4096 | n=16384 |
|---|---|---|---|---|---|
| vLLM (Llama-3.2-1B) | 4.45 | 9.60 | 20.37 | 58.64 | **976.50** |
| Gated DeltaNet | 4.56 | 9.11 | 18.22 | 36.41 | 145.87 |
| Mamba-2 | 4.66 | 9.32 | 18.62 | 37.22 | 149.02 |
| **Mamba-3 SISO** | **4.39** | **8.78** | **17.57** | **35.11** | **140.61** |
| Mamba-3 MIMO (r=4) | 4.74 | 9.48 | 18.96 | 37.85 | 151.81 |

> *"Mamba-3 (SISO variant) achieves the fastest prefill + decode latency across all sequence lengths, outperforming Mamba-2, Gated DeltaNet, and even the Transformer with its highly optimized vLLM ecosystem."*  
> — [Together AI Blog](https://www.together.ai/blog/mamba-3)

**Speedup at 16K tokens:** Llama-3.2-1B (vLLM): **122.06s** vs Mamba-3 SISO: **54.79s** (~2.2× faster)

> *"7× faster on long-sequence inference vs Transformers at 128K+ tokens."*  
> — [arXiv:2603.15569](https://arxiv.org/abs/2603.15569)

### Prefill Latency Only (H100-SXM 80GB, 1.5B)

| Model | n=512 | n=1024 | n=2048 | n=4096 | n=16384 |
|---|---|---|---|---|---|
| vLLM (Llama-3.2-1B) | 0.26 | 0.52 | 1.08 | 2.08 | 1.52 |
| Gated DeltaNet | 0.48 | 0.95 | 1.90 | 3.79 | 1.91 |
| Mamba-2 | 0.48 | 0.96 | 1.91 | 3.81 | 1.92 |
| **Mamba-3 SISO** | **0.48** | **0.95** | **1.90** | **3.80** | **1.91** |
| Mamba-3 MIMO (r=4) | 0.58 | 1.15 | 2.30 | 4.55 | 2.36 |

---

## Theoretical Scaling Advantage

| Sequence Length | Transformer Cost (Relative) | SSM Cost (Relative) |
|---|---|---|
| 4K tokens | 1× | 1× |
| 32K tokens | 64× | 8× |
| 128K tokens | 1,024× | 32× |
| **1M tokens** | **62,500×** | **250×** |

---

## Hybrid Mamba-3 (5:1 Mamba-to-Attention Ratio)

Architecture: 4 attention layers, 1.5B scale, 5:1 Mamba:attention block ratio

### Real-World Retrieval

| Model | SWDE ↑ | SQD ↑ | FDA ↑ | TQA ↑ | NQ ↑ | DROP ↑ |
|---|---|---|---|---|---|---|
| Transformer | 48.9 | 46.6 | 58.4 | **67.5** | 31.7 | 26.4 |
| **Mamba-3 Hybrid** | **58.5** | **47.0** | **65.9** | 64.8 | **33.4** | 27.0 |

> *Hybrid outperforms Transformer by **~10pp on SWDE** and **~7pp on FDA**.*

### Long-Context Retrieval (NIAH, 1.3B Hybrid w/ SP)

| Model | NIAH-1 (8K) | NIAH-1 (16K) | NIAH-1 (32K) |
|---|---|---|---|
| Mamba-3 1.3B | 31.2 | 13.6 | 7.4 |
| **Mamba-3 Hybrid w/ SP** | **98.8** | **99.0** | 32.2 |

### Reasoning Benchmarks (1.5B)

| Model | BoolQ ↑ | CSQA ↑ | MMLU ↑ | SciQ ↑ | LogiQA2 ↑ |
|---|---|---|---|---|---|
| Mamba-3 SISO | 60.6 | 21.3 | 34.9 | 91.0 | 28.2 |
| **Mamba-3 Hybrid** | **64.1** | **22.5** | 34.5 | **92.5** | 28.5 |

---

## DNA / Genomics

> *"In addition to language text they tested it on DNA sequences. It was able to outperform Pythia, RWKV, OPT, and GPT-Neo models of comparable size."*  
> — [HackerNoon](https://hackernoon.com/mamba-outperforms-hyenadna-in-dna-sequence-modeling)

- Mamba outperforms HyenaDNA in DNA sequence modeling
- Reference: [PMC: Selective State Space Models for functional genomics](https://pmc.ncbi.nlm.nih.gov/articles/PMC11870438/)

---

## Audio

- **AudioMamba** achieves similar performance to AST (Audio Spectrogram Transformer) for audio classification
- For audio generation, Mamba-based models have been trained successfully for speech synthesis
- Key use cases: audio classification, voice/speech synthesis, audio generation models

---

## Third-Party Reproductions & Independent Evaluations

### ICLR 2026 Peer Reviews

> *"Theoretically grounded SSM-focused improvements"* and *"thorough empirical validation across numerous scales which show the strengths of Mamba-3 over current state-of-the-art sub-quadratic models."*  
> — ICLR Reviewers (scores: kFu5, 5Dn9, h5BJ, 2tn7)

**Verdict:** Accepted to **ICLR 2026 (Oral)**

### Production Hybrid Models Already Shipping

| Model | Architecture | Key Stats |
|---|---|---|
| **NVIDIA Nemotron-H** | 92% Mamba-2 + 8% attention | Up to 3× faster throughput than Llama-3.1 at matched accuracy |
| **AI21 Jamba 1.5** | 1:7 attention-to-Mamba ratio | 398B total params, 256K context |
| **IBM Bamba-9B** | Hybrid Mamba + attention | Production deployment |

### Community Discussion

> *"Mamba-3 SISO beats Mamba-2, Gated DeltaNet, and even Llama-3.2-1B (Transformer) on prefill+decode latency across all sequence lengths at the 1.5B scale."*  
> — [r/LocalLLaMA](https://reddit.com/r/LocalLLaMA)

---

## Key Claims Summary

| Metric | Mamba-3 Claim | Evidence |
|---|---|---|
| Language modeling accuracy | +1.8pp over Gated DeltaNet at 1.5B | FW-Edu + LAMB benchmarks |
| Parity state tracking | 100% (vs Mamba-2's 0.9%) | Formal language task |
| Inference latency | Fastest at 1.5B across all seq lengths | H100 benchmark |
| State efficiency | 2× state reduction for same quality | Perplexity comparison |
| Long-context (128K+) | ~7× faster than Transformer | BuildMVPFast analysis |
| Hybrid retrieval | +10pp SWDE over Transformer | ICLR poster |

---

## Caveats & Open Questions

1. **Not yet proven at frontier scales (100B+)** — tested primarily at sub-10B parameter scales
2. **Short-sequence tasks** — advantage is marginal
3. **Hybrid design not fully understood** — pre-output projection, norm placement have non-negligible effects
4. **NIAH-1 at 32K** — hybrid drops to 32.2% vs 99%+ at shorter lengths
5. **Training cost increases** with MIMO but inference stays constant (asymmetric behavior)

---

## 📚 References

- [arXiv:2603.15569](https://arxiv.org/abs/2603.15569) — primary paper
- [OpenReview](https://openreview.net/forum?id=HwCvaJOiCj)
- [ICLR 2026 Poster](https://iclr.cc/virtual/2026/poster/10010352)
- [Together AI Blog](https://www.together.ai/blog/mamba-3)
- [BuildMVPFast Analysis](https://www.buildmvpfast.com/blog/mamba-3-vs-transformer-ssm-inference-2026)
- [PMC DNA Paper](https://pmc.ncbi.nlm.nih.gov/articles/PMC11870438/)
- [HackerNoon DNA](https://hackernoon.com/mamba-outperforms-hyenadna-in-dna-sequence-modeling)
