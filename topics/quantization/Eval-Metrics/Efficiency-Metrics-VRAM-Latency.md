# Efficiency Metrics: VRAM vs Latency

> **Research Method:** Direct HTTP fetching from arxiv.org/abs/ and GitHub API (April 2026).
>
> **GitHub:** [ggerganov/llama.cpp](https://github.com/ggerganov/llama.cpp) ⭐104.8K · [vllm-project/vllm](https://github.com/vllm-project/vllm) ⭐77.3K · [mit-han-lab/llm-awq](https://github.com/mit-han-lab/llm-awq) ⭐3.5K · [SJTU-IPADS/PowerInfer](https://github.com/SJTU-IPADS/PowerInfer) ⭐9.3K

---

## Overview

> *"Quantization is fundamentally a trade-off: every bit you save in VRAM costs something in latency, and the ratio depends on hardware, precision, and batch size."*

Efficiency metrics measure the relationship between memory footprint (VRAM) and inference speed (latency/throughput) for quantized models.

---

## Key Metrics

| Metric | Description | Trade-off |
|--------|-------------|-----------|
| **VRAM Usage** | Peak GPU memory during inference | Lower = more users can run |
| **Latency (ms/token)** | Time per output token | Lower = faster response |
| **Throughput (tok/s)** | Tokens per second | Higher = more concurrent users |
| **Memory Bandwidth Utilization** | % of bandwidth used | Higher = better hardware use |
| **Batch Size** | Concurrent sequences | Larger = better throughput |

---

## VRAM Breakdown (70B Model)

| Precision | VRAM (70B) | Compression vs FP16 |
|-----------|-----------|---------------------|
| FP16 (native) | ~140 GB | 1× |
| INT8 | ~70 GB | 2× |
| INT4 | ~35 GB | 4× |
| NF4 (bitsandbytes) | ~35 GB | 4× |
| INT4 + KV Cache | ~38 GB | ~3.7× |

> *KV cache adds significant overhead for long-context models.*

---

## Latency Scaling Patterns

> *"Latency doesn't scale linearly with bit width — memory bandwidth becomes the bottleneck at low precisions."*

| Precision | Latency vs FP16 | Notes |
|-----------|----------------|-------|
| FP16 | 1.0× (baseline) | Full precision |
| FP8 | ~0.9–1.0× | Minimal overhead |
| INT8 | ~0.8–0.95× | Mixed results — depends on kernel optimization |
| INT4 | ~0.6–0.8× | Significant speedup on memory-bound operations |
| INT4 + dequant | ~0.5–0.7× | Dequantization overhead can reduce gains |

---

## Hardware Dependency

| Hardware | Optimal Strategy | Bottleneck |
|----------|-----------------|------------|
| **H100/H200** | FP8 > INT8 > INT4 | Compute (Tensor Core) |
| **A100** | INT8 > INT4 > FP16 | Memory bandwidth (2 TB/s) |
| **RTX 4090** | INT4 (Q4_K_M) | Memory bandwidth +算术 |
| **Mac M1/M2/M3** | INT8 + Apple Silicon | Unified memory bandwidth |
| **RTX 3090** | INT8 | Memory bandwidth |

---

## Batch Size vs Latency Trade-off

| Setting | Latency | Throughput | Best For |
|---------|---------|------------|----------|
| **BS=1** | Low per-token | Low total | Interactive use |
| **BS=8** | Medium | Higher | Small deployments |
| **BS=32** | Higher | Much higher | Server deployments |
| **BS=256+** | High | Maximum | High-volume inference |

> At low batch sizes, INT4 models on consumer GPUs (RTX 4090) can match FP16 throughput of A100s at a fraction of the cost — but at high batch sizes, memory bandwidth advantages of FP8 on H100 dominate.

---

## Key Papers

### "QServe: Efficient LLM Serving with 4-bit Quantization"

> **arXiv:** [2408.12757](https://arxiv.org/abs/2408.12757)

Analyzes throughput gains from 4-bit quantization on modern GPUs — shows w4a16 can be more efficient than w8a8 for certain workloads.

### "PowerInfer: Fast Large Language Model Serving with Sparse Quantization"

> **GitHub:** [SJTU-IPADS/PowerInfer](https://github.com/SJTU-IPADS/PowerInfer)

Exploits layer-wise activation sparsity for faster inference on consumer GPUs — INT4 with sparse activation patterns.

---

## Repos

| Resource | Link | Description |
|----------|------|-------------|
| llama.cpp | [ggerganov/llama.cpp](https://github.com/ggerganov/llama.cpp) | Efficient CPU/GPU inference with quantization |
| vLLM | [vllm-project/vllm](https://github.com/vllm-project/vllm) | High-throughput LLM inference |
| PowerInfer | [SJTU-IPADS/PowerInfer](https://github.com/SJTU-IPADS/PowerInfer) | Sparse-quantized inference |
| AutoAWQ | [mit-han-lab/llm-awq](https://github.com/mit-han-lab/llm-awq) | Activation-aware weight quantization |

---

## Real-World Efficiency Examples

### Consumer GPU (RTX 4090, 24GB)

| Model | Precision | VRAM | Throughput |
|-------|-----------|------|------------|
| Llama-3 70B | FP16 | OOM | — |
| Llama-3 70B | INT4 (Q4_K_M) | ~38 GB | ~15 tok/s |
| Llama-3 70B | INT8 | ~70 GB | ~25 tok/s |

### Data Center GPU (A100 80GB)

| Model | Precision | VRAM | Throughput |
|-------|-----------|------|------------|
| Llama-3 70B | FP16 | ~140 GB | ~50 tok/s |
| Llama-3 70B | INT8 | ~70 GB | ~80 tok/s |
| Llama-3 70B | INT4 | ~35 GB | ~120 tok/s |

---

## Key Insight

> The **batch size-latency trade-off**: Hardware choice depends heavily on your use case:
> - **Interactive (BS=1)**: Consumer GPUs with INT4 are very competitive
> - **High-throughput (BS=32+)**: Data center GPUs with FP8/INT8 dominate

---

## See Also

- [Eval Metrics README](./README.md)
- [llama.cpp](../../Hardware-Kernels-Runtimes/gguf.md)
- [vLLM](../../Hardware-Kernels-Runtimes/compressed-tensors.md)
- [AWQ](../weight-quantization/awq.md)
