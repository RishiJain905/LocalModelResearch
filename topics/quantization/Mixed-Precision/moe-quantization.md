# MoE Quantization: Mixture-of-Experts Model Compression

> **Key Papers:** MoQE ([arXiv:2310.02410](https://arxiv.org/abs/2310.02410)) · MoEQuant ([arXiv:2505.03804](https://arxiv.org/abs/2505.03804)) · MoPEQ ([arXiv:2509.02512](https://arxiv.org/abs/2509.02512)) · QuantMoE-Bench ([arXiv:2406.08155](https://arxiv.org/html/2406.08155v2)) · GEMQ (ICLR 2026)  
> **GitHub:** [UNITES-Lab/moe-quantization](https://github.com/UNITES-Lab/moe-quantization) · [chenzx921020/MoEQuant](https://github.com/chenzx921020/MoEQuant) · [vllm-project/llm-compressor](https://github.com/vllm-project/llm-compressor/blob/main/examples/quantizing_moe/README.md)  
> **Benchmarks:** [QuantMoE-Bench](https://arxiv.org/html/2406.08155v2) · [pprp/Awesome-Efficient-MoE](https://github.com/pprp/Awesome-Efficient-MoE)

---

## Overview

> *"MoE models challenge conventional quantization: sparse activation patterns mean only some experts are active at inference, calibration data may never visit less-used experts, and the router's precision sensitivity differs from expert weights."* — QuantMoE-Bench (arXiv:2406.08155)

**MoE (Mixture of Experts)** quantization is fundamentally different from dense LLM quantization due to: (1) **sparse activation** — only top-K experts are active per token, (2) **expert imbalance** — some experts are activated far more than others, (3) **router precision sensitivity** — routing decisions affect which experts compute, (4) **shared vs routed experts** — some experts are always active while others are conditionally selected.

---

## MoE Architecture Basics

### MoE vs Dense

| Aspect | Dense LLM | MoE LLM |
|--------|-----------|---------|
| **Parameters** | All active per token | Only selected experts active |
| **Example** | LLaMA-2-7B | Mixtral-8x7B (~46.7B total) |
| **Active params** | ~7B | ~12.9B (top-2 of 8 experts) |
| **Expert count** | 1 (all layers) | 8 per layer (Mixtral) |
| **Memory for weights** | All weights loaded | All weights stored, subset active |

### Key MoE Models & Architecture

| Model | Total Params | Experts | Top-K | Shared Expert |
|-------|-------------|---------|-------|--------------|
| **Mixtral-8x7B** | ~46.7B | 8 | 2 | No |
| **DeepSeek-MoE-16B** | ~16.4B | 64 routed + 2 shared | 6 | Yes |
| **DBRX (Databricks)** | ~132B | 16 | 4 | No |
| **Grok-1 (xAI)** | ~314B | 8 | 2 | No |
| **OLMoE-1B-7B** | ~1B active / 7B total | 8 | 2 | No |

---

## The MoE Quantization Problem

### Why Standard PTQ Fails on MoE

| Problem | Description | Impact |
|---------|-------------|--------|
| **Expert imbalance** | Some experts activated 100× more than others | Less-used experts get poor calibration |
| **Router sensitivity** | Quantizing router weights changes routing decisions | Entire downstream expert selection changes |
| **Shared experts** | Always-active experts handle ALL tokens | Need higher precision than routed experts |
| **Calibration gap** | Calibration data may not trigger all experts | Expert coverage is incomplete |

> *"Standard calibration data causes inter-expert imbalance — less-used experts get insufficient calibration, leading to severe quality degradation on the underrepresented expert pathways."* — MoEQuant (arXiv:2505.03804)

---

## Key Methods

### 1. MoQE: Mixture of Quantized Experts (2023)

> **arXiv:** [2310.02410](https://arxiv.org/abs/2310.02410)

**Key contribution:** Ultra low-bit (2-bit) weight-only quantization for MoE models.

> *"MoQE demonstrates that MoE models can survive more aggressive quantization than dense models because of their sparse activation — quantization errors in inactive experts don't affect inference."*

| Aspect | Details |
|--------|---------|
| **Target** | 2-bit weight-only quantization per expert |
| **Key insight** | Sparse activation limits error propagation |
| **Results** | Viable 2-bit per expert on Mixtral-class models |

---

### 2. MoEQuant (2025)

> **arXiv:** [2505.03804](https://arxiv.org/abs/2505.03804) · **GitHub:** [chenzx921020/MoEQuant](https://github.com/chenzx921020/MoEQuant)

**Key contribution:** Two-part solution addressing both inter-expert and intra-expert imbalance:

| Component | Full Name | Purpose |
|-----------|-----------|---------|
| **EBSS** | Expert-Balanced Self-Sampling | Data-free calibration using LLM's own vocabulary |
| **AGQ** | Affinity-Guided Quantization | Token-expert affinity in Hessian space |

> *"EBSS generates synthetic calibration data that ensures all experts receive balanced activation during calibration. AGQ uses token-expert affinity information from the Hessian to guide per-expert precision allocation."*

---

### 3. MoPEQ: Mixture of Mixed-Precision Quantized Experts (ICCV 2025)

> **arXiv:** [2509.02512](https://arxiv.org/abs/2509.02512)

**Key contribution:** Assigns **optimal bit-width (2/3/4-bit)** to each expert using **Hessian trace approximation** + K-means clustering for model-wise precision assignment.

> *"MoPEQ treats each expert as an independent quantization unit. Experts that are more sensitive (high Hessian trace) get higher precision."*

| Result | Details |
|--------|---------|
| **Compression** | ~1.5× model size reduction |
| **Quality** | Within 5% accuracy of full-precision |

---

### 4. GEMQ: Global Expert-level Mixed-Precision (ICLR 2026)

> **arXiv:** (under review) | **Status:** ICLR 2026

**Key contribution:** Addresses how **expert quantization alters router behavior** — a critical issue previous methods ignored.

> *"When you quantize an expert, you change its effective output distribution. This changes the router's softmax inputs, which can alter routing decisions. GEMQ models this coupling between router and expert quantization."*

---

### 5. QuantMoE-Bench: Comprehensive MoE PTQ Benchmark (2024)

> **arXiv:** [2406.08155v2](https://arxiv.org/html/2406.08155v2)

**Key contribution:** Comprehensive benchmark for MoE post-training quantization. **Critical finding:**

> *"Shared experts need MORE bits (4-bit) compared to token-conditioned (routed) experts (2-bit). Rationale: shared experts are accessed by ALL tokens, so their errors affect every forward pass. Routed experts only affect the subset of tokens that select them."*

| Expert Type | Recommended Bits | Reason |
|-------------|----------------|--------|
| **Shared expert** | 4-bit | Always active, accessed by all tokens |
| **Routed expert** | 2-bit | Conditionally active, sparse access |

---

### 6. DynaExq: Dynamic Expert Quantization (2025)

> **arXiv:** [2511.15015](https://arxiv.org/abs/2511.15015)

**Key contribution:** **Long-horizon expert hotness estimation** with asynchronous promotions/demotions.

> *"Experts that are frequently activated get promoted to higher precision; rarely-used experts are demoted. This is done asynchronously during inference."*

| Mechanism | Description |
|-----------|-------------|
| **Expert hotness** | Track activation frequency over long context windows |
| **Promotion** | Rarely-used experts: 2-bit → 4-bit |
| **Demotion** | Frequently-used experts: 4-bit → 2-bit |
| **Async** | Promotion/demotion happens without blocking inference |

---

### 7. EAQuant: Enhancing PTQ for MoE via Activation Outliers (2025)

> **arXiv:** [2506.13329v3](https://arxiv.org/html/2506.13329v3)

**Key contribution:** Addresses **activation outliers** in MoE that standard PTQ methods don't handle well.

> *"MoE models have activation patterns that differ from dense models — per-token activation variance is much higher due to routing. EAQuant specifically targets these outlier patterns."*

---

### 8. DeepSeek-V3 Technical Report: FP8 Mixed Precision (2024)

> **arXiv:** [2412.19437](https://arxiv.org/html/2412.19437v1)

**Key contribution:** **FP8 mixed precision training** for MoE with online quantization — not pure PTQ but relevant for MoE quantization.

> *"DeepSeek-V3 uses FP8 for most activations and weights, with higher precision for shared experts and router. The architecture has 1 shared expert + 256 routed experts per MoE layer."*

---

## Per-Expert vs Shared Expert Quantization

### Why Shared Experts Need More Bits

```
DeepSeek-V3 architecture:
- 1 shared expert: ALWAYS active → must handle ALL tokens with high quality
- 256 routed experts: CONDITIONALLY active → sparsity limits error impact

Mixtral architecture:
- 8 routed experts: All similar activation frequency (balanced routing)
- No shared expert → all experts can use similar bit allocation
```

| Model | Shared Expert | Routed Expert | Bit Difference |
|-------|--------------|--------------|--------------|
| **DeepSeek-MoE** | Yes (1) | 64 | Shared: 4-bit, Routed: 2-bit |
| **Mixtral** | No | 8 | Balanced → uniform allocation |
| **DBRX** | No | 16 | All similar frequency |

---

## Calibration Strategies for MoE

### The Expert Coverage Problem

Standard calibration: use a generic dataset (e.g., Pile samples)

```
Problem: Calibration on generic text
→ ~20% of experts never get activated during calibration
→ These experts get poor quantization parameters
→ Quality degrades when they ARE activated at inference
```

### Solutions

| Method | Approach | Paper |
|--------|---------|-------|
| **EBSS** | Generate synthetic data from LLM's own vocabulary to balance expert activation | MoEQuant |
| **All-tokens routing** | During calibration, route ALL tokens through ALL experts (even if not normally selected) | llm-compressor |
| **Expert frequency weighting** | Weight calibration samples by expert activation frequency | QuantMoE-Bench |
| **Hessian-guided** | Use token-expert affinity from Hessian to guide per-expert precision | MoEQuant, MoPEQ |

### llm-compressor's Approach

```python
# llm-compressor calibration for MoE
# Route ALL tokens through ALL experts during calibration
# even those not normally selected by the router

class MoECalibration:
    def calibration_forward(self, x):
        # Force all experts to be active
        for expert in self.experts:
            expert_output = expert(x)
        # Average or accumulate all expert outputs
        # This ensures all experts get calibration data
```

---

## Model-Specific Notes

### Mixtral-8x7B

| Aspect | Details |
|--------|---------|
| **Architecture** | 8 experts, top-2 routing, ~46.7B total params |
| **Routing** | Balanced — all experts have similar activation frequency |
| **Standard PTQ** | Works well — balanced routing makes expert frequency less useful as a signal |
| **Recommended** | Per-expert uniform precision (all experts same bits) |
| **Quantization** | INT8 or INT4 via standard PTQ works fine |

### DeepSeek-MoE-16B

| Aspect | Details |
|--------|---------|
| **Architecture** | 64 routed + 2 shared experts, top-6 routing, ~16.4B total |
| **Routing** | Highly unbalanced — expert frequency is highly predictive |
| **Shared experts** | 1 shared expert is ALWAYS active → needs 4-bit |
| **Routed experts** | Top-6 of 64 → only ~9% activation rate per expert |
| **Recommended** | Shared: 4-bit, Routed: 2-bit (per QuantMoE-Bench) |

### DBRX

| Aspect | Details |
|--------|---------|
| **Architecture** | Fine-grained MoE, 16 experts, top-4 selection, 36B active / 132B total |
| **Expert count** | Higher expert-to-token ratio than Mixtral |
| **Quantization** | Per-expert mixed precision recommended |

### Grok-1

| Aspect | Details |
|--------|---------|
| **Architecture** | 314B params, 8 experts, top-2 selection |
| **Released weights** | Some weights quantized to 8-bit in official release |
| **Quantization** | W8A8 works, per-expert INT4 also viable |

---

## Framework Support

### vLLM / llm-compressor

```python
# vLLM MoE quantization example
# See: github.com/vllm-project/llm-compressor/blob/main/examples/quantizing_moe/README.md

from llm_compressor import parse_recursive

# Calibration-friendly MoE: routes all tokens through all experts
# This ensures all experts are properly calibrated
```

### HuggingFace Transformers

```python
# MixtralForCausalLM with BitsAndBytes
from transformers import BitsAndBytesConfig

quantization_config = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_compute_dtype="float16"
)

model = AutoModelForCausalLM.from_pretrained(
    "mistralai/Mixtral-8x7B-v0.1",
    quantization_config=quantization_config
)
```

---

## Papers Table

| Paper | Year | Venue | arXiv | Key Contribution |
|-------|------|-------|-------|-----------------|
| **MoQE** | 2023 | — | [2310.02410](https://arxiv.org/abs/2310.02410) | 2-bit ultra-low-bit MoE quantization |
| **Fast MoE Inference w/ Offloading** | 2023 | — | [2312.17238](https://arxiv.org/abs/2312.17238) | Mixed quantization for desktop Mixtral |
| **QuantMoE-Bench** | 2024 | — | [2406.08155v2](https://arxiv.org/html/2406.08155v2) | Benchmark: shared experts need 4-bit, routed 2-bit |
| **MoEQuant** | 2025 | — | [2505.03804](https://arxiv.org/abs/2505.03804) | EBSS + AGQ for balanced calibration |
| **MoPEQ** | 2025 | ICCV | [2509.02512](https://arxiv.org/abs/2509.02512) | Per-expert bit-width via Hessian trace |
| **DynaExq** | 2025 | — | [2511.15015](https://arxiv.org/abs/2511.15015) | Dynamic expert precision residency |
| **EAQuant** | 2025 | — | [2506.13329v3](https://arxiv.org/html/2506.13329v3) | Activation outlier handling for MoE |
| **GEMQ** | 2026 | ICLR | (under review) | Router-expert quantization coupling |
| **DeepSeek-V3** | 2024 | — | [2412.19437](https://arxiv.org/html/2412.19437v1) | FP8 mixed precision for MoE |

---

## GitHub Repositories

| Repo | Description |
|------|-------------|
| [UNITES-Lab/moe-quantization](https://github.com/UNITES-Lab/moe-quantization) | Benchmark for MoE PTQ — built on AutoGPTQ, supports Mixtral & DeepSeek |
| [chenzx921020/MoEQuant](https://github.com/chenzx921020/MoEQuant) | MoEQuant implementation (EBSS + AGQ) |
| [vllm-project/llm-compressor](https://github.com/vllm-project/llm-compressor/blob/main/examples/quantizing_moe/README.md) | Official MoE quantization examples — AWQ for GLM-4.7, FP8 for Mixtral |
| [Taishi-N324/Drop-Upcycling](https://github.com/Taishi-N324/Drop-Upcycling) | Drop-Upcycling — training sparse MoE with partial experts |
| [allenai/OLMoE](https://github.com/allenai/OLMoE) | Sparse Upcycling implementation for MoE |
| [pprp/Awesome-Efficient-MoE](https://github.com/pprp/Awesome-Efficient-MoE) | Curated list of efficient MoE methods including quantization |

---

## Blog Posts & Technical Reports

| Title | Source | URL |
|-------|--------|-----|
| **"DeepSeek Model Architecture"** | Fireworks AI | [Link](https://fireworks.ai/blog/deepseek-model-architecture) |
| **"Mixtral TensorRT-LLM Quantization"** | Baseten | [Link](https://www.baseten.co/blog/faster-mixtral-inference-with-tensorrt-llm-and-quantization/) |
| **"Running Quantized Mixtral on a Single GPU"** | FriendliAI | [Link](https://medium.com/friendliai/running-quantized-mixtral-8x7b-on-a-single-gpu-555c012c73eb) |
| **"Quantizing Mixtral: Performance Analysis"** | Medium | [Link](https://medium.com/@ingridwickstevens/quantizing-mixtral-exploring-performance-and-challenges-across-diverse-prompts-671f0c49ddc7) |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Core difference from dense** | Sparse activation, expert imbalance, router sensitivity, shared vs routed experts |
| **Key finding (QuantMoE-Bench)** | Shared experts need 4-bit, routed experts can use 2-bit |
| **Calibration challenge** | Less-used experts get poor calibration — EBSS and all-token routing solve this |
| **MoEQuant** | EBSS (balanced synthetic data) + AGQ (Hessian-guided precision) |
| **MoPEQ** | Per-expert bit-width via Hessian trace + K-means |
| **Mixtral** | Balanced routing → standard PTQ works fine |
| **DeepSeek-MoE** | Unbalanced routing → shared (4-bit) vs routed (2-bit) allocation |
| **Best for** | Serving large MoE models (Mixtral, DBRX, DeepSeek-MoE) within memory budgets |

---

*Last updated: 2026-04-19*
