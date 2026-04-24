# QAT — Quantization-Aware Training

> *"QAT integrates quantization simulation into the training process, allowing the model to adjust for quantization noise when updating the weights"* — [Better ML](https://medium.com/better-ml/quantization-aware-training-qat-vs-post-training-quantization-ptq-cd3244f43d9a)

## Contents

1. [Overview](#overview)
2. [QAT vs PTQ Comparison](#qat-vs-ptq-comparison)
3. [Key Papers](#key-papers)
4. [Implementation Frameworks](#implementation-frameworks)
5. [Key Techniques](#key-techniques)
6. [Benchmarks & Accuracy Recovery](#benchmarks--accuracy-recovery)
7. [Limitations & Debates](#limitations--debates)
8. [Related Topics](#related-topics)

---

## Overview

Quantization-Aware Training (QAT) simulates quantization effects during training so that the model learns to compensate for quantization noise. Unlike Post-Training Quantization (PTQ), which quantizes a frozen model, QAT allows weight updates to adapt to low-precision representation.

> *"QAT can recover accuracy lost during PTQ, especially at INT4 and lower bit-widths"* — [apxml.com](https://apxml.com/courses/practical-llm-quantization/chapter-4-quantization-aware-training-qat/qat-vs-ptq-comparison)

---

## QAT vs PTQ Comparison

| Aspect | PTQ | QAT |
|--------|-----|-----|
| **Training Required** | No | Yes (full or partial) |
| **Accuracy at Low Bits** | Degradation | Better preservation |
| **Compute Cost** | Low | High |
| **Time to Deploy** | Fast | Longer |
| **Best For** | Large models, limited resources | Accuracy-critical, low-bit (INT4) |
| **Data Requirements** | Few-shot calibration | Full training dataset |

> *"QAT integrates quantization simulation into the training process, allowing the model to adjust for quantization noise when updating the weights"* — [Better ML](https://medium.com/better-ml/quantization-aware-training-qat-vs-post-training-quantization-ptq-cd3244f43d9a)

> *"QAT can recover up to **96% of accuracy degradation** on hellaswag and **68% of perplexity degradation on wikitext** for Llama3 compared to PTQ"* — [PyTorch Blog](https://pytorch.org/blog/quantization-aware-training/)

---

## Key Papers

### LLM-QAT: Data-Free Quantization Aware Training (Meta, 2023)

> *"Enables data-free training by generating synthetic data. Achieves ~20 points improvement over training-free methods at 4-bit weight/8-bit activation/4-bit KV cache quantization."* — [arXiv:2305.17888](https://arxiv.org/abs/2305.17888)

- **Paper:** [arXiv:2305.17888](https://arxiv.org/abs/2305.17888)
- **Repo:** [facebookresearch/LLM-QAT](https://github.com/facebookresearch/LLM-QAT)

### LR-QAT: Low-Rank Quantization-Aware Training (Qualcomm AI Research, 2024)

> *"Enables training a 7B LLM on a **single consumer GPU with 24GB memory**"* — [arXiv:2406.06385](https://arxiv.org/abs/2406.06385)

- **Paper:** [arXiv:2406.06385](https://arxiv.org/abs/2406.06385)
- **Repo:** [qualcomm-ai-research/LR-QAT](https://github.com/qualcomm-ai-research/LR-QAT)
- Uses low-rank reparameterization + checkpointing for memory efficiency

### EfficientQAT (ICLR 2025)

> *"Two-phase approach: Block-wise training (Block-AP) + end-to-end quantization parameter training (E2E-QP). Outperforms previous QAT methods across 7B-70B models."* — [arXiv:2407.11062](https://arxiv.org/abs/2407.11062)

- **Paper:** [arXiv:2407.11062](https://arxiv.org/abs/2407.11062)
- Block-wise training of all parameters → fine-tune quantization params end-to-end

### DL-QAT: Weight-Decomposed Low-Rank QAT (EMNLP 2024)

> *"Trains only **<1% of total parameters** while maintaining QAT benefits. Combines LoRA with quantization-aware fine-tuning."* — [ACL Anthology](https://aclanthology.org/2024.emnlp-industry.10/)

- **Source:** [aclanthology.org/2024.emnlp-industry.10](https://aclanthology.org/2024.emnlp-industry.10/)
- Weight-decomposed approach: updates weights in quantization space using low-rank adapters

---

## Implementation Frameworks

### PyTorch TorchAO / Torchtune

```python
# API for QAT
torchao.quantization.prototype.qat.Int8DynActInt4WeightQATQuantizer
```

> *"Disable fake quantization for first 1000 steps to allow weights to stabilize"* — [PyTorch QAT Guide](https://pytorch.org/blog/quantization-aware-training/)

**Distributed QAT training:**
```bash
tune run --nproc_per_node 8 qat_distributed --config llama3/8B_qat_full
```

**Key Finding:** QAT can recover **96% of hellaswag accuracy degradation** for Llama3 vs 62% recovery for Llama2 with PTQ.

### Unsloth QAT

> *"QAT recovers up to 70% of lost accuracy."* — [Unsloth Docs](https://unsloth.ai/docs/blog/quantization-aware-training-qat)

- **Supported Schemes:** INT4, FP8-INT4, FP8-FP8, INT8-INT4
- **Blog:** [kaitchup.substack.com](https://kaitchup.substack.com/p/unsloths-quantization-aware-training)

### Axolotl QAT

> *"`axolotl finetune qat-llama-3.2-3b`"* — [Axolotl Docs](https://docs.axolotl.ai/)

---

## Key Techniques

### Fake Quantization
Simulates quantization during forward passes while keeping weights in float dtype. Allows gradients to flow through quantization boundaries.

### Straight-Through Estimator (STE)
Handles non-differentiable rounding operations during backpropagation by passing gradients through unchanged.

### Block-wise Training (EfficientQAT)
Trains all parameters in blocks before fine-tuning quantization parameters end-to-end. Combines Block-AP + E2E-QP phases.

### Weight-Decomposed LoRA (DL-QAT)
Updates weights in quantization space using low-rank adapters, training <1% of total parameters while maintaining QAT accuracy benefits.

---

## Benchmarks & Accuracy Recovery

| Method | Llama3-8B Hellaswag Recovery | Wikitext Perplexity Recovery |
|--------|-------------------------------|-------------------------------|
| **PTQ (baseline)** | 62% | — |
| **QAT (PyTorch)** | 96% | 68% |

> *"QAT can recover up to **96% of accuracy degradation** on hellaswag and **68% of perplexity degradation on wikitext** for Llama3 compared to PTQ"* — [PyTorch Blog](https://pytorch.org/blog/quantization-aware-training/)

| Model | Memory Requirement | Method |
|-------|--------------------:|--------|
| 7B LLM | Single 24GB GPU | LR-QAT |

> *"Enables training a 7B LLM on a **single consumer GPU with 24GB memory**"* — [Qualcomm AI Research](https://github.com/qualcomm-ai-research/LR-QAT)

---

## Limitations & Debates

| Issue | Description |
|-------|-------------|
| **Training Cost** | QAT requires full training — significantly more compute than PTQ |
| **Implementation Complexity** | Fake quantization, STE, and per-layer calibration require careful implementation |
| **Bit-width Floor** | QAT still degrades at extremely low bits (<INT4); gains diminish |
| **Hyperparameter Sensitivity** | When to disable fake quantization (e.g., first 1000 steps) is critical and model-dependent |
| **Not a Silver Bullet** | Cannot fully recover all accuracy lost to aggressive quantization |

---

## Related Topics

- [Unsloth](./Unsloth.md) — Implements QAT with up to 70% accuracy recovery
- [Axolotl](./Axolotl.md) — QAT fine-tuning via `axolotl finetune qat-*` configs
- [FSDP2](./FSDP2.md) — Distributed backend that supports QAT workloads
