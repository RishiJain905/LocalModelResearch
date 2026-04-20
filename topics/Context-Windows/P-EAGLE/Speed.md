# P-EAGLE: Speed & Benchmarks

> **Source:** [arXiv:2602.01469](https://arxiv.org/abs/2602.01469) · [AWS ML Blog](https://aws.amazon.com/blogs/machine-learning/p-eagle-faster-llm-inference-with-parallel-speculative-decoding-in-vllm/) · [SGLang Issue #23171](https://github.com/sgl-project/sglang/issues/23171) · [EAGLE-3 arXiv:2503.01840](https://arxiv.org/abs/2503.01840)  
> **Subtopic of:** [P-EAGLE](./README.md)

---

## P-EAGLE Speedup Numbers

### vs. Autoregressive EAGLE-3

| Target Model | Speedup |
|---|---|
| GPT-OSS 120B | ~1.10× |
| GPT-OSS 20B | ~1.36× (max observed) |
| Qwen3-Coder 30B | 1.10×–1.36× |

> *"P-EAGLE achieves **1.10×–1.36× speedup** over autoregressive EAGLE-3 across GPT-OSS 120B, 20B, and Qwen3-Coder 30B, implemented in vLLM."*  
> — [arXiv:2602.01469](https://arxiv.org/abs/2602.01469) (Abstract)

### NVIDIA B200 Upper Bound

| Concurrency | MT-Bench | HumanEval | SPEED-Bench |
|---|---|---|---|
| c=1 | **1.55×** | **1.55×** | **1.69×** |
| c=2 | 1.29× | 1.53× | 1.61× |
| c=4 | 1.35× | 1.45× | 1.54× |
| c=8 | 1.28× | 1.35× | 1.45× |
| c=16 | 1.27× | 1.31× | 1.40× |
| c=32 | 1.09× | 1.37× | 1.22× |
| c=64 | 1.05× | 1.23× | 1.25× |

> *"On NVIDIA B200 GPUs, P-EAGLE achieves **1.05×–1.69× speedup over vanilla EAGLE-3** on GPT-OSS 20B across MT-Bench, HumanEval, and SpeedBench."*  
> — [AWS ML Blog](https://aws.amazon.com/blogs/machine-learning/p-eagle-faster-llm-inference-with-parallel-speculative-decoding-in-vllm/)

### SGLang Issue Numbers

> *"GPT-OSS 120B, BS=8: P-EAGLE K=3 → ~2389 TPS, 1.34× over baseline, 1.07× over best EAGLE3"*  
> — [SGLang Issue #23171](https://github.com/sgl-project/sglang/issues/23171)

### End-to-End Throughput

| Configuration | Throughput | Speedup | Source |
|---|---|---|---|
| P-EAGLE GPT-OSS 120B K=3 | ~2389 TPS | 1.34× baseline | SGLang Issue, BS=8 |
| EAGLE-3 gpt-oss-120B peak | 2,418 tok/s | vs 2,328 baseline | Red Hat (independent) |
| Standard EAGLE Llama-3.1-8B | 601 tok/s | 1.43× baseline | JarvisLabs |

---

## EAGLE-3 Baseline (for Context)

EAGLE-3 is the base that P-EAGLE improves upon. These numbers are independently important as they represent the **current state-of-the-art** in speculative decoding before P-EAGLE.

### EAGLE-3 Speedup vs Vanilla Autoregressive Decoding

| Model | Method | MT-Bench | HumanEval | GSM8K | Alpaca | CNN/DM | **Mean** |
|---|---|---|---|---|---|---|---|
| Vicuna 13B | EAGLE-2 | 4.26× | 4.96× | 4.22× | 4.25× | 3.40× | 4.22× |
| Vicuna 13B | **EAGLE-3** | **5.58×** | **6.47×** | **5.32×** | **5.16×** | **5.01×** | **5.51×** |
| LLaMA-3.1 8B | EAGLE-2 | 3.16× | 3.66× | 3.39× | 3.28× | 2.65× | 3.23× |
| LLaMA-3.1 8B | **EAGLE-3** | **4.40×** | **4.85×** | **4.48×** | **4.82×** | **3.65×** | **4.44×** |
| LLaMA-3.3 70B | EAGLE-2 | 2.83× | 3.12× | 2.83× | 3.03× | 2.44× | 2.85× |
| LLaMA-3.3 70B | **EAGLE-3** | **4.11×** | **4.79×** | **4.34×** | **4.30×** | **3.27×** | **4.12×** |
| DeepSeek-R1-Distill 8B | EAGLE-2 | 2.92× | 3.42× | 3.40× | 3.01× | 3.53× | 3.26× |
| DeepSeek-R1-Distill 8B | **EAGLE-3** | **4.05×** | **4.59×** | **5.01×** | **3.65×** | **3.52×** | **4.16×** |

> *EAGLE-3 vs EAGLE-2: ~1.4× improvement*  
> *Maximum speedup: **6.47× on HumanEval** (Vicuna 13B)*  
> — [arXiv:2503.01840](https://arxiv.org/abs/2503.01840)

### EAGLE-3 Throughput vs Batch Size (vLLM)

| Batch Size | 2 | 4 | 8 | 16 | 24 | 32 | 48 | 56 |
|---|---|---|---|---|---|---|---|---|
| EAGLE | 1.30× | 1.25× | 1.21× | 1.10× | 1.03× | 0.93× | 0.82× | 0.71× |
| **EAGLE-3** | **1.75×** | **1.68×** | **1.58×** | **1.49×** | **1.42×** | **1.36×** | **1.21×** | **1.01×** |

> *EAGLE peaks at batch size 24; EAGLE-3 peaks at batch size 56. EAGLE-3 maintains throughput improvement at larger batch sizes.*  
> — [arXiv:2503.01840](https://arxiv.org/abs/2503.01840)

### EAGLE-3 Temperature=1 Results

| Model | EAGLE-2 | EAGLE-3 |
|---|---|---|
| Vicuna 13B | 3.76× | **4.65×** |
| LLaMA-3.1 8B | 2.80× | **3.45×** |
| LLaMA-3.3 70B | 2.65× | **3.95×** |
| DeepSeek-R1-Distill-LLaMA 8B | 2.77× | **3.52×** |

---

## Context Scaling Comparison

Acceptance length = avg draft tokens accepted per verification round. Higher = better throughput per round.

| Method | Layers | 1K | 4K | 8K | 20K |
|---|---|---|---|---|---|
| ParallelSpec + EAGLE-3 | 1 | 1.5 | 1.6 | OOM | OOM |
| PARD + EAGLE-3 | 4 | 2.4 | Infeasible | OOM | OOM |
| **P-EAGLE** | **4** | **2.4** | **2.8** | **2.9** | **3.0** |

> *Only P-EAGLE handles 20K context length. All other methods OOM at 8K+.*  
> — [arXiv:2602.01469](https://arxiv.org/abs/2602.01469)

---

## Important Caveats

### 1. Speedup Mechanism
> *"P-EAGLE's speedup comes from eliminating sequential drafting overhead, NOT from improving acceptance rate. The acceptance length of P-EAGLE is comparable to or slightly lower than AR EAGLE-3, but the single-forward-pass architecture removes the K-pass bottleneck."*  
> — [arXiv:2602.01469](https://arxiv.org/abs/2602.01469)

### 2. Hardware Dependency
> *"The 1.69× upper bound on B200 GPUs may not translate to H100/A100, where memory bandwidth differs."*  
> — [AWS ML Blog](https://aws.amazon.com/blogs/machine-learning/p-eagle-faster-llm-inference-with-parallel-speculative-decoding-in-vllm/)

### 3. Distribution Mismatch Fix
> *"P-EAGLE specifically addresses the ~25% acceptance rate degradation when draft models trained on short sequences (4K) are applied to long reasoning outputs (median 3,891 tokens, P90 10,800 for GPT-OSS 120B)."*  
> — [arXiv:2602.01469](https://arxiv.org/abs/2602.01469)

### 4. Independent Evaluations
No independent academic benchmarks of P-EAGLE were found. All primary numbers come from the authors' implementation in vLLM or the AWS blog post.

---

## Key Quotes

> *"P-EAGLE removes this ceiling by generating all K draft tokens in a single forward pass, delivering up to 1.69× speedup over vanilla EAGLE-3 on real workloads on NVIDIA B200."*  
> — [AWS ML Blog](https://aws.amazon.com/blogs/machine-learning/p-eagle-faster-llm-inference-with-parallel-speculative-decoding-in-vllm/)

> *"P-EAGLE achieves 1.10×–1.36× speedup over autoregressive EAGLE-3 across GPT-OSS 120B, 20B, and Qwen3-Coder 30B, implemented in vLLM."*  
> — [arXiv:2602.01469](https://arxiv.org/abs/2602.01469)

---

## 📚 References

- [arXiv:2602.01469](https://arxiv.org/abs/2602.01469) — P-EAGLE paper
- [arXiv:2503.01840](https://arxiv.org/abs/2503.01840) — EAGLE-3 paper
- [AWS ML Blog](https://aws.amazon.com/blogs/machine-learning/p-eagle-faster-llm-inference-with-parallel-speculative-decoding-in-vllm/)
- [SGLang Issue #23171](https://github.com/sgl-project/sglang/issues/23171)
- [Red Hat Developer Article](https://developers.redhat.com/articles/2026/04/16/performance-improvements-speculative-decoding-vllm-gpt-oss)
- [JarvisLabs Blog](https://jarvislabs.ai/blog/speculative-decoding-vllm-faster-llm-inference)
