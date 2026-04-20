# Ring Attention

> **Research Method:** Direct HTTP fetching from arxiv.org/abs/ and GitHub API (April 2026).
>
> **Paper:** [arXiv:2310.01889](https://arxiv.org/abs/2310.01889) · **GitHub:** [lhao499/llm_large_context](https://github.com/lhao499/llm_large_context) ⭐770

---

## Overview

> *"Ring Attention with Blockwise Transformers for Near-Infinite Context"* — [Liu et al., 2023](https://arxiv.org/abs/2310.01889)

Distributed attention mechanism that splits the attention computation across multiple devices in a ring topology, enabling near-infinite context processing (10M+ tokens).

---

## Key Paper

### "Ring Attention with Blockwise Transformers for Near-Infinite Context"

> **arXiv:** [2310.01889](https://arxiv.org/abs/2310.01889) | **GitHub:** [lhao499/llm_large_context](https://github.com/lhao499/llm_large_context) ⭐770

> *"Enables training and inference with context lengths of 10M+ tokens by distributing attention across devices."*

---

## How Ring Attention Works

### The Problem

Standard attention is O(N²) in sequence length — for 1M tokens, that's 10¹² operations per layer. No single device can hold this.

### The Solution: Ring Topology

```
Device Ring:
┌─────────┐    Qᵢ,Kᵢ,Vᵢ    ┌─────────┐
│ Device 0│ ←────────────→ │ Device 1│
│   ◯    │                 │   ◯    │
└─────────┘                 └─────────┘
     ↑                           ↓
     │ Qₙ,Kₙ,Vₙ                 │ Q₂,K₂,V₂
     └───────────────────────────┘
              (ring communication)
```

### Blockwise Computation

1. **Split by KV blocks** — Each device holds N/KV blocks (K devices)
2. **Ring pass** — Device i computes partial attention for its query block
3. **Receive & accumulate** — Receive KV block from previous device, accumulate
4. **Pass along** — Forward to next device
5. **Overlap** — Computation overlaps with communication

### Mathematical Flow

```
For device i processing query block Q_i:
1. Receive K_j, V_j from device (i-1) mod K
2. Compute partial: O_ij = softmax(Q_i @ K_j^T / √d) @ V_j
3. Accumulate: O_i += O_ij
4. Send K_i, V_i to device (i+1) mod K
5. Repeat K times → full attention output
```

---

## Key Features

| Feature | Description |
|---------|-------------|
| **Near-infinite context** | 10M+ token context lengths |
| **Device distribution** | Linear scaling with number of devices |
| **Exact attention** | No approximation, unlike sparse methods |
| **Distributed training** | Enables long-context LLM training |
| **Communication overlap** | Computation hides communication latency |

---

## Comparison with Alternatives

| Method | Context Limit | Memory/Device | Approximation | Hardware |
|--------|--------------|---------------|--------------|----------|
| **Ring Attention** | 10M+ tokens | O(N/K) | None | Multi-GPU |
| Flash Attention | 64K tokens | O(N) | None | Single GPU |
| Sparse attention | 1M tokens | O(N/K) | Yes | Single/Multi |
| StreamingLLM | Infinite | O(1) | Yes (window) | Single GPU |

---

## Related Repos

| Resource | Link | Stars | Description |
|----------|------|-------|-------------|
| Large Context Attention | [lhao499/llm_large_context](https://github.com/lhao499/llm_large_context) | ⭐770 | Main implementation |
| Triton Ring FlashAttn | [lyj20071013/Triton-Ring-FlashAttn](https://github.com/lyj20071013/Triton-Ring-FlashAttn) | ⭐0 | Educational Triton implementation |
| SP + Ring-Attention | [Jerry-ji0501/SP-Infer](https://github.com/Jerry-ji0501/SP-Infer) | ⭐0 | Distributed SP + Ring-Attention inference |

---

## See Also

- [README](./README.md) — Context Windows overview
- [PagedAttention](./PagedAttention-vLLM.md) — OS-style memory paging
- [StreamingLLM](./StreamingLLM-Attention-Sinks.md) — Constant memory streaming
- [Flash Attention](./Flash-Attention-Family.md) — Foundation for ring implementation
