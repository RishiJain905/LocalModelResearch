# paTH Attention & GD-Attention (Ghost-Drift)

> **Research Method:** Direct HTTP fetching from arxiv.org/abs/ and GitHub API (April 2026).

---

## Research Notes

### paTH Attention

> **Status:** No public research repo or paper found under "paTH Attention" or "Page Token Attention" naming.

Based on the research landscape, "paTH Attention" likely refers to one of:

| Possible Meaning | Related Work | Notes |
|-----------------|--------------|-------|
| **Paged Attention** | [vLLM/vllm](https://github.com/vllm-project/vllm) ⭐77.3K | Most likely interpretation — KV cache paging |
| **Path Attention** | No direct match found | May be a specific variant |
| **PAX Attention** | No direct match found | May be internal/academic term |

> See: [PagedAttention (vLLM)](./PagedAttention-vLLM.md) for the most likely intended topic.

---

### GD-Attention (Ghost-Drift)

> **Status:** No public research repo or paper found under "GD-Attention" or "Ghost-Drift Attention" naming.

Possible interpretations:

| Possible Meaning | Related Work | Notes |
|-----------------|--------------|-------|
| **Gated Attention** | Gated Linear Attention | [mit-han-lab/duo-attention](https://github.com/mit-han-lab/duo-attention) ⭐536 |
| **Ghost Attention** | No direct match | May be related to attention sink concepts |
| **Gradient Drift** | No direct match | May be a quantization/optimization term |
| **Grouped-Drift** | No direct match | May be a specific variant |

> See: [DuoAttention](./DuoAttention.md) for a related gated/streaming separation approach.

---

## Summary

| Term | Likely Meaning | Best Match |
|------|---------------|------------|
| **paTH Attention** | Paged Attention | [PagedAttention (vLLM)](./PagedAttention-vLLM.md) |
| **GD-Attention** | Gated/Streaming Attention | [DuoAttention](./DuoAttention.md) |

If you have more context on these terms (paper titles, author names, or specific organizations), I can search more specifically.

---

## See Also

- [PagedAttention (vLLM)](./PagedAttention-vLLM.md)
- [DuoAttention](./DuoAttention.md)
- [StreamingLLM](./StreamingLLM-Attention-Sinks.md)
- [Flash Attention Family](./Flash-Attention-Family.md)
