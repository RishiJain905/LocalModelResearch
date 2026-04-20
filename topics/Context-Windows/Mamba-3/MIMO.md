# Mamba-3 MIMO (Multi-Input, Multi-Output)

> **Source:** [arXiv:2603.15569](https://arxiv.org/abs/2603.15569) · [Together AI Blog](https://www.together.ai/blog/mamba-3) · [Goomba Lab Blog](https://goombalab.github.io/blog/2026/mamba3-part1/)  
> **Subtopic of:** [Mamba-3](./README.md)

---

## What Is MIMO?

MIMO (Multi-Input, Multi-Output) replaces the **vector outer-product** recurrence (`B_t x_t^⊤`) of standard SISO SSMs with a **matrix-multiplication** (`B_t X_t^⊤`), where both `B_t` and `X_t` are rank-R projections.

| Property | SISO | MIMO |
|---|---|---|
| `B_t` shape | `ℝ^{N×P}` | `ℝ^{N×R}` |
| `X_t` shape | `ℝ^{P×1}` | `ℝ^{P×R}` |
| Recurrence | `B_t x_t^⊤` (vector outer product) | `B_t X_t^⊤` (matrix multiply) |
| Arithmetic intensity | ~2.5 ops/byte (memory-bound) | `Θ(R)` — compute-efficient |
| SSMs modeled | 1 per layer | R SSMs in parallel |

> *"Replaces outer-product state update `B_t x_t^⊤` with matrix-multiplication `B_t X_t^⊤` (B_t ∈ ℝ^{N×R}, X_t ∈ ℝ^{P×R}). R× more FLOPs at similar decode latency (matmul uses fast tensor cores)."*  
> — [arXiv:2603.15569](https://arxiv.org/abs/2603.15569)

---

## The Hardware Utilization Problem MIMO Solves

During decoding, each timestep requires so little compute that **GPU tensor cores sit idle most of the time**:

- SISO: ~2.5 ops/byte vs. ~295 ops/byte capacity of bfloat16 matmul on H100
- Memory bandwidth is the bottleneck, not compute
- MIMO adds FLOPs that **overlap with existing memory I/O** — idle compute cycles get used without increasing wall-clock time

> *"SISO SSM decoding has arithmetic intensity ≈ 2.5 ops/byte (heavily memory-bound). MIMO increases FLOPs without increasing memory traffic or wall-clock latency."*  
> — [arXiv:2603.15569](https://arxiv.org/abs/2603.15569)

> *"Current linear models have been designed to use lots of GPU tensor cores for training, but during decoding, each timestep requires so little compute that the hardware remains cold most of the time."*  
> — [Goomba Lab Blog](https://goombalab.github.io/blog/2026/mamba3-part1/)

> *"Why training cost ↑ but inference stays same: Training is compute-bound (uses GPU tensor cores heavily); decoding is memory-bound (hardware idle most of the time). Adding FLOPs per timestep uses idle cores during decode → latency roughly constant."*  
> — [Goomba Lab Blog](https://goombalab.github.io/blog/2026/mamba3-part1/)

---

## Code Instantiation

```python
# From github.com/state-spaces/mamba (mamba3.py)
model = Mamba3(
    d_model=dim,
    d_state=128,
    headdim=64,
    is_mimo=True,        # Enable MIMO mode
    mimo_rank=4,         # MIMO rank R
    chunk_size=16,       # 64/mimo_rank if bf16, else 32/mimo_rank
)
```

> *"MIMO models are parameter-matched by reducing MLP inner dimension (e.g., 1.5B: 4096 → 3824, only 6.6% reduction)."*  
> — [Goomba Lab Blog](https://goombalab.github.io/blog/2026/mamba3-part1/)

---

## Benchmark Results

### Language Modeling (FineWeb-Edu, 1.5B scale)

| Model | FW-Edu Perplexity ↓ | LAMB Perplexity ↓ | Avg. Downstream Acc ↑ |
|---|---|---|---|
| Transformer | 10.51 | 11.1 | 55.4 |
| Gated DeltaNet | 10.45 | 10.9 | 55.8 |
| Mamba-2 | 10.47 | 12.0 | 55.7 |
| Mamba-3 SISO | 10.35 | 10.9 | 56.4 |
| **Mamba-3 MIMO (R=4)** | **10.24** | **10.2** | **57.6** |

> *"Mamba-3 MIMO boosts accuracy by >1 percentage point* over Mamba-3 SISO at 1B scale."  
> — [arXiv:2603.15569](https://arxiv.org/abs/2603.15569)

### State Size Efficiency

| Model | State Size | FineWeb-Edu Perplexity |
|---|---|---|
| Mamba-2 | 128 | 10.47 |
| **Mamba-3 MIMO** | **64** | **10.24** |

> *"Mamba-3 MIMO with a state size of 64 achieves comparable perplexity to Mamba-2 with a state size of 128."*  
> — [Plain English AI / MarkTechPost](https://www.marktechpost.com/2026/03/18/meet-mamba-3-a-new-state-space-model-frontier-with-2x-smaller-states-and-enhanced-mimo-decoding-hardware-efficiency/)

### Decode Latency (1.5B, H100-SXM 80GB, batch size 128)

| Model | n=512 | n=1024 | n=2048 | n=4096 | n=16384 |
|---|---|---|---|---|---|
| vLLM (Llama-3.2-1B) | 4.45 | 9.60 | 20.37 | 58.64 | 976.50 |
| Gated DeltaNet | 4.56 | 9.11 | 18.22 | 36.41 | 145.87 |
| Mamba-2 | 4.66 | 9.32 | 18.62 | 37.22 | 149.02 |
| **Mamba-3 SISO** | **4.39** | **8.78** | **17.57** | **35.11** | **140.61** |
| Mamba-3 MIMO (r=4) | 4.74 | 9.48 | 18.96 | 37.85 | 151.81 |

> *"Mamba-3 (SISO variant) achieves the fastest prefill + decode latency across all sequence lengths, outperforming Mamba-2, Gated DeltaNet, and even the Transformer with its highly optimized vLLM ecosystem."*  
> — [Together AI Blog](https://www.together.ai/blog/mamba-3)

**Key finding:** MIMO adds ~4× FLOPs but decode latency stays within ~5% of Mamba-2 because extra compute overlaps with memory I/O.

### MIMO Wall-Clock Overhead (1.5B, A100)

| MIMO Rank | Wallclock (μs) | Memory BW (GB/s) |
|---|---|---|
| 1 (SISO) | 239 | 2286 |
| 2 | 255 | 2156 |
| 4 | 260 | 2145 |
| 8 | 283 | 2030 |

> *"Decode latency is NOT very sensitive to r (memory-bound nature; compute overlaps with memory IO)."*  
> — [arXiv:2603.15569](https://arxiv.org/abs/2603.15569)

---

## How MIMO Relates to the Other Two Innovations

| Innovation | How it connects to MIMO |
|---|---|
| **Complex-Valued States** | MIMO with complex-valued B,C projections enables richer state dynamics per rank |
| **Exp.-Trapezoidal Discretization** | Three-term recurrence fits naturally with MIMO's expanded B,C projections |

---

## Key Quotes

> *"MIMO increases decoding FLOPs by up to 4x relative to Mamba-2 at fixed state size. Additional computation is overlaid with existing memory I/O required for state update → improves modeling quality and perplexity while maintaining similar wall-clock decode latency."*  
> — [MarkTechPost](https://www.marktechpost.com/2026/03/18/meet-mamba-3-a-new-state-space-model-frontier-with-2x-smaller-states-and-enhanced-mimo-decoding-hardware-efficiency/)

> *"MIMO improves retrieval without increasing state size."*  
> — [Goomba Lab Blog](https://goombalab.github.io/blog/2026/mamba3-part1/)

---

## 📚 References

- [arXiv:2603.15569](https://arxiv.org/abs/2603.15569) — primary paper
- [Together AI Blog](https://www.together.ai/blog/mamba-3)
- [Goomba Lab Blog Part 1](https://goombalab.github.io/blog/2026/mamba3-part1/)
- [MarkTechPost](https://www.marktechpost.com/2026/03/18/meet-mamba-3-a-new-state-space-model-frontier-with-2x-smaller-states-and-enhanced-mimo-decoding-hardware-efficiency/)
- [github.com/state-spaces/mamba](https://github.com/state-spaces/mamba)
