# EXL3 (ExLlamaV3): Ultra-Low-Bit LLM Inference

> **GitHub:** [turboderp-org/exllamav3](https://github.com/turboderp-org/exllamav3) · [ExLlamaV3 Documentation](https://github.com/turboderp-org/exllamav3/blob/master/doc/exl3.md)  
> **Foundation:** QTIP — Trellis-Coded Quantization ([NeurIPS 2024](https://arxiv.org/abs/2402.04396))  
> **Backend API:** [TabbyAPI](https://github.com/theroyallab/tabbyAPI/) (OpenAI-compatible API server)  
> **Model Collection:** [huggingface.co/collections/turboderp/exl3-models](https://huggingface.co/collections/turboderp/exl3-models) (53+ models)

---

## Overview

> *"EXL3 is a variant of QTIP — Quantization with Trellis and Incoherence Processing. It uses tail-biting trellis structures for encoding high-dimensional vectors, enabling ultra-high-dimensional quantization far beyond the ≤8 dim limit of standard VQ-based methods."* — [exl3.md](https://github.com/turboderp-org/exllamav3/blob/master/doc/exl3.md)

**ExLlamaV3 (EXL3)** is a high-performance inference engine for **arbitrary-bit-per-weight (2–8 bpw)** quantization of LLMs. It is NOT a quantization algorithm itself — it implements the QTIP (Trellis-Coded Quantization) format with modifications for efficient inference.

**Key differentiator from GGUF:** GGUF uses fixed quantization steps (Q2_K, Q4_K, Q5_K, etc.). EXL3 allows **arbitrary bits-per-weight** from 2.0 to 8.0 bpw, enabling fine-grained memory/quality tradeoffs.

---

## How EXL3 Differs from GGUF (llama.cpp)

| Aspect | **GGUF (llama.cpp)** | **EXL3 (ExLlamaV3)** |
|--------|---------------------|---------------------|
| **Quantization type** | Scalar per-channel/per-group | Trellis-coded vector quantization |
| **Bit granularity** | Fixed (Q2/3/4/5/6_K, IQ variants) | **Arbitrary 2–8 bpw** |
| **Encoding** | Lookup table | Hybrid lookup + computed codes ("bitshift" trellis) |
| **Quality at low bits** | IQ2_XS, IQ3_XXS | Optimized for 2–4 bpw with better tail distribution |
| **Throughput** | Baseline | **~1.5–2× faster** than GGUF on NVIDIA GPU |
| **Hardware** | CPU, Apple Silicon, NVIDIA | NVIDIA GPU (CUDA) |
| **Inference engine** | llama.cpp | ExLlamaV3 |
| **VRAM at same bits** | Baseline | Slightly lower at equivalent quality |

> *"EXL3 achieves lower perplexity than GGUF at equivalent bitrates, and higher throughput on NVIDIA GPUs. The tradeoff is compatibility — GGUF runs everywhere, EXL3 requires NVIDIA GPU + ExLlamaV3."*

---

## QTIP: The Foundation (NeurIPS 2024)

EXL3 is built on **QTIP** — "QTIP: Trellis-Coded Quantization with Incoherence Processing" from Cornell-RelaxML.

> **Paper:** [arXiv:2402.04396](https://arxiv.org/abs/2402.04396) · **Code:** [Cornell-RelaxML/qtip](https://github.com/Cornell-RelaxML/qtip)

### Key Innovation

QTIP uses **trellis-coded quantization** with Hadamard incoherence processing:

```
Standard VQ: Codebook lookup for each vector
QTIP: Trellis structure where encoding = path through state machine
→ Each step produces variable bits
→ Tail-biting trellis handles arbitrary-dim vectors
→ Codebook is procedural — no storage cost
```

### QTIP Performance (from paper)

| Method | Bits | Throughput (tok/s) |
|--------|------|-------------------|
| FP16 | 16 | 55.9 |
| AQLM 2-bit | 2 | 81.5 |
| QuIP# 2-bit | 2 | 186 |
| **QTIP 2-bit** | 2 | **188** |
| QTIP 3-bit | 3 | 161 |
| QTIP 4-bit | 4 | 140 |

> QTIP achieves the best throughput at 2-bit among all methods while maintaining near-FP16 quality.

---

## Quantization Format

### Arbitrary Bits-Per-Weight

EXL3 supports **any bitrate from 2.0 to 8.0 bpw** — not just discrete steps:

| Model Example | Available Bitrates |
|--------------|-------------------|
| **Qwen3-30B-A3B-exl3** | 2.25, 3.00, 4.00, 5.00, 6.00, 8.00 bpw |
| **Llama 3.1 70B-exl3** | 2.5, 3.0, 3.5, 4.0, 4.5, 5.0, 6.0 bpw |

This enables precise memory/quality calibration — if you need exactly 4.1GB, you can set exactly 2.65 bpw on a 70B model.

---

## Performance Benchmarks

### Speed: EXL2 vs GGUF

> *"EXL2 was ~85% faster than llama.cpp for token generation on equivalent hardware."* — Community benchmarks

| Format | Perplexity (Wikitext-2) | Speed | VRAM |
|--------|------------------------|-------|------|
| **EXL2-4.65bpw** | 4.307 | **52 tok/s** | 9.3GB |
| Q4_K_M.gguf | 4.333 | 31 tok/s | 9.0GB |
| **Speed ratio** | Similar quality | **+68% faster** | Similar |

### Quality: HumanEval (Qwen3-30B-A3B)

| Bits Per Weight | EXL3 HumanEval | Notes |
|-----------------|---------------|-------|
| 2.25 bpw | 88.41% | Extreme compression |
| 3.00 bpw | 89.63% | "Bump" region — best quality/perf |
| 4.00 bpw | 92.07% | Recommended |
| 5.00 bpw | 93.29% | Near-optimal |
| FP16 baseline | 91.46% | Reference |

> *"HumanEval shows a repeatable 'bump' around 3 bpw that is statistically significant. At 2.25 bpw, KL-divergence vs FP16 is 0.1416; at 3 bpw it drops to 0.0688."*

---

## Supported Models

ExLlamaV3 supports 50+ model architectures:

| Family | Models |
|--------|--------|
| **Llama** | Llama 2/3/3.1/3.2, Llama 3.1-Nemotron (253B), Llama 3.3-Nemotron (49B) |
| **Qwen** | Qwen2/2.5, Qwen3 (8B–235B), Qwen3-VL, Qwen3.5, Qwen3-Next |
| **Mistral** | Mistral 7B, Mistral-Large, Mixtral 8x7B, Ministral 3, Mistral-Small 3.1 |
| **Gemma** | Gemma 2 (9B), Gemma 3 (27B), Gemma 4 (26B–31B) |
| **GLM** | GLM-4, GLM-4.5, GLM-4.6, GLM-4.1V, GLM-4.5V |
| **Command-R** | Cohere Command-R, Command-R7B, Command-R+ |
| **MoE** | Qwen3-30B-A3B, Qwen3.5-35B-A3B, Mixtral, ERNIE 4.5 (300B) |
| **Vision-Language** | Qwen3-VL, GLM-4.1V, GLM-4.5V, Gemma 3 multimodal |
| **Other** | Phi-4-mini, SmolLM3, EXAONE 4.0, Devstral 2, Apertus |

**Model collection:** [huggingface.co/collections/turboderp/exl3-models](https://huggingface.co/collections/turboderp/exl3-models)

---

## Quality vs Speed Tradeoffs

| Bits Per Weight | Quality (relative) | Speed | Best For |
|----------------|--------------------|-------|---------|
| 8 bpw | ~99%+ | Baseline | Maximum quality |
| 6 bpw | ~98–99% | ~1.1× | High quality |
| 4–5 bpw | ~95–97% | ~1.5–2× | **Recommended production** |
| 3 bpw | ~90–94% | ~2–3× | VRAM-constrained |
| 2–2.5 bpw | ~85–90% | ~3×+ | Extreme compression |

---

## Framework Support

### ExLlamaV3 Backend (standalone)

```bash
# Clone and install
git clone https://github.com/turboderp-org/exllamav3
cd exllamav3

# Run inference
python3 example.py -m model-directory -c model.gguf
```

### TabbyAPI (OpenAI-compatible API)

```bash
# TabbyAPI is the official ExLlamaV3 API server
# github.com/theroyallab/tabbyAPI
pip install tabbyapi
python3 run.py --model /path/to/exl3/model
```

### HuggingFace Transformers Plugin

```python
# Via HF Transformers (limited)
from transformers import AutoModelForCausalLM
model = AutoModelForCausalLM.from_pretrained(
    "turboderp/Qwen3-30B-A3B-exl3",
    torch_dtype=torch.float16
)
```

---

## Limitations & Notes

| Issue | Details |
|-------|---------|
| **NVIDIA only** | No CPU, Apple Silicon, or AMD support |
| **Model availability** | Only ~5% of models on HF are EXL3 format |
| **Tool ecosystem** | Limited to ExLlamaV3/TabbyAPI — no Ollama, GPT4All, etc. |
| ** llama.cpp parity** | Some reports (2025) suggest llama.cpp has caught up in speed on certain hardware |
| **Flash Linear Attention** | For Qwen3-Next/Qwen3.5: requires Triton, JIT compilation can be "shaky" |

---

## GitHub Repositories

| Repo | Description |
|------|-------------|
| [turboderp-org/exllamav3](https://github.com/turboderp-org/exllamav3) | **Main repo** — 739 stars, MIT license, v0.0.28 (March 2026) |
| [theroyallab/tabbyAPI](https://github.com/theroyallab/tabbyAPI) | OpenAI-compatible API server backed by ExLlamaV3 |
| [Cornell-RelaxML/qtip](https://github.com/Cornell-RelaxML/qtip) | QTIP foundation — Trellis-coded quantization paper |
| [turboderp/exllamav2](https://github.com/turboderp-org/exllamav2) | EXL2 (predecessor) — still used for existing quantizations |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Type** | Inference engine + quantization format (QTIP-based) |
| **Bit granularity** | Arbitrary 2–8 bpw (not fixed steps) |
| **Speed** | ~1.5–2× faster than GGUF on NVIDIA GPU |
| **Quality** | Lower perplexity than GGUF at equivalent bitrates |
| **Hardware** | NVIDIA GPU only (CUDA) |
| **Models** | 50+ architectures supported |
| **Best for** | Power users with NVIDIA GPUs wanting maximum throughput and fine-grained bitrate control |
| **Foundation** | QTIP (NeurIPS 2024) — trellis-coded vector quantization |

---

*Last updated: 2026-04-19*
