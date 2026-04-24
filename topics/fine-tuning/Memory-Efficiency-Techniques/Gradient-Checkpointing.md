# Gradient Checkpointing

## 📌 Overview

| Field | Value |
|-------|-------|
| **Also Known As** | Activation Recomputation, Activation Checkpointing |
| **Original Paper** | Chen et al. 2016 — arXiv:1604.06174 |
| **Memory Savings** | 60–70% activation memory reduction |
| **Compute Overhead** | 20–33% additional FLOPs |
| **Memory Complexity** | O(n) → O(√n) per layer |
| **AMD ROCm Support** | ✅ Full (PyTorch 2.0+ / ROCm 5.4+) |

> "Activation checkpointing is a technique that trades compute for memory."

— [PyTorch Documentation](https://pytorch.org/docs/stable/checkpoint.html)

---

## What Is Gradient Checkpointing?

Gradient checkpointing (also called **activation recomputation**) is a memory optimization technique for neural network training that trades **compute for memory**:

- During forward pass: only **selected checkpoint activations** are stored
- Intermediate activations are **discarded** rather than stored
- During backprop: discarded activations are **recomputed on-the-fly** from nearest checkpoint
- Result: memory reduces from **O(n) to O(√n)** for an n-layer network

> "We design an algorithm that costs **O(sqrt(n)) memory** to train a n layer network, with only the computational cost of **an extra forward pass per mini-batch**."

— [Chen et al. 2016](https://arxiv.org/abs/1604.06174)

> "We can reduce the memory cost of a 1,000-layer deep residual network from **48G to 7G** on ImageNet problems."

— [Chen et al. 2016](https://arxiv.org/abs/1604.06174)

---

## Memory & Compute Tradeoffs

### Theoretical Reduction

| Configuration | Memory Complexity | Notes |
|--------------|-------------------|-------|
| Standard backprop | O(n) | All activations stored |
| Gradient checkpointing | O(√n) | Optimal for feed-forward nets |
| Nested checkpointing (PyTorch 2.1+) | O(log n) | Multiple levels |

### Empirical Benchmarks

**Llama 3 8B (Sequence Length 2048):**

| Metric | Without Checkpointing | With Checkpointing |
|--------|----------------------|-------------------|
| Activation Memory | ~40 GB | ~12 GB |
| Batch Size | 4 | 16 (4× larger) |
| Training Time | 100 min | 130 min (+30%) |

— [21medien](https://21medien.de/en/library/gradient-checkpointing)

**General:**

| Metric | Value |
|--------|-------|
| Peak memory reduction | ~60% |
| Compute overhead | 20–33% additional FLOPs |
| Feed-forward model capacity | 10× larger models fit in same GPU memory |

> "For feed-forward models we were able to fit more than **10x larger models** onto our GPU, at only a **20% increase in computation time**."

— [cybertronai/gradient-checkpointing](https://github.com/cybertronai/gradient-checkpointing)

> "Model checkpointing reduced peak model memory usage by about 60%, while increasing model training time by about 25%."

— [ResidentMario](https://residentmario.github.io/pytorch-training-performance-guide/gradient-checkpoints.html)

---

## VRAM Requirements by Model Size

| Model | Full Fine-tuning | LoRA Fine-tuning | LoRA + Gradient Checkpointing |
|-------|-----------------|-----------------|-------------------------------|
| 7B | 100–120 GB | ~28 GB | ~12 GB |
| 13B | 160–240 GB | ~40 GB | ~20 GB |
| 70B | 640+ GB | 80+ GB | ~40 GB |

> "Full fine-tuning of a 7-billion parameter model requires 100-120 GB of VRAM. The same model fine-tunes on a $1,500 RTX 4090 using QLoRA."

— [Introl](https://introl.com/blog/fine-tuning-infrastructure-lora-qlora-peft-scale-guide-2025)

> "QLoRA enables 70B model fine-tuning on single A100 80GB vs 4-8 GPUs for full fine-tuning"

— [Introl](https://introl.com/blog/fine-tuning-infrastructure-lora-qlora-peft-scale-guide-2025)

---

## APIs

### PyTorch `torch.utils.checkpoint`

```python
import torch.utils.checkpoint as checkpoint

# Method 1: checkpoint() — flexible for any callable
x = checkpoint.checkpoint(self.block, x, use_reentrant=False)

# Method 2: checkpoint_sequential() — for nn.Sequential models
x = checkpoint.checkpoint_sequential(model, num_segments, input_var)
```

**Key Parameters:**
| Parameter | Notes |
|-----------|-------|
| `preserve_rng_state` | Stash/restore RNG state (default True) |
| `use_reentrant` | **Must pass explicitly**; use_reentrant=False recommended |
| `early_stop` | Stop recomputation once needed tensors available |

### HuggingFace Transformers

```python
from transformers import AutoModelForCausalLM, TrainingArguments

model = AutoModelForCausalLM.from_pretrained("meta-llama/Llama-3-8B")
model.gradient_checkpointing_enable()

training_args = TrainingArguments(
    gradient_checkpointing=True,
    gradient_checkpointing_kwargs={"use_reentrant": False},
)
```

### DeepSpeed

```python
import deepspeed.checkpointing as checkpointing

checkpointing.configure(
    mpu=mpu,
    partition_activations=True,
    contiguous_checkpointing=True,
    num_checkpoints=4,
    checkpoint_in_cpu=True,
)
```

**JSON config:**
```json
{
  "activation_checkpointing": {
    "enabled": true,
    "partition_activations": true,
    "contiguous_checkpointing": true,
    "num_checkpoints": 4,
    "checkpoint_in_cpu": true
  }
}
```

---

## AMD ROCm Compatibility

### Status: ✅ FULLY SUPPORTED

Gradient checkpointing in PyTorch relies on standard autograd mechanisms — fully supported on AMD GPUs through ROCm.

> "With PyTorch 2.0 and ROCm 5.4+, LLM training works out of the box on AMD MI250 accelerators with **zero code changes** when running our LLM Foundry training stack."

— [Databricks Blog](https://www.databricks.com/blog/amd-mi250)

> "Given the consistent performance we see across many MPT model sizes, we believe that **at the right price, AMD and NVIDIA systems are interchangeable for LLM training**."

— [Databricks Blog](https://www.databricks.com/blog/amd-mi250)

> "Core PyTorch functionality on ROCm includes tensor operations, neural network layers, automatic differentiation, distributed training, mixed-precision training, compilation features."

— [ROCm Documentation](https://rocm.docs.amd.com/en/latest/compatibility/ml-compatibility/pytorch-compatibility.html)

**Requirements:**
- PyTorch 2.0+ with ROCm 5.4+ support
- Compatible AMD GPU (gfx90a for MI250, gfx942 for MI300)

---

## LoRA + Gradient Checkpointing — Known Issue

### ⚠️ Memory Savings Don't Materialize

> "Gradient checkpointing fails to reduce memory usage when using PEFT LoRA adapters, even with correct configuration."

— [HuggingFace GitHub Issue #42947](https://github.com/huggingface/transformers/issues/42947)

**Workarounds:**
1. Ensure `use_cache=False` is set on the model
2. Call `model.gradient_checkpointing_enable()` **BEFORE** applying LoRA adapters
3. Use `use_reentrant=False` in gradient_checkpointing_kwargs

---

## Common Pitfalls

| Pitfall | Solution |
|---------|----------|
| Incompatible with Dropout/BatchNorm | Move Dropout and BatchNorm outside checkpointed segments |
| Non-deterministic gradients | Set preserve_rng_state=True |
| use_reentrant warning | Pass use_reentrant=False explicitly |
| LoRA + checkpointing issue | Enable checkpointing BEFORE applying LoRA adapters |
| use_cache=True incompatibility | Set model.config.use_cache=False |

---

## Summary Data Points

| Metric | Value |
|--------|-------|
| Memory reduction | 60–70% (activation memory) |
| Compute overhead | 20–33% additional FLOPs |
| Memory complexity | O(n) → O(√n) |
| Nested checkpointing | O(√n) → O(log n) |
| Checkpointing overhead | ~1 extra forward pass per batch |
| Original paper | Chen et al. 2016, arXiv:1604.06174 |
| AMD MI250 support | ✅ Confirmed |
| PyTorch min version | 2.0+ / ROCm 5.4+ |

---

## Related Topics

- [xformers-ROCm.md](./xformers-ROCm.md) — Memory-efficient attention on AMD ROCm
- [gfx-hardware-overrides](./gfx-hardware-overrides/) — HSA_OVERRIDE_GFX, RDNA3 optimization
- [Memory-Efficiency-Techniques](./README.md) — Parent folder overview

---

## References

- [Chen et al. 2016 — Training Deep Nets with Sublinear Memory Cost](https://arxiv.org/abs/1604.06174)
- [PyTorch torch.utils.checkpoint](https://pytorch.org/docs/stable/checkpoint.html)
- [DeepSpeed Activation Checkpointing](https://deepspeed.readthedocs.io/en/latest/activation-checkpointing.html)
- [cybertronai/gradient-checkpointing](https://github.com/cybertronai/gradient-checkpointing)
- [21medien — Gradient Checkpointing](https://21medien.de/en/library/gradient-checkpointing)
- [ResidentMario — PyTorch Training Performance Guide](https://residentmario.github.io/pytorch-training-performance-guide/gradient-checkpoints.html)
- [Databricks — AMD MI250 LLM Training](https://www.databricks.com/blog/amd-mi250)
- [ROCm — PyTorch Compatibility](https://rocm.docs.amd.com/en/latest/compatibility/ml-compatibility/pytorch-compatibility.html)
- [HuggingFace #42947 — LoRA + Checkpointing Issue](https://github.com/huggingface/transformers/issues/42947)
- [Introl — Fine-tuning Infrastructure Guide](https://introl.com/blog/fine-tuning-infrastructure-lora-qlora-peft-scale-guide-2025)
