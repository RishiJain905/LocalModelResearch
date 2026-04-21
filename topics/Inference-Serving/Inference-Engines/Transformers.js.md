# Transformers.js (v4)

> **GitHub:** [huggingface/transformers.js](https://github.com/huggingface/transformers.js)  
> **Blog:** [Transformers.js v4 Announcement](https://huggingface.co/blog/transformersjs-v4)  
> **Docs:** [huggingface.co/docs/transformers.js](https://huggingface.co/docs/transformers.js)  
> **NPM:** [@huggingface/transformers](https://www.npmjs.com/package/@huggingface/transformers)

---

## Overview

> *"Transformers.js is designed to be functionally equivalent to Hugging Face's Python transformers library — running in the browser with WebGPU and WASM backends."* — [HuggingFace](https://github.com/huggingface/transformers.js)

**Transformers.js** is Hugging Face's **JavaScript/TypeScript ML library** for browser and Node.js inference. v4 (released March 30, 2026) introduces a new C++ WebGPU runtime via ONNX Runtime, ~4× speedup for embedding models, and a modular architecture.

---

## v4 Key Changes

| Change | Description |
|--------|-------------|
| **New C++ WebGPU Runtime** | ONNX Runtime Web collaboration — custom ops for attention (GQA, MLA), MatMulNBits, QMoE |
| **Universal JS environments** | Same code in browsers, Node.js, Bun, Deno with WebGPU |
| **pnpm monorepo** | Modular packages; `transformers.web.js` is 53% smaller |
| **Build times** | 2s → 200ms (esbuild vs Webpack) |
| **Model architectures** | Mamba (SSM), MLA, MoE support added |

---

## Architecture: WebGPU vs WASM

| Backend | Best For | Throughput (TinyLlama 1.1B) | Cold Start |
|---------|----------|------------------------------|------------|
| **WebGPU** | Large models (>100M params), autoregressive generation | 25-40 tok/s | 1-5s shader compilation |
| **WASM (SIMD)** | Small models (<100M params), single-pass tasks | 2-5 tok/s | Negligible |

> *"Choosing incorrectly between WebGPU and WebAssembly can mean a 10 to 15× performance difference."* — [SitePoint](https://www.sitepoint.com/webgpu-vs-webasm-transformers-js/)

> *"WebGPU introduces marshaling costs — input tensors must be uploaded to GPU buffers and results read back, making WebGPU slower than WASM for small, fast inference passes."* — [SitePoint](https://www.sitepoint.com/webgpu-vs-webasm-transformers-js/)

**WebGPU support:** ~70% global (Chrome 113+, Edge stable, Firefox behind flag, Safari experimental)

---

## vs WebLLM (MLC)

| Dimension | Transformers.js | WebLLM (MLC) |
|-----------|-----------------|--------------|
| **Runtime** | ONNX Runtime Web | Apache TVM/LLVM-based |
| **Approach** | JavaScript-first, ONNX via Optimum | Pre-compiled WASM/Metal/CCUDA |
| **Model format** | ONNX models | MLC-compatible (GGUF, etc.) |
| **Flexibility** | Broader — NLP, Vision, Audio | LLM-focused |
| **Mobile NPUs** | Less optimized | Better mobile NPU support |

> *"WebLLM is purpose-built for LLM inference and can leverage NPUs on some devices; Transformers.js is more general-purpose with broader modality support."* — [Reddit](https://www.reddit.com/r/LocalLLaMA/comments/1lw6jz5/transformersjs_vs_webllm/)

---

## Quantization Options

| Format | Description |
|--------|-------------|
| `fp32` | Full precision (default WebGPU) |
| `fp16` | Half precision |
| `q8` / `int8` / `uint8` | 8-bit (default WASM) |
| `q4` / `bnb4` / `q4f16` | 4-bit quantization |

Models converted via **Hugging Face Optimum** → ONNX format.

---

## v4 New Models

> *"Support for advanced patterns: Mamba (state-space models), Multi-head Latent Attention (MLA), Mixture of Experts (MoE)."* — [HuggingFace Blog](https://huggingface.co/blog/transformersjs-v4)

- LFM2 / LFM2.5 / LFM2-VL / LFM2-MoE (Liquid AI)
- GPT-OSS 20B
- Qwen 3.5, Voxtral (Mistral audio)
- Chatterbox, TranslateGemma, GraniteMoeHybrid

---

## Privacy & Client-Side Use Cases

| Use Case | Description |
|----------|-------------|
| **Privacy-first AI** | No data leaves client — medical, financial, mental health |
| **Offline operation** | WASM with caching enables fully offline inference |
| **Reduced server costs** | All inference on client hardware |
| **RAG in browser** | Combined with vector databases |
| **Speech recognition** | Whisper via browser |
| **Client-side translation** | TranslateGemma, NLLB |

---

## Node.js Support

> *"Same WebGPU backend as browsers when GPU available, or falls back to native ONNX execution."* — [HuggingFace Docs](https://huggingface.co/docs/transformers.js/tutorials/node)

```bash
npm i @huggingface/transformers
```

Supports CommonJS and ECMAScript modules. Models cached to local filesystem (`env.cacheDir`).

---

## How It Works

```
1. Model Export: PyTorch/TensorFlow/JAX → ONNX via 🤗 Optimum CLI
2. Runtime: ONNX Runtime Web executes ONNX graphs
3. Backends:
   - WASM: CPU-based, SIMD-enabled, multi-threaded via Web Workers + SharedArrayBuffer
   - WebGPU: GPU compute via ONNX Runtime WebGPU execution provider
4. API: pipeline() — functionally equivalent to Python transformers.pipeline()
```

---

## GitHub Repos

| Repo | Description |
|------|-------------|
| [huggingface/transformers.js](https://github.com/huggingface/transformers.js) | Main repo — ONNX JS ML library |
| [huggingface/transformers.js-examples](https://github.com/huggingface/transformers.js-examples) | Example applications |
| [huggingface/transformers.js-benchmarking](https://github.com/huggingface/transformers.js-benchmarking) | WASM/WebGPU/Node.js benchmarking |

---

## Papers & Documentation

| Resource | URL |
|----------|-----|
| **v4 Announcement** | [huggingface.co/blog/transformersjs-v4](https://huggingface.co/blog/transformersjs-v4) |
| **Main Docs** | [huggingface.co/docs/transformers.js](https://huggingface.co/docs/transformers.js) |
| **WebGPU Guide** | [huggingface.co/docs/transformers.js/guides/webgpu](https://huggingface.co/docs/transformers.js/guides/webgpu) |
| **Quantization Guide** | [huggingface.co/docs/transformers.js/guides/dtypes](https://huggingface.co/docs/transformers.js/guides/dtypes) |
| **Node.js Tutorial** | [huggingface.co/docs/transformers.js/tutorials/node](https://huggingface.co/docs/transformers.js/tutorials/node) |
| **arXiv Paper** | [arXiv:2407.01972](https://arxiv.org/pdf/2407.01972) |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Best for** | Browser-based inference, privacy-first apps, cross-platform JS deployment |
| **Key advantage** | ONNX-based (broad model support), WebGPU/WASM, ~4× embedding speedup in v4 |
| **vs WebLLM** | More general-purpose; WebLLM better for mobile NPUs |
| **Quantization** | q4/q8/int8/FP16 via ONNX conversion |
| **Node.js** | ✅ Full support with WebGPU fallback |

---

*Last updated: 2026-04-21*
