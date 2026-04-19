# Vector Quantization for Robustness (Q-MLLM)

> **Q-MLLM Paper:** [arXiv:2511.16229](https://arxiv.org/abs/2511.16229) (NDSS 2026) · **Code:** [github.com/Amadeuszhao/QMLLM](https://github.com/Amadeuszhao/QMLLM)  
> **Survey:** [arXiv:2507.22920](https://arxiv.org/abs/2507.22920) (Discrete Tokenization for MLLMs) · [jindongli-Ai/LLM-Discrete-Tokenization-Survey](https://github.com/jindongli-Ai/LLM-Discrete-Tokenization-Survey)  
> **Related:** [lucidrains/vector-quantize-pytorch](https://github.com/lucidrains/vector-quantize-pytorch)

---

## Overview

> *"Q-MLLM: Integrating two-level vector quantization to create a discrete bottleneck against adversarial attacks while preserving multimodal reasoning capabilities."* — Q-MLLM (arXiv:2511.16229)

**Vector Quantization (VQ)** in multimodal LLMs serves a dual purpose: compressing visual representations and creating **non-differentiable discrete bottlenecks** that block gradient-based adversarial attacks.

---

## Q-MLLM: Vector Quantization for Multimodal LLM Security

> **Paper:** [arXiv:2511.16229](https://arxiv.org/abs/2511.16229) (NDSS 2026)  
> **Code:** [github.com/Amadeuszhao/QMLLM](https://github.com/Amadeuszhao/QMLLM)  
> **Models:** `vincentchao/qmllm_unitok` (v2 with UniTok)

### Architecture

```
Input Image → Encoder → [Semantic-Level VQ] → [Patch-Level VQ] → Discrete Tokens → LLM
                                          K=128              P=16,000
              Creates non-differentiable bottleneck ← blocks gradient attacks
```

| Quantization Level | Codebook Size | Purpose |
|-------------------|--------------|---------|
| **Semantic-level** | K=128 | Global semantic compression |
| **Patch-level** | P=16,000 | Fine-grained visual representation |

### Results

| Defense Method | Jailbreak DSR↑ | Toxic Image DSR↑ |
|---------------|----------------|------------------|
| LLaVA-1.5 | 49.5% | 1.0% |
| ETA | 92.1% | 54.7% |
| **Q-MLLM-7B** | **98.4%** | **75.9%** |

> **DSR (Defense Success Rate)** = percentage of adversarial examples successfully blocked.

---

## Why VQ Creates Robustness

### The Adversarial Attack Pathway

```
Continuous visual features → gradient-based optimization → adversarial perturbation
     ↓
VQ discretization breaks this pathway
     ↓
Small perturbation ≠ different codebook index
```

### Key Mechanism

> *"Nearest-neighbor quantization is piecewise constant — small shifts at cell boundaries alter codebook indices. The adversarial perturbation maximizes changes to pre-quantization embeddings."* — "On the Adversarial Robustness of Discrete Image Tokenizers" (arXiv:2602.18252)

| Property | Effect |
|----------|--------|
| **Non-differentiable** | Gradient-based optimizers cannot backprop through VQ |
| **Discrete bottleneck** | Perturbations don't translate to different tokens |
| **Codebook discretization** | Error confined to nearest neighbor |

---

## Discrete Tokenization Methods

### Taxonomy from Survey

> **Survey:** [arXiv:2507.22920](https://arxiv.org/abs/2507.22920) · [jindongli-Ai/LLM-Discrete-Tokenization-Survey](https://github.com/jindongli-Ai/LLM-Discrete-Tokenization-Survey)

| Method | Key Feature |
|--------|------------|
| **VQ-VAE / VQGAN** | Standard discrete tokenization via codebook lookup |
| **Residual VQ (RVQ)** | Multiple quantizers recursively quantize residuals |
| **Finite Scalar Quantization (FSQ)** | Eliminates codebook, uses scalar rounding |
| **Look-up Free Quantization (LFQ)** | Binary latents without embedding lookup (MAGVIT-v2) |
| **SimVQ** | Frozen codebook with linear projection |
| **VAEVQ** | Variational modeling for discrete visual tokenization (AAAI 2026) |

### Methods by Application

| Application | Method |
|------------|--------|
| **Image generation** | VQ-Diffusion (CVPR 2022), DALL-E style |
| **Video LLMs** | VQToken (NeurIPS 2025) — 0.07% token reduction |
| **Multimodal security** | Q-MLLM — adversarial defense |
| **Object-centric** | Slot-MLLM — Slot Attention + Residual VQ |
| **Unified codebook** | UniCode (ECCV 2024) — visual + text tokenization |

---

## VQToken: Extreme Token Reduction for Video LLMs

> **Paper:** [arXiv:2503.16980](https://arxiv.org/abs/2503.16980) (NeurIPS 2025)  
> **Code:** [github.com/Hai-chao-Zhang/VQToken](https://github.com/Hai-chao-Zhang/VQToken)

**Key contribution:** Extreme token reduction to **0.07%** of original video tokens for video LLMs.

> *"VQToken enables video LLMs to process long videos by compressing 14,000 tokens per frame to ~10 tokens."*

---

## VAEVQ: Variational Modeling for Discrete Tokenization

> **Paper:** [arXiv:2511.06863](https://arxiv.org/abs/2511.06863) (AAAI 2026 Oral)  
> **Code:** [github.com/script-Yang/VAEVQ](https://github.com/script-Yang/VAEVQ)

**Key contribution:** Variational approach to VQ — models token distribution rather than just point estimates.

---

## LG-VQ: Language-Guided Codebook Learning

> **Paper:** [arXiv:2405.14206](https://arxiv.org/abs/2405.14206) (NeurIPS 2024)

**Key contribution:** Uses language supervision to guide codebook learning for multimodal tasks.

---

## Discrete Tokenizer Robustness Study

> **Paper:** [arXiv:2602.18252](https://arxiv.org/abs/2602.18252)

**First systematic study** of adversarial vulnerability in discrete tokenizers (TiTok, UniTok, FlexTok):

| Finding | Detail |
|---------|--------|
| **Attack target** | Pre-quantization embeddings |
| **Attack method** | Maximizes perturbation to alter codebook indices |
| **Defense** | Unsupervised adversarial fine-tuning of tokenizer encoder |

---

## VQ Libraries

### vector-quantize-pytorch

> **GitHub:** [lucidrains/vector-quantize-pytorch](https://github.com/lucidrains/vector-quantize-pytorch)

Comprehensive PyTorch library for VQ methods:

```
VQ, ResidualVQ, FSQ, LFQ, GIVT, GroupedResidualVQ, EMAVQ, StochasticResidualVQ, ...
```

---

## Security vs Utility Tradeoff

| Scenario | Approach | Quality Impact |
|----------|----------|---------------|
| **Maximum security** | Q-MLLM (2-level VQ) | Small utility trade-off |
| **Balanced** | UniTok encoder (v2) | Better quality, still secure |
| **Maximum quality** | No VQ bottleneck | Full vulnerability |

> **Key tradeoff:** VQ creates a discrete bottleneck that blocks adversarial gradients but also limits the model's ability to represent fine-grained visual details.

---

## Papers Table

| Paper | Year | Venue | arXiv | Key Contribution |
|-------|------|-------|-------|-----------------|
| **Q-MLLM** | 2026 | NDSS | [2511.16229](https://arxiv.org/abs/2511.16229) | 2-level VQ for adversarial defense |
| **Discrete Tokenization Survey** | 2025 | — | [2507.22920](https://arxiv.org/abs/2507.22920) | Comprehensive taxonomy |
| **VQToken** | 2025 | NeurIPS | [2503.16980](https://arxiv.org/abs/2503.16980) | 0.07% token reduction for video |
| **VAEVQ** | 2026 | AAAI | [2511.06863](https://arxiv.org/abs/2511.06863) | Variational VQ for tokenization |
| **LG-VQ** | 2024 | NeurIPS | [2405.14206](https://arxiv.org/abs/2405.14206) | Language-guided codebook learning |
| **Adversarial Tokenizer Robustness** | 2026 | — | [2602.18252](https://arxiv.org/abs/2602.18252) | Systematic study of tokenizer vulnerability |
| **UniCode** | 2024 | ECCV | [2403.09072](https://arxiv.org/abs/2403.09072) | Unified codebook for visual+text |

---

## GitHub Repositories

| Repo | Description |
|------|-------------|
| [Amadeuszhao/QMLLM](https://github.com/Amadeuszhao/QMLLM) | Q-MLLM implementation (NDSS 2026) |
| [Hai-chao-Zhang/VQToken](https://github.com/Hai-chao-Zhang/VQToken) | VQToken for video LLMs |
| [jindongli-Ai/LLM-Discrete-Tokenization-Survey](https://github.com/jindongli-Ai/LLM-Discrete-Tokenization-Survey) | Comprehensive survey repo |
| [lucidrains/vector-quantize-pytorch](https://github.com/lucidrains/vector-quantize-pytorch) | All VQ methods in PyTorch |
| [script-Yang/VAEVQ](https://github.com/script-Yang/VAEVQ) | VAEVQ implementation |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Core idea** | VQ creates non-differentiable discrete bottleneck blocking adversarial gradients |
| **Q-MLLM** | 2-level VQ (semantic K=128 + patch P=16K) for defense; 98.4% jailbreak DSR |
| **Why it works** | Gradient-based attacks can't backprop through discrete codebook |
| **Key tradeoff** | Security vs utility — discrete bottleneck limits fine-grained representation |
| **VQToken** | NeurIPS 2025 — 0.07% token reduction for video LLMs |
| **VAEVQ** | AAAI 2026 — variational modeling for better token distributions |

---

*Last updated: 2026-04-19*
