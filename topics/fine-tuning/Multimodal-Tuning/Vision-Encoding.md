# Vision-Encoding

## 📌 Overview

| Field | Value |
|-------|-------|
| **Full Name** | Vision Encoding for Multimodal LLMs |
| **Role** | Converts images into token representations the LLM can process |
| **Key Architectures** | ViT (Vision Transformer), CLIP, DINO, SigLIP |
| **Core Debate** | CLIP vs DINO — which produces better visual representations for MLLMs? |

> "A visual tokenizer is a mechanism that maps input visual signals (such as images or videos) into a set of compact and structured visual units (tokens), which may be continuous vectors, discrete indices, or a hybrid of both."

— [Awesome-Visual-Tokenizers](https://github.com/lavinal712/Awesome-Visual-Tokenizers)

---

## Definitions

### Visual Tokenizer

> "A core requirement of a visual tokenizer is that the generated visual units must possess sufficient representational capacity to enable high-quality reconstruction of the original visual input through a corresponding decoder or generator."

— [Awesome-Visual-Tokenizers](https://github.com/lavinal712/Awesome-Visual-Tokenizers)

### Vision Transformer (ViT) Patch Encoding

> "An image is split into smaller fixed-sized patches which are treated as a sequence of tokens, similar to words for NLP tasks."

— [Dosovitskiy et al., ViT (ICLR 2021)](https://arxiv.org/abs/2010.11929)

### CLIP Latent Space Property

> "CLIP's latent tokens have a cross-modal / synaesthetic ability — they are already mostly aligned with their captions. Because the text tokens bypass CLIP's text encoder, the visual tokens must be re-aligned with the LLM's embedding space."

— [Hugging Face Blog](https://huggingface.co/blog/gigant/vlm-design)

---

## Key Sources

### CLIP vs DINO Studies

#### From CLIP to DINO (ICLR 2024)

| Field | Value |
|-------|-------|
| Paper | [OpenReview](https://openreview.net/forum?id=syoLhUJmth) |
| Authors | Dongshiang Jiang, Yuchen Liu, Songlin Liu, Xiaopeng Zhang, Jin Li, Hongkai Xiong, Qi Tian |
| Date | September 2023 → February 2024 (revised) |

> "Surprisingly, the vision-only model DINO, which is not pretrained with text-image alignment, demonstrates promising performance as a visual branch within MLLMs. By simply equipping it with an MLP layer for alignment, DINO surpasses CLIP in fine-grained related perception tasks."

**Core Finding**: Combining CLIP and DINO via multi-level feature merging (COMM) improves visual capabilities across image captioning, VQA, visual grounding, and object hallucination benchmarks.

#### Data or Language Supervision: What Makes CLIP Better than DINO? (EMNLP 2025)

| Field | Value |
|-------|-------|
| Paper | [arXiv:2510.11835](https://arxiv.org/abs/2510.11835) |
| Date | October 2025 |

> "When integrated into VLMs and evaluated on 20 VQA benchmarks, CLIP excels at text-intensive tasks, while DINO slightly outperforms on vision-centric ones."

> "Language supervision primarily benefits vision encoders by fostering high-level semantic understanding, improving fine-grained classification, and significantly enhancing performance on text-intensive tasks within VLMs."

### Perception Encoder (Meta AI / FAIR, April 2025)

| Field | Value |
|-------|-------|
| Paper | [Meta AI Research](https://ai.meta.com/research/publications/perception-encoder-the-best-visual-embeddings-are-not-at-the-output-of-the-network/) |
| Authors | Daniel Bolya, Po-Yao Huang, Peize Sun et al. |

> "Surprisingly, after scaling our carefully tuned image pretraining recipe and refining with our robust video data engine, we find that contrastive vision-language training alone can produce strong, general embeddings for all of these downstream tasks. There is only one caveat: these embeddings are hidden within the intermediate layers of the network."

**Core Finding**: State-of-the-art vision representations for classification, detection, depth, tracking, and VQA are found in **INTERMEDIATE layers**, not the final output. PE family includes Core, Lang, Spatial, and AV variants.

### OpenVision (ICCV 2025)

| Field | Value |
|-------|-------|
| Paper | [arXiv:2505.04601](https://arxiv.org/abs/2505.04601) |
| Project Page | [UCSC-VLAA/OpenVision](https://ucsc-vlaa.github.io/OpenVision/) |
| Authors | Xianhang Li, Yanqing Liu, Haoqin Tu, Hongru Zhu, Cihang Xie (UCSC) |

> "This paper fills this gap with OpenVision, a fully-open, cost-effective family of vision encoders that match or surpass the performance of OpenAI's CLIP when integrated into multimodal frameworks like LLaVA."

**Model Range**: 5.9M to 632.1M parameters; releases all training data, recipes, and checkpoints publicly.

### Visual Autoregressive Modeling — VAR (NeurIPS 2024 Oral)

| Field | Value |
|-------|-------|
| Paper | [VAR (OpenReview)](https://openreview.net/forum?id=gojL67CfS8) |
| Authors | Keyu Tian, Yi Jiang, Zehuan Ying, Jingdong Chen (FoundationVision) |

> "We present Visual AutoRegressive modeling (VAR), a new generation paradigm that redefines the autoregressive learning on images as coarse-to-fine next-scale prediction, diverging from the standard raster-scan next-token prediction."

**Core Finding**: VAR is the **FIRST GPT-style AR model to surpass diffusion transformers** in image generation (FID: 1.80 vs 18.65 baseline AR). Exhibits clean power-law scaling with near-perfect correlation coefficient of -0.998. ~20x faster inference.

### Fuyu-8B (Adept AI)

| Field | Value |
|-------|-------|
| Blog | [Adept AI](https://www.adept.ai/blog/fuyu-8b/) |

> "Fuyu is a vanilla decoder-only transformer with no specialized image encoder. Image patches are linearly projected directly into the first layer."

**Core Finding**: Eliminates separate vision encoder entirely — feeds raw image patches directly into the language model. Handles arbitrary image resolutions naturally.

### Exploring How Generative MLLMs Perceive More Than CLIP

| Field | Value |
|-------|-------|
| Paper | [arXiv:2411.05195](https://arxiv.org/html/2411.05195v3) |
| Authors | Siting Li, Pang Wei Koh, Simon Shaolei Du (UW) |

> "The encoder gathers query-relevant visual information, while CLIP fails to extract it."

> "Using ALL patch tokens (not just [CLS]) and question conditioning are key."

---

## Repositories & Tools

### Visual Tokenizers

| Repository | URL |
|------------|-----|
| Awesome-Visual-Tokenizers | [GitHub](https://github.com/lavinal712/Awesome-Visual-Tokenizers) |
| VQGAN | [GitHub](https://github.com/Westlake-AI/VQGAN) |
| OmniTokenizer | [GitHub](https://github.com/FoundationVision/OmniTokenizer) |

### Vision Encoder Models

| Model | URL | Description |
|-------|-----|-------------|
| Perception Encoder | [GitHub](https://github.com/facebookresearch/perception_models) | Meta's intermediate-layer embeddings |
| OpenVision | [GitHub](https://github.com/UCSC-VLAA/OpenVision) | Fully-open CLIP-replacement |
| CLIP | [arXiv:2103.00020](https://arxiv.org/abs/2103.00020) | OpenAI's contrastive V&L |
| SigLIP | [arXiv:2303.15343](https://arxiv.org/abs/2303.15343) | Google's CLIP alternative |
| DINOv2 | [arXiv:2304.07193](https://arxiv.org/abs/2304.07193) | Meta's self-supervised ViT |
| VAR | [GitHub](https://github.com/FoundationVision/VAR) | Visual autoregressive modeling |

### Evaluation Benchmarks

| Benchmark | URL |
|-----------|-----|
| MMBench | [OpenReview](https://openreview.net/forum?id=BfMQIJ0nLc) |
| MME Benchmark | [GitHub](https://github.com/aiben-ch/LMM-Evaluation-Survey) |

---

## Benchmark Results

### CLIP vs DINO (Same Frozen Encoder, EMNLP 2025)

| Task | LLaVA-CLIP | LLaVA-DINO |
|------|------------|------------|
| OCRVQA/TextVQA | **47.5%** | 40.0% |
| Stanford Cars | **74.7%** | 54.1% |

CLIP excels at text-intensive tasks (+7.5pp). DINO slightly better on vision-centric tasks.

### Key Finding: [CLS] vs All Patch Tokens

Using only [CLS] token collapses spatial reasoning (68.9% → 8.7% on Left/Right pairs). All patch tokens essential.

### Spatial Reasoning: CLIP vs LLaVA (Same Frozen Encoder)

| Benchmark | CLIP-ViT-L/14-336px | LLaVA-1.5-7B |
|-----------|---------------------|--------------|
| WhatsUp A Left/Right | 1.9% | **93.2%** |
| WhatsUp B Left/Right | 10.8% | **97.1%** |
| GQA-spatial Two-obj. | 49.1% | **95.2%** |
| COCO-spatial Two-obj. | 51.1% | **88.9%** |

### Perception Encoder Performance

| Model | Doc VQA | TextVQA |
|-------|---------|---------|
| PLM-3B + PE-Lang | 93.8 | 84.3 |
| PLM-8B + PE-Lang | **94.6** | **86.5** |

### OpenVision Model Zoo

| Model | Params | Top-1 (IN-1K) |
|-------|--------|---------------|
| ViT-Tiny | 5M | 49.6% |
| ViT-Small | 22M | 65.9% |
| ViT-Base | 86M | 73.9% |
| ViT-Large | 307M | 78.5% |
| ViT-Huge | 632M | 80.3% |

### VAR Scaling Laws (vs Diffusion)

| Metric | Baseline AR | VAR |
|--------|------------|-----|
| FID | 18.65 | **1.80** |
| IS | 80.4 | **356.4** |
| Inference Speed | 1x | **20x faster** |

---

## Vision Encoders Used in Major MLLMs

| Model | Vision Encoder |
|-------|---------------|
| LLaVA | CLIP-ViT-L-14-336 |
| LLaVA-NeXT | clip-vit-large-patch14-336 |
| DeepSeek-VL | ViT-SO400M-14-SigLIP-384 |
| moondream2 | ViT-SO400M-14-SigLIP-384 |

---

## Key Debates

### 1. CLIP vs DINO
CLIP excels at text-intensive tasks. DINO better for fine-grained perception. Language supervision (not dataset scale) is key factor.

### 2. [CLS] vs All Patch Tokens
Only [CLS] token collapses spatial reasoning. All 576 patch tokens essential for fine-grained visual reasoning.

### 3. Intermediate vs Final Layer Representations
Perception Encoder (Meta 2025): Best embeddings are in INTERMEDIATE layers, not final output. Requires specialized alignment adapters.

### 4. Encoder-Free vs Encoder-Based
- **Encoder-based**: CLIP-style (LLaVA) — pretrained vision + projector
- **Encoder-free**: Fuyu — raw patches directly into LLM, simpler but needs massive training data

### 5. Proprietary vs Fully-Open Encoders
OpenVision (ICCV 2025): Fully-open encoders (5.9M–632.1M params) match or surpass OpenAI CLIP and SigLIP.

---

## Related Topics

- [VLM](./VLM.md) — Vision-Language Models that use vision encoders
- [Projector-Alignment](./Projector-Alignment.md) — Projectors that connect vision encoders to LLMs
- [CLIP](../weight-quantization/CLIP.md) — Contrastive Language-Image Pretraining foundation
- [SigLIP](../model-cards/SigLIP.md) — Google's CLIP alternative

---

## References

- [ViT — An Image is Worth 16x16 Words (arXiv:2010.11929)](https://arxiv.org/abs/2010.11929)
- [From CLIP to DINO (ICLR 2024)](https://openreview.net/forum?id=syoLhUJmth)
- [CLIP vs DINO (EMNLP 2025, arXiv:2510.11835)](https://arxiv.org/abs/2510.11835)
- [Perception Encoder (Meta AI, 2025)](https://ai.meta.com/research/publications/perception-encoder-the-best-visual-embeddings-are-not-at-the-output-of-the-network/)
- [OpenVision (arXiv:2505.04601)](https://arxiv.org/abs/2505.04601)
- [VAR (NeurIPS 2024, OpenReview)](https://openreview.net/forum?id=gojL67CfS8)
- [Fuyu-8B (Adept AI)](https://www.adept.ai/blog/fuyu-8b/)
- [Hugging Face — VLM Design Choices](https://huggingface.co/blog/gigant/vlm-design)
