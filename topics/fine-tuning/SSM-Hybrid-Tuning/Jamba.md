# Jamba: Hybrid Transformer-Mamba Architecture

## 📌 Overview

**Jamba** is a production-grade hybrid Transformer-Mamba-MoE architecture developed by [AI21 Labs](https://www.ai21.com), representing the first hybrid SSM-Transformer model at scale. It interleaves Mamba layers (for efficient long-context processing) with Transformer layers (for in-context learning and retrieval) and applies Mixture-of-Experts selectively.

> *"Jamba is a hybrid Transformer-Mamba language model — the first production-grade hybrid SSM-Transformer model at scale."*
> — [AI21 Blog](https://www.ai21.com/blog/announcing-jamba/)

---

## Architecture

### Jamba Block Structure

| Component | Configuration |
|-----------|--------------|
| **Layers per block** | 8 total: 1 Attention + 7 Mamba (1:7 ratio) |
| **MoE** | 16 experts, top-2 selection, every 2 layers |
| **Positional embeddings** | None required — Mamba layers provide implicit positional info |

> *"No positional embeddings required — Mamba layers provide implicit positional information."*
> — [arXiv:2403.19887](https://arxiv.org/abs/2403.19887)

### Available Models

| Model | Total Params | Active Params | Context Length |
|-------|-------------|---------------|----------------|
| **Jamba (original)** | 52B | 12B | 256K |
| **Jamba-1.5-Mini** | 52B | 12B | 256K |
| **Jamba-1.5-Large** | 398B | 94B | 256K |

> *"Effective context length of 256K tokens."*
> — [AI21 Blog](https://www.ai21.com/blog/announcing-jamba/)

---

## Why Hybrid?

### Ablation: Pure Attention vs Pure Mamba vs Hybrid

| Model | IMDB | QuAC | NarrativeQA |
|-------|------|------|-------------|
| Pure Attention | 84.1 | 27.9 | 45.8 |
| Pure Mamba | 48.8 | 20.2 | 27.7 |
| **Hybrid Jamba** | **90.9** | **26.6** | **43.7** |

> *"Pure Mamba struggles with datasets requiring format adherence (IMDB, QuAC, NarrativeQA). Hybrid Jamba outperforms both pure attention and pure Mamba models."*
> — [arXiv:2403.19887](https://arxiv.org/abs/2403.19887)

### Why Attention Layers Matter

> *"Attention layers act as 'induction heads' enabling in-context learning — even with just 1 attention layer out of 8."*
> — [arXiv:2403.19887](https://arxiv.org/abs/2403.19887)

### Mamba-1 vs Mamba-2 in Hybrid

> *"We found that in a hybrid architecture, the Mamba-1-Attention combination works better than Mamba-2-Attention."*
> — [arXiv:2408.12570](https://arxiv.org/abs/2408.12570)

> *"Mamba-2's larger state size advantages are diminished when interleaved with full attention layers."*
> — [arXiv:2408.12570](https://arxiv.org/abs/2408.12570)

### Retrieval and SSM Layers

> *"SSM layers do not contribute to retrieval — even fuzzy memory observed in pure SSMs is absent. Retrieval depends exclusively on self-attention layers."*
> — [arXiv:2403.19887](https://arxiv.org/abs/2403.19887)

---

## Efficiency

### KV Cache Comparison

| Model | KV Cache Size | Context on Single GPU |
|-------|--------------|----------------------|
| **Jamba** | **8× smaller** (4GB) | 140K tokens on 80GB A100 |
| Mixtral-8x7B | 32GB | — |

> *"8x smaller KV cache than vanilla Transformers (4GB vs 32GB for Mixtral at 256K)."*
> — [AI21 Blog](https://www.ai21.com/blog/announcing-jamba/)

### Throughput

> *"3x faster than Mixtral-8x7B at 128K tokens."*
> — [AI21 Blog](https://www.ai21.com/blog/announcing-jamba/)

> *"3.73× higher throughput compared to Transformers with grouped-query attention for 128K prompts."*
> — [Samba Paper](https://openreview.net/forum?id=bIlnpVM4bc)

---

## Fine-Tuning Jamba

### Recommended LoRA Target Modules

```
embed_tokens,
x_proj, in_proj, out_proj,  # Mamba layers
gate_proj, up_proj, down_proj,  # MLP layers  
q_proj, k_proj, v_proj, o_proj  # Attention layers
```

### What Works / What Doesn't

| Method | Effectiveness | Notes |
|--------|---------------|-------|
| **LoRA on linear projections** | ✅ Effective | q_proj, k_proj, v_proj, o_proj, MLP layers |
| **LoRA on SSM modules (A/B)** | ⚠️ Less effective | Not recommended |
| **Input-injection (prefix-tuning)** | ❌ Fails | "Expressivity reduces to tuning only initial hidden state" |

> *"LoRA on linear projections outperforms LoRA on SSM modules."*
> — [arXiv:2410.09016](https://arxiv.org/abs/2410.09016)

> *"Input-injection methods (prefix-tuning) FAIL on SSM-based models — expressivity reduces to tuning only initial hidden state."*
> — [arXiv:2410.09016](https://arxiv.org/abs/2410.09016)

### Optimal Approach

> *"Optimal approach: LoRA on linear projections + SDT (Sparse Dimension Tuning) on SSM modules."*
> — [arXiv:2410.09016](https://arxiv.org/abs/2410.09016)

---

## Resources

| Resource | URL |
|----------|-----|
| **AI21 Blog** | [ai21.com/blog/announcing-jamba](https://www.ai21.com/blog/announcing-jamba/) |
| **AI21 Fine-tuning Docs** | [docs.ai21.com/docs/fine-tuning](https://docs.ai21.com/docs/fine-tuning) |
| **HuggingFace** | [huggingface.co/ai21labs/AI21-Jamba-Mini-1.5](https://huggingface.co/ai21labs/AI21-Jamba-Mini-1.5) |

---

## Sources

| Type | Citation |
|------|----------|
| **Paper** | [arXiv:2403.19887 — Jamba](https://arxiv.org/abs/2403.19887) |
| **Paper** | [arXiv:2408.12570 — Jamba-1.5](https://arxiv.org/abs/2408.12570) |
| **Blog** | [AI21 Blog — Jamba Announcement](https://www.ai21.com/blog/announcing-jamba/) |
| **Paper** | [arXiv:2410.09016 — PEFT for SSMs (ICML 2025)](https://arxiv.org/abs/2410.09016) |
| **Docs** | [AI21 Fine-tuning Documentation](https://docs.ai21.com/docs/fine-tuning) |

---

## Related Topics

- [Mamba-3](./Mamba-3.md) — Mamba SSM architecture lineage
- [State-Optimization](./State-Optimization.md) — SSM state compression and hybrid optimization
