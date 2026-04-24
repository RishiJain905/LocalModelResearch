# FSDP2 — Fully Sharded Data Parallel v2

> *"FSDP2 uses per-parameter DTensor sharding for simpler representation compared to FSDP1's flat-parameter sharding."* — [PyTorch Docs](https://docs.pytorch.org/docs/stable/distributed.fsdp.fully_shard.html)

## Contents

1. [Overview](#overview)
2. [FSDP1 vs FSDP2](#fsdp1-vs-fsdp2)
3. [Sharding Strategy Mapping](#sharding-strategy-mapping)
4. [Key Technical Sources](#key-technical-sources)
5. [GitHub Repositories](#github-repositories)
6. [Benchmarks & Performance](#benchmarks--performance)
7. [FSDP2 Configuration (Axolotl)](#fsdp2-configuration-axolotl)
8. [Known Issues & Limitations](#known-issues--limitations)
9. [When to Use FSDP2 vs Alternatives](#when-to-use-fsdp2-vs-alternatives)
10. [Related Topics](#related-topics)

---

## Overview

FSDP2 is PyTorch's rewritten implementation of Fully Sharded Data Parallel, using **per-parameter DTensor sharding** instead of the deprecated `FlatParameter` abstraction from FSDP1. It shards model parameters, gradients, and optimizer states across GPUs.

> *"FSDP1 is deprecated. This tutorial uses FSDP2 (formerly called torch.distributed.fsdp), which offers improved memory management, DTensor support, and simpler APIs."* — [PyTorch FSDP2 Tutorial](https://docs.pytorch.org/tutorials/intermediate/FSDP_tutorial.html)

| Metric | Value |
|--------|-------|
| **API Entry Point** | `torch.distributed.fsdp.fully_shard()` |
| **Parameter Sharding** | Per-parameter via DTensor dim-0 |
| **Default State Dict** | `SHARDED_STATE_DICT` (communication-free) |
| **Recommended Prefetch** | `BACKWARD_PRE` (only correct option) |
| **BF16 MFU (H100, 8B)** | 33-39% |
| **Memory Savings vs FSDP1** | 7% lower peak memory |

---

## FSDP1 vs FSDP2

| Aspect | FSDP1 | FSDP2 |
|--------|-------|-------|
| **Parameter Representation** | `FlatParameter` — flattens/concatenates params into 1D tensor for communication buckets | `DTensor` — per-parameter dim-0 sharding via `torch.chunk()` |
| **State Dict** | Requires all-gather communication | Communication-free sharded state dicts |
| **Memory Management** | Uses `torch.Tensor.record_stream` with CPU sync via `limit_all_gathers=True` | Avoids `recordStream`; lower/deterministic memory |
| **Meta-Device Init** | Complex, requires `param_init_fn` | Simplified flow with meta device |
| **Auto-Wrap Policy** | Required for performance | **Removed** |
| **Backward Prefetch** | Configurable (`BACKWARD_PRE`/`BACKWARD_POST`) | Always `BACKWARD_PRE` (only correct way) |
| **Frozen Parameters** | Constrained | No extra memory for mixed frozen/non-frozen |
| **LoRA/PEFT** | Limited/complex | Native per-parameter manipulation |
| **FP8 Support** | Not native | Native via custom all-gather extension |

> *"FSDP2 uses DTensor-based dim-0 per-parameter sharding for a simpler sharding representation compared to FSDP1's flat-parameter sharding."* — [PyTorch `fully_shard` API Reference](https://docs.pytorch.org/docs/stable/distributed.fsdp.fully_shard.html)

### FSDP1 → FSDP2 Migration Reference

| FSDP1 | FSDP2 | Notes |
|-------|-------|-------|
| `fsdp_sharding_strategy` | `reshard_after_forward` | Renamed |
| `fsdp_backward_prefetch` | **REMOVED** | Always `BACKWARD_PRE` |
| `fsdp_forward_prefetch` | **REMOVED** | Under discussion |
| `fsdp_sync_module_states` | **REMOVED** | Redundant |
| `fsdp_cpu_ram_efficient_loading` | `cpu_ram_efficient_loading` | Same behavior |
| `fsdp_state_dict_type` | `state_dict_type` | `LOCAL_STATE_DICT` obsolete |
| `fsdp_use_orig_params` | **REMOVED** | Always uses original params |
| `auto_wrap_policy` | **REMOVED** | No replacement |
| `param_init_fn` | **REMOVED** | Meta-device flow available |

> *"FSDP2 is the recommended approach — it uses per-parameter sharding via DTensor instead of the monolithic FlatParameter"* — [Hugging Face Accelerate](https://huggingface.co/docs/accelerate/concept_guides/fsdp1_vs_fsdp2)

---

## Sharding Strategy Mapping

| FSDP1 | FSDP2 | DeepSpeed Equivalent |
|-------|-------|---------------------|
| `FULL_SHARD` | `reshard_after_forward=True` | ZeRO-3 |
| `SHARD_GRAD_OP` | `reshard_after_forward=False` | ZeRO-2 |
| `HYBRID_SHARD` | 2D mesh + `reshard_after_forward=True` | MiCS |

> *"TorchTitan integrates a new version of Fully Sharded Data Parallel (FSDP2), which uses the per-parameter Distributed Tensor sharding representation and thus provides better composability with model parallelism techniques."* — [TorchTitan Paper](https://arxiv.org/html/2410.06511v1)

---

## Key Technical Sources

### Official Documentation

| Resource | URL |
|----------|-----|
| **PyTorch FSDP2 Tutorial** | [docs.pytorch.org/tutorials/intermediate/FSDP_tutorial.html](https://docs.pytorch.org/tutorials/intermediate/FSDP_tutorial.html) |
| **`fully_shard` API Reference** | [docs.pytorch.org/docs/stable/distributed.fsdp.fully_shard.html](https://docs.pytorch.org/docs/stable/distributed.fsdp.fully_shard.html) |
| **PyTorch FSDP Main Docs** | [docs.pytorch.org/docs/stable/fsdp.html](https://docs.pytorch.org/docs/stable/fsdp.html) |

### Academic Papers

**TorchTitan: One-stop PyTorch Native Solution for Production Ready LLM Pre-training**
> *"On some Llama-7B runs on 8× H100s, FSDP2 achieves higher MFU with 7% lower peak memory than FSDP1, matching the same loss curve."* — [arXiv:2410.06511v1](https://arxiv.org/html/2410.06511v1)

**veScale-FSDP: Flexible and High-Performance FSDP at Scale**
> *"FSDP2 per-parameter DTensor introduces interleaved copy-out after all-gather and interleaved copy-in"* — [arXiv:2602.22437v2](https://arxiv.org/html/2602.22437v2)
> Claims: 5-66% higher throughput, 16-30% lower memory than existing FSDP

**The Trade-offs of FP8 vs BF16 Training in LLMs**
> *"FP8 training improved the training speed from 415 TFLOPS (with BF16) to a maximum of 570 TFLOPS."* — [arXiv:2411.08719v1](https://arxiv.org/html/2411.08719v1)

### Blog Posts

**Hugging Face — FSDP1 vs FSDP2**
> *"FSDP2 is the recommended approach — it uses per-parameter sharding via DTensor instead of the monolithic FlatParameter"* — [Hugging Face Accelerate](https://huggingface.co/docs/accelerate/concept_guides/fsdp1_vs_fsdp2)

**IBM Research — Maximizing Training Throughput**
> Achieved 3,700 tokens/sec/GPU, 40B tokens/day on 128 A100 GPUs (57% MFU) — [IBM Research Blog](https://research.ibm.com/blog/pytorch-fsdp)

**ZeRO-3 vs FSDP**
> *"ZeRO-3 often has a slight edge in extremely constrained environments due to its highly optimized memory allocator... FSDP on the other hand, can sometimes suffer from higher memory fragmentation."* — [Lyceum Technology](https://lyceum.technology/magazine/zero-3-vs-fsdp-memory-efficiency/)

> *"FSDP excels in the mid range (100M–1B). DeepSpeed opens its advantage later (10B+)"* — [Romeo Kienzler (Medium)](https://romeokienzler.medium.com/fsdp-vs-deepspeed-9df47ee5ccbb)

---

## GitHub Repositories

| Repository | Description |
|-----------|-------------|
| [pytorch/torchtitan](https://github.com/pytorch/torchtitan) | Primary FSDP2 reference — Llama training recipes, benchmarks |
| [pytorch/examples](https://github.com/pytorch/examples/blob/main/distributed/FSDP2/checkpoint.py) | FSDP2 checkpoint.py example |
| [pytorch/tutorials](https://github.com/pytorch/tutorials/blob/main/intermediate_source/FSDP_tutorial.rst) | FSDP2 tutorial source |
| [axolotl-ai-cloud/axolotl](https://github.com/axolotl-ai-cloud/axolotl) | Fine-tuning with FSDP2 via `fsdp_version: 2` |
| [huggingface/accelerate](https://huggingface.co/docs/accelerate/concept_guides/fsdp1_vs_fsdp2) | FSDP2 via Accelerate |

---

## Benchmarks & Performance

### TorchTitan Official Benchmarks (H100 GPUs)

| Configuration | MFU (Model FLOPS Utilization) |
|--------------|------------------------------|
| Llama 3.1 8B, 8× H100, no Float8 | 33-39% (with/without torch.compile) |
| Llama 3.1 8B, 128× H100 | ~33% MFU |

> *"On some Llama-7B runs on 8× H100s, FSDP2 achieves higher MFU with 7% lower peak memory than FSDP1, matching the same loss curve."* — [TorchTitan](https://github.com/pytorch/torchtitan/blob/main/docs/fsdp.md)

### FSDP vs DeepSpeed ZeRO Comparison

| Aspect | ZeRO-3 | FSDP |
|--------|--------|------|
| Memory Efficiency | Superior in extreme constraints | Better when model fits in VRAM |
| TFLOPS/GPU | Lower at 10B+ | Higher at mid-range (100M-1B) |
| Offloading | ZeRO-Infinity (CPU+NVMe) | Limited to CPU param/gradient |
| Fragmentation Risk | Lower | Higher if misconfigured |

> *"FSDP excels in the mid range (100M–1B). DeepSpeed opens its advantage later (10B+), where memory savings become more important than raw throughput."* — [Romeo Kienzler](https://romeokienzler.medium.com/fsdp-vs-deepspeed-9df47ee5ccbb)

### FP8 Performance

> *"FP8 training improved the training speed from 415 TFLOPS (with BF16) to a maximum of 570 TFLOPS."* — [arXiv:2411.08719v1](https://arxiv.org/html/2411.08719v1)

---

## FSDP2 Configuration (Axolotl)

```yaml
fsdp_version: 2
fsdp_config:
  offload_params: false
  cpu_ram_efficient_loading: true
  auto_wrap_policy: TRANSFORMER_BASED_WRAP
  transformer_layer_cls_to_wrap: Qwen3DecoderLayer
  state_dict_type: FULL_STATE_DICT
  reshard_after_forward: true
```

> *"FSDP2 is recommended for new users. FSDP1 is deprecated and will be removed in a future release."* — [Axolotl Multi-GPU Docs](https://docs.axolotl.ai/docs/multi-gpu.html)

---

## Known Issues & Limitations

| Issue | URL / Reference |
|-------|-----------------|
| Memory leak with reentrant gradient checkpointing | [PyTorch Issue #169349](https://github.com/pytorch/pytorch/issues/169349) |
| FSDP2 + SDPA attention bug | [PyTorch Issue #173709](https://github.com/pytorch/pytorch/issues/173709) |
| Parameter sharing not supported | [TorchTitan Issue #2817](https://github.com/pytorch/torchtitan/issues/2817) |
| Fails with different parameter usage across ranks | [PyTorch Issue #158719](https://github.com/pytorch/pytorch/issues/158719) |
| Ray Train FSDP2 hangs | [Ray Issue #57860](https://github.com/ray-project/ray/issues/57860) |
| Memory leak with `get_model_state_dict` | [PyTorch Issue #149100](https://github.com/pytorch/pytorch/issues/149100) |
| Saving models with FSDP2 hangs | [Buttondown Weekly](https://buttondown.com/weekly-project-news/archive/weekly-github-report-for-pytorch-december-01-2025-3825/) |
| Qwen-3 FSDP2 failures | [Axolotl Issue #3056](https://github.com/axolotl-ai-cloud/axolotl/issues/3056) |

| Limitation | Description |
|------------|-------------|
| **No Parameter Sharing** | Shared parameters across modules not supported |
| **CPU Offloading** | Less feature-rich than DeepSpeed ZeRO-Infinity |
| **Mixed Frozen Parameters** | State dict iteration can hang |
| **2D Mesh Complexity** | HSDP configuration more complex than 1D |

---

## When to Use FSDP2 vs Alternatives

| Scenario | Recommendation |
|----------|----------------|
| Native PyTorch experience | **FSDP2** |
| Extreme memory constraints (100B+) | **DeepSpeed ZeRO-3** |
| LoRA/QLoRA fine-tuning | **FSDP2** with `fsdp_version: 2` |
| CPU offloading needed | **DeepSpeed** (ZeRO-Infinity) |
| Mid-range models (100M-10B) | **FSDP2** |
| torch.compile integration | **FSDP2** |
| Production with complex orchestration | **DeepSpeed** |

---

## Related Topics

- [Axolotl](./Axolotl.md) — Fine-tuning framework with native FSDP2 support
- [Unsloth](./Unsloth.md) — Single-GPU optimized fine-tuning (FSDP2 not needed)
- [QAT](./QAT.md) — Quantization-Aware Training that can be combined with FSDP2
