# Token Merging (ToMe)

> **Paper:** [arXiv:2210.09461](https://arxiv.org/abs/2210.09461) (ICLR 2023 Oral, Top 5%) · **GitHub:** [facebookresearch/tome](https://github.com/facebookresearch/tome)  
> **Follow-ups:** [PiToMe NeurIPS 2024](https://arxiv.org/abs/2405.16148) · [AIM ICCV 2025](https://arxiv.org/abs/2412.03248) · [K-Token Merging arXiv:2604.15153](https://arxiv.org/abs/2604.15153)  
> **Blogs:** [Meta Research Blog](https://research.facebook.com/blog/2023/2/token-merging-your-vit-but-faster/) · [Hugging Face ToMe Guide](https://huggingface.co/docs/diffusers/optimization/tome)

---

## Overview

> *"We introduce Token Merging (ToMe), a simple method to increase the throughput of existing ViT models without needing to train."* — Bolya et al., [ICLR 2023](https://arxiv.org/abs/2210.09461)

Token Merging (ToMe) progressively combines similar tokens in transformer architectures using a **bipartite soft matching** algorithm. Unlike token pruning which deletes tokens and loses information, ToMe **preserves information** by merging tokens — achieving nearly the speed of pruning with better accuracy.

| Aspect | Details |
|--------|---------|
| **Method** | Bipartite Soft Matching — merges similar tokens without training |
| **Speedup** | 2–2.2× throughput for ViT-L/H with only 0.2–0.3% accuracy drop |
| **Key Insight** | Merging preserves information; pruning loses it |
| **Training-free** | Works off-the-shelf with pretrained models |
| **Applications** | Images, video, audio, Stable Diffusion, LLMs |

---

## Core Method: Bipartite Soft Matching

### The Algorithm (4 Steps)

```
1. PARTITION — Split tokens into sets A and B (alternating assignment)
2. DRAW EDGES — Each token in A connects to its most similar token in B (cosine similarity on keys K)
3. KEEP — Retain only the r most similar edges
4. MERGE — Connected tokens merge via weighted average by token size
```

### Why It Works

- Uses **keys (K)** from attention for similarity matching — captures what each token is "about"
- **Proportional attention** maintains information from merged tokens
- Nearly as fast as random pruning (constant runtime w.r.t. r)

### ToMe vs. Pruning

| Pruning | ToMe (Merging) |
|---------|----------------|
| Information loss limits reduction | Preserves information by combining |
| Requires re-training | Works **without training** |
| Can't speed up training | Cuts training time in half |
| Dynamic token counts prevent batching | Fixed token reduction enables batching |

---

## Key Results

### Vision Transformers (ICLR 2023)

| Application | Speed Increase | Quality Impact |
|-------------|---------------|----------------|
| ViT-L @ 512 | 2× throughput | 0.3% acc drop |
| ViT-H @ 518 | 2× throughput | 0.3% acc drop |
| ViT-L on video | 2.2× throughput | 0.2% acc drop |
| ViT-B on audio | 2× throughput | 0.4% mAP drop |

### Stable Diffusion (CVPR 2023 Workshop)

> *"Up to 2× faster inference, 5.7× less memory with 60% tokens merged."*

**Hugging Face Benchmarks:**

| GPU | Resolution | Batch | Vanilla (s) | ToMe (s) | Speed-up |
|-----|-----------|-------|-------------|----------|----------|
| A100 | 512 | 10 | 6.88 | 4.69 | +31.83% |
| A100 | 768 | 4 | OOM | 5.98 | Works! |
| A100 | 1024 | 1 | 6.40 | 2.81 | +56.09% |

**tomesd Speed vs Quality (SD v1.5, 4090 GPU):**

| Token Reduction | FID | Time (s/im) | Memory (GB) |
|-----------------|-----|-------------|-------------|
| 0% (baseline) | 33.12 | 3.09 | 3.41 |
| 50% | 33.02 | 1.65 (**1.87×**) | 0.89 (**3.83×**) |
| 60% | 33.37 | 1.52 (**2.03×**) | 0.60 (**5.68×**) |

---

## Follow-up Work

### PiToMe — Spectrum-Preserving Token Merging (NeurIPS 2024)

> **Paper:** [arXiv:2405.16148](https://arxiv.org/abs/2405.16148) · **GitHub:** [hchautran/PiToMe](https://github.com/hchautran/PiToMe)

**Problem:** BSM (Bipartite Soft Matching) has drawbacks — sensitivity to token-splitting, damage to informative tokens.

**Solution:** Uses **"energy score"** metric — large clusters = high-energy (merge candidates), small/unique clusters = low-energy (preserve).

### AIM — Adaptive Inference for Multi-Modal LLMs (ICCV 2025)

> **Paper:** [arXiv:2412.03248](https://arxiv.org/abs/2412.03248) · **GitHub:** [LaVi-Lab/AIM](https://github.com/LaVi-Lab/AIM)

**Key Result:** 7-fold FLOPs reduction, +4.6 on MLVU benchmark.

**Components:** Token merging before LLMs + progressive token pruning within LLM layers.

### K-Token Merging for LLMs (arXiv 2026)

> **Paper:** [arXiv:2604.15153](https://arxiv.org/abs/2604.15153)

**Focus:** Compressing sequences in latent embedding space for LLMs by merging groups of tokens into single embeddings.

---

## Usage

### Vision Transformers (tome)

```python
import tome
model = timm.create_model('vit_base_patch16_224', pretrained=True)
tome.patch(model)
# Model is now accelerated with token merging
```

### Stable Diffusion (tomesd)

```python
import tomesd
tomesd.apply_patch(model, ratio=0.5)  # Merge 50% of tokens
```

### Hugging Face Diffusers

```python
from diffusers import StableDiffusionPipeline
pipe = StableDiffusionPipeline.from_pretrained("runwayml/stable-diffusion-v1-5")
pipe = tome.apply(pipe, ratio=0.5)
```

---

## Papers Table

| Paper | Year | Venue | URL | Key Contribution |
|-------|------|-------|-----|-----------------|
| **ToMe (original)** | 2022 | ICLR 2023 | [arXiv](https://arxiv.org/abs/2210.09461) | Bipartite soft matching for training-free token merging |
| **ToMe for SD** | 2023 | CVPR Workshop | [arXiv](https://arxiv.org/abs/2303.17604) | 2× speedup, 5.7× memory reduction for Stable Diffusion |
| **PiToMe** | 2024 | NeurIPS 2024 | [arXiv](https://arxiv.org/abs/2405.16148) | Spectrum-preserving merging via energy score |
| **AIM** | 2024 | ICCV 2025 | [arXiv](https://arxiv.org/abs/2412.03248) | 7× FLOPs reduction for multi-modal LLMs |
| **K-Token Merging** | 2026 | — | [arXiv](https://arxiv.org/abs/2604.15153) | Latent embedding space merging for LLMs |

---

## GitHub Repos

| Repo | Description |
|------|-------------|
| [facebookresearch/tome](https://github.com/facebookresearch/tome) | Original ToMe for Vision Transformers |
| [dbolya/tomesd](https://github.com/dbolya/tomesd) | ToMe for Stable Diffusion |
| [hchautran/PiToMe](https://github.com/hchautran/PiToMe) | Spectrum-Preserving Token Merging |
| [LaVi-Lab/AIM](https://github.com/LaVi-Lab/AIM) | Adaptive Inference for Multi-Modal LLMs |
| [hutaihang/ToMe](https://github.com/hutaihang/ToMe) | ToMe for semantic binding in text-to-image |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Problem solved** | Accelerates ViT/LLM inference without training by merging redundant tokens |
| **Key innovation** | Bipartite soft matching — preserves information unlike pruning |
| **Speedup** | 2× throughput for ViT, 2× for Stable Diffusion, up to 5.7× memory reduction |
| **Best for** | Scenarios where tokens are spatially/structurally redundant (images, video, diffusion) |
| **Limitation** | Works best when tokens have spatial structure; less obvious for pure text |

---

*Last updated: 2026-04-19*
