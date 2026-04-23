# rsLoRA (Rank-Stabilized LoRA)

## 📌 Overview

| Field | Value |
|-------|-------|
| **Full Name** | Rank-Stabilized LoRA |
| **Key Innovation** | Changes scaling factor from α/r → α/√r |
| **Primary Benefit** | Enables stable training at high ranks (256–2048) |
| **Paper** | [arXiv:2312.03732](https://arxiv.org/abs/2312.03732) |
| **Author** | Damjan Kalajdzievski (Tenyx), December 2023 |
| **Implementation** | `use_rslora=True` in Hugging Face PEFT `LoraConfig` |

> "In this work, we study the impact of the scaling factor on the learning process and prove that LoRA adapters should be divided by a factor of the square root of their rank rather than their rank."

— [Kalajdzievski, arXiv:2312.03732](https://arxiv.org/abs/2312.03732)

---

## Definitions

### Standard LoRA Scaling

```
γ_r = α / r
h = W₀x + (α/r)BAx
```

Standard LoRA saturates at r=4–32 because α/r becomes too small at higher ranks, stunting gradient updates.

### rsLoRA Scaling

```
γ_r = α / √r
```

> "Only 1/√r scaling ensures stability for arbitrary r, preventing both collapse and explosion."

— [Kalajdzievski, arXiv:2312.03732](https://arxiv.org/abs/2312.03732)

**Theorem 3.2**: All LoRA adapters are rank-stabilized if and only if γᵣ ∈ Θ(1/√r).

### Why It Matters

| Rank | Standard (α/r) | rsLoRA (α/√r) |
|------|---------------|---------------|
| 16 | 0.0625α | 0.25α |
| 64 | 0.0156α | 0.125α |
| 256 | 0.0039α | 0.0625α |
| 2048 | 0.0005α | 0.022α |

At high rank, standard LoRA scaling becomes vanishingly small. rsLoRA maintains meaningful update magnitude.

---

## Key Sources

### Original Paper

| Field | Value |
|-------|-------|
| Paper | [A Rank Stabilization Scaling Factor for Fine-Tuning with LoRA](https://arxiv.org/abs/2312.03732) |
| Author | Damjan Kalajdzievski (Tenyx) |
| Citations | ~214 |

> "The best LoRA rank-4 result (perplexity 1.863) cannot match rsLoRA rank-2048 (perplexity 1.836)."

### Hugging Face Blog

| URL | [huggingface.co/blog/damjan-k/rslora](https://huggingface.co/blog/damjan-k/rslora) |
|------|------|

MT-Bench Results:

| Model | MT-Bench Average |
|-------|-----------------|
| Base model | 7.790625 |
| LoRA rank 16 | 7.93125 |
| LoRA rank 256 | 7.9625 |
| **rsLoRA rank 256** | **8.0875** |

### LoRA-GA (NeurIPS 2024)

| Paper | [NeurIPS 2024](https://proceedings.neurips.cc/paper_files/paper/2024/file/62c4718cc334f6a0a62fb81c4a2095a1-Paper-Conference.pdf) |

Performance (LR=1e-5):

| Method | MT-Bench | GSM8K | HumanEval |
|--------|----------|-------|-----------|
| Full FT | 5.63 | 43.95 | 15.97 |
| LoRA | 5.53 | 35.73 | 14.35 |
| rsLoRA | 5.60 | 40.56 | 15.69 |
| LoRA-GA | 5.82 | 51.33 | 17.64 |

rsLoRA consistently outperforms standard LoRA on GSM8K and HumanEval.

---

## Implementation

### Hugging Face PEFT

```python
from peft import LoraConfig, get_peft_model

lora_config = LoraConfig(
    r=256,
    lora_alpha=16,
    use_rslora=True,
    target_modules=["q_proj", "v_proj"]
)
peft_model = get_peft_model(base_model, lora_config)
```

> "If confused about alpha for rsLoRA, just leave default and sweep rank."

— BenjaminBossan (PEFT Maintainer)

### Recommended Configuration

| Model Size | Suggested Rank | Alpha | Notes |
|-----------|---------------|-------|-------|
| ~7B (dim=4096) | 256 (up to 2048) | Fixed at 8 or 16 | Do NOT scale alpha with rank |

**Critical**: Do NOT scale alpha proportionally with rank in rsLoRA. Fix alpha at 8–16 and sweep rank.

---

## Repositories

| Repository | URL |
|------------|-----|
| Hugging Face PEFT | [huggingface.co/docs/peft](https://huggingface.co/docs/peft/package_reference/lora) |
| rsLoRA Blog Repo | [github.com/Damjan-Kalajdzievski/rsLoRA_Blog](https://github.com/Damjan-Kalajdzievski/rsLoRA_Blog) |
| Kaitchup rsQLoRA | [kaitchup.substack.com](https://kaitchup.substack.com/p/rsqlora-fine-tune-llama-3-with-higher) |

---

## Controversies & Bugs

### Alpha Scaling Error (PEFT Discussion #1387)

> "Do NOT modify alpha for rsLoRA. alpha=8 for rank=256 with use_rslora=True is correct."

User scaled alpha proportionally with rank and got ~1% worse results.

### Open Bug: LoraLayer.set_scale (PEFT Issue #2733)

rsLoRA scale incorrectly computed in `set_scale` when loading adapters — currently open.

### Unsloth Issue #2531

`use_rslora` flag being ignored in `FastModel.get_peft_model`.

### Limitations

> "Depending on finetuning dataset, there may be forgetting/overfitting that saturates at large rank."

— Author note

---

## Summary Data Points

| Configuration | Metric | Value |
|--------------|--------|-------|
| LoRA r=4 | perplexity | 1.863 |
| rsLoRA r=2048 | perplexity | **1.836** |
| LoRA r=256, MT-Bench | avg | 7.9625 |
| rsLoRA r=256, MT-Bench | avg | **8.0875** |
| rsLoRA r=256, GSM8K | % | 40.56 (vs LoRA 35.73) |

---

## Related Topics

- [Rank-Alpha-Scaling.md](./Rank-Alpha-Scaling.md) — α/r vs α/√r theory and scaling laws
- [Paged-Optimizers.md](./Paged-Optimizers.md) — Memory optimization for training
- [rsLoRA vs LoRA](../PEFT-Consumer-VRAM/LoRA.md) — Comparison with standard LoRA

---

## References

- [Kalajdzievski — rsLoRA (arXiv:2312.03732)](https://arxiv.org/abs/2312.03732)
- [Hugging Face Blog — rsLoRA](https://huggingface.co/blog/damjan-k/rslora)
- [BayJarvis — rsLoRA Summary](https://blog.bayjarvis.com/paper/a-rank-stabilization-scaling-factor-for-fine-tuning-with-lora)
- [PEFT Discussion #1387](https://github.com/huggingface/peft/discussions/1387)
- [PEFT Issue #2733](https://github.com/huggingface/peft/issues/2733)
