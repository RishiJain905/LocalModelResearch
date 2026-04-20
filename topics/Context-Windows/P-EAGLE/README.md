# P-EAGLE: Parallel Speculative Drafting

> **Paper:** [arXiv:2602.01469](https://arxiv.org/abs/2602.01469) · February 2026  
> **Authors:** Mude Hui, Xin Huang, Jaime Campos Salas, Yue Sun, Nathan Pemberton, Xiang Song, Ashish Khetan, George Karypis (UC Santa Cruz · AWS Amazon)  
> **Integration:** [vLLM v0.16.0+](https://github.com/vllm-project/vllm/pull/32887) · [AWS Blog](https://aws.amazon.com/blogs/machine-learning/p-eagle-faster-llm-inference-with-parallel-speculative-decoding-in-vllm/)

---

## What Is P-EAGLE?

P-EAGLE (Parallel-EAGLE) transforms EAGLE-style speculative decoding from **autoregressive sequential drafting** to **parallel multi-token prediction**. The core problem it solves:

> *"EAGLE drafts tokens autoregressively: K draft tokens require K forward passes through the draft model. As drafter models improve at longer outputs, this drafting latency scales linearly with speculation depth, capping gains."*  
> — [AWS Blog](https://aws.amazon.com/blogs/machine-learning/p-eagle-faster-llm-inference-with-parallel-speculative-decoding-in-vllm/)

P-EAGLE generates **all K draft tokens in a single forward pass** instead of K sequential passes.

---

## 📚 Wiki Sections

### [🔀 The Shift](./The-Shift.md)
- Sequential → parallel drafting paradigm shift
- Learnable `h_shared` and `mask_token_embedding` as neutral placeholders
- Why 4 transformer layers instead of 1
- Theoretical justification (attention injectivity theorem)

### [⚙️ Implementation](./Implementation.md)
- vLLM v0.16.0+ integration (`parallel_drafting: true`)
- Triton fused kernel for batch metadata rebuilding
- KV cache handling (PADDING_SLOT_ID for rejected tokens)
- TensorRT-LLM support (two-model approach)
- SGLang feature request tracking

### [📊 Speed & Benchmarks](./Speed.md)
- 1.10×–1.36× over EAGLE-3 (paper); up to 1.69× on B200
- EAGLE-3 baseline: up to 6.5× speedup over vanilla decoding
- Context scaling: only P-EAGLE handles 20K (others OOM at 8K)
- End-to-end TPS numbers from SGLang/GPT-OSS 120B

---

## Key Numbers

| Metric | P-EAGLE | EAGLE-3 (baseline) |
|---|---|---|
| Forward passes for K tokens | **1** | K |
| Optimal speculation depth K | **7** | 3 |
| Speedup vs vanilla decoding | ~6–9× | 4–6× |
| Speedup vs EAGLE-3 | 1.10×–1.69× | 1× |
| Long context (20K) | ✅ Scales | ❌ OOM |

---

## 🌐 Official Resources

| Resource | Link |
|---|---|
| **Paper** | [arXiv:2602.01469](https://arxiv.org/abs/2602.01469) |
| **Paper PDF** | [arXiv PDF](https://arxiv.org/pdf/2602.01469) |
| **AWS ML Blog** | [aws.amazon.com/blogs/machine-learning/p-eagle...](https://aws.amazon.com/blogs/machine-learning/p-eagle-faster-llm-inference-with-parallel-speculative-decoding-in-vllm/) |
| **vLLM PR #32887** | [github.com/vllm-project/vllm/pull/32887](https://github.com/vllm-project/vllm/pull/32887) |
| **vLLM Blog Post** | [vllm-project.github.io/_posts/2026-03-13-p-eagle.md](https://github.com/vllm-project/vllm-project.github.io/blob/main/_posts/2026-03-13-p-eagle.md) |
| **SGLang Feature Request** | [github.com/sgl-project/sglang/issues/23171](https://github.com/sgl-project/sglang/issues/23171) |
| **TensorRT-LLM Docs** | [nvidia.github.io/TensorRT-LLM/...](https://nvidia.github.io/TensorRT-LLM/1.2.0rc6/features/speculative-decoding.html) |

### Pre-trained Checkpoints

| Model | HuggingFace |
|---|---|
| GPT-OSS 120B P-EAGLE | [amazon/gpt-oss-120b-p-eagle](https://huggingface.co/amazon/gpt-oss-120b-p-eagle) |
| GPT-OSS 20B P-EAGLE | [amazon/GPT-OSS-20B-P-EAGLE](https://huggingface.co/amazon/GPT-OSS-20B-P-EAGLE) |
| Qwen3-Coder 30B P-EAGLE | [amazon/Qwen3-Coder-30B-A3B-Instruct-P-EAGLE](https://huggingface.co/amazon/Qwen3-Coder-30B-A3B-Instruct-P-EAGLE) |

---

## Related Methods

| Method | Key Difference from P-EAGLE |
|---|---|
| **EAGLE (original)** | Feature-level sequential drafting |
| **EAGLE-3** | Autoregressive multi-token prediction drafter; K sequential passes |
| **PARD** | Target-independent (one draft model for all targets); 4.08× reported |
| **ParallelSpec** | Single-layer; no long-context scaling; missing implementation details |

---

## 🔗 Connected Topics

| Topic | Relevance |
|---|---|
| [KV-Cache-Management](../KV-Cache-Management/) | Speculative decoding is a KV-cache optimization strategy |
| [FlashAttention-4](../FlashAttention-4/) | Hardware-aware kernels relevant to draft verification overhead |
| [Mamba-3](../Mamba-3/) | Alternative inference efficiency approach — SSM vs speculative decoding |

---

## Citation

```bibtex
@article{hui2026peagle,
  title     = {P-EAGLE: Parallel-Drafting EAGLE with Scalable Training},
  author    = {Mude Hui and Xin Huang and Jaime Campos Salas and Yue Sun 
               and Nathan Pemberton and Xiang Song and Ashish Khetan 
               and George Karypis},
  journal   = {arXiv:2602.01469},
  year      = {2026},
  month     = {February}
}
```
