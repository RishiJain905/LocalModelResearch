# BitsAndBytes NF4 & QLoRA

> **QLoRA Paper:** [arXiv:2305.14314](https://arxiv.org/abs/2305.14314) — NeurIPS 2023  
> **LLM.int8() Paper:** [arXiv:2208.07339](https://arxiv.org/abs/2208.07339)  
> **Main Library:** [bitsandbytes-foundation/bitsandbytes](https://github.com/bitsandbytes-foundation/bitsandbytes)

---

## Overview

> *"We present QLoRA, an efficient finetuning approach that reduces memory usage enough to finetune a 65B parameter model on a single 48GB GPU while preserving full 16-bit finetuning task performance."*

> *"bitsandbytes enables accessible large language models via k-bit quantization for PyTorch."*

---

## Papers

| Paper | Venue | arXiv |
|-------|-------|-------|
| **QLoRA: Efficient Finetuning of Quantized LLMs** | NeurIPS 2023 | [arXiv:2305.14314](https://arxiv.org/abs/2305.14314) |
| **LLM.int8(): 8-bit Matrix Multiplication** | — | [arXiv:2208.07339](https://arxiv.org/abs/2208.07339) |
| **8-bit Optimizers via Block-wise Quantization** | ICLR 2022 | — |

---

## GitHub Repositories

### bitsandbytes Library

| | |
|---|---|
| **Repo** | [bitsandbytes-foundation/bitsandbytes](https://github.com/bitsandbytes-foundation/bitsandbytes) |
| **Stars** | 8.1k | **Forks** | 842 | **Used by** | 36.1k |
| **License** | MIT |
| **Latest Release** | 0.49.2 (Feb 2026) |

### QLoRA Repository

| | |
|---|---|
| **Repo** | [artidoro/qlora](https://github.com/artidoro/qlora) |
| **Stars** | 10.9k | **Forks** | 870 |

---

## How It Works: NF4 Data Type

### 4-bit NormalFloat (NF4)

**Core Concept:** NF4 is a non-uniform 4-bit quantization scheme where quantization levels are placed at the quantile boundaries of a standard normal distribution N(0,1).

> *"NF4 uses k-bit quantile quantization — estimates 2^b equally spaced quantile values from the quantiles of a normal distribution using `scipy.stats.norm.ppf` (inverse CDF), then normalizes to [-1, 1]."*

**Quantile Formula:**
```
q_i = (1/2)[Q(i/17) + Q((i+1)/17)]
```

**NF4 Quantization Levels (16 predefined values):**
```python
NF4_quant_levels = [
    -1.0, -0.6961928009986877, -0.5250730514526367,
    -0.39491748809814453, -0.28444138169288635, -0.18477343022823334,
    -0.09105003625154495, 0.0, 0.07958029955625534, 0.16093020141124725,
    0.24611230194568634, 0.33791524171829224, 0.44070982933044434,
    0.5626170039176941, 0.7229568362236023, 1.0
]
```

### Why NF4 Outperforms Uniform INT4

| Property | Uniform INT4 | NF4 |
|----------|-------------|-----|
| Spacing | Equal intervals | Denser near zero |
| Tail coverage | Uneven | Properly covers tails |
| Outlier handling | Poor | Better for normal distributions |
| **Perplexity difference** | Baseline | ~0.5 points better |

### Quantization Process (Block-wise)

1. **Flatten** weight matrix to 1D sequence
2. **Split** into blocks of 64 values
3. **Normalize** each block by absolute maximum (absmax scaling) → values fit [-1, 1]
4. **Map** each normalized value to nearest NF4 quantization level
5. **Store** absmax as FP16 metadata for dequantization

> *"4Q is a compression technique, not a true 4-bit data type. Compressed weights must be decompressed back to 16-bit before use."*

### Double Quantization

> *"Double quantization reduces the average memory footprint by quantizing the quantization constants."*

**Process:**
1. Quantize weights to INT4 with per-group FP16 scales
2. Group the FP16 scales (typically groups of 256)
3. Quantize scales to **INT8** with per-group FP32 meta-scales

**Memory Savings from Double Quantization:**

| Model Size | Memory Saved |
|------------|-------------|
| 8B params | ~3 GB |
| 70B params | ~26 GB |

> *"Double quantization reduces from ~4.5 to ~4.13 bits/weight — saves >3 GB for a 70B model."*

---

## QLoRA: Fine-tuning with NF4

### Architecture

```
Frozen 4-bit NormalFloat Base Weights
         ↓
Trainable LoRA Adapters (16-bit)
         ↓
Gradient Backpropagation Through Quantized Model
```

- **Frozen 4-bit quantized base weights** (NF4)
- **Trainable LoRA adapters** (16-bit, injected into attention layers)
- **Storage dtype:** 4-bit NormalFloat for base weights
- **Compute dtype:** 16-bit BrainFloat for forward/backward passes

> *"LoRA on all linear transformer block layers are required to match full finetuning performance."* — QLoRA Paper, pg. 6

### Key Results

| Model | Benchmark | ChatGPT | GPT-4 | QLoRA (Guanaco) |
|-------|-----------|---------|-------|-----------------|
| 65B | Vicuna | 68.7% | 47.6% | **67.6%** |
| 33B | Vicuna | 63.4% | 46.2% | **71.9%** |
| 13B | Vicuna | 52.8% | 37.4% | **62.2%** |

> *"QLoRA achieves 99.3% of ChatGPT performance on Vicuna benchmark."*

> *"65B model finetuned in 24 hours on a single GPU."*

---

## Performance Benchmarks

### Comparison Table

| Method | Best For | Quality (4-bit) | Speed (vLLM) | CPU Support | Training |
|--------|----------|-----------------|--------------|-------------|----------|
| **GGUF** | CPU/Ollama | 92% (Q4_K_M) | 93 tok/s | ✅ Yes | ❌ |
| **AWQ** | vLLM production | 95% | 741 tok/s | ❌ | ❌ |
| **GPTQ** | GPU inference | 90% | 712 tok/s | ❌ | ❌ |
| **bitsandbytes NF4** | Training/QLoRA | 95%+ | 168 tok/s | ❌ | ✅ |

### Quality vs Speed (HumanEval via PremAI)

| Method | Throughput | Quality (Pass@1) |
|--------|-----------|-------------------|
| FP16 Baseline | 461 tok/s | 56.1% |
| **AWQ + Marlin** | **741 tok/s** | **51.8%** |
| GPTQ + Marlin | 712 tok/s | 46.3% |

> *"AWQ without an optimized kernel is actually slower than FP16. With Marlin, it's 1.6x faster than baseline."*

### Memory Comparison (Training)

| Stage | Memory |
|-------|--------|
| FP32 (original) | 0.48 GB |
| FP16 | 0.24 GB |
| NF4 Quantized | 0.12 GB |
| After `prepare_model_for_kbit_training` | 0.20 GB |

---

## Practical Usage

### Installation

```bash
pip install -q -U bitsandbytes
pip install -q -U git+https://github.com/huggingface/transformers.git
pip install -q -U git+https://github.com/huggingface/peft.git
pip install -q -U git+https://github.com/huggingface/accelerate.git
```

### Simple 4-bit Loading

```python
from transformers import AutoModelForCausalLM

model = AutoModelForCausalLM.from_pretrained(
    "facebook/opt-350m", 
    load_in_4bit=True, 
    device_map="auto"
)
```

### Full BitsAndBytesConfig

```python
from transformers import BitsAndBytesConfig
import torch

compute_dtype = torch.bfloat16 if torch.cuda.is_bf16_supported() else torch.float16

nf4_config = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_quant_type="nf4",           # NF4 (default, recommended) or "fp4"
    bnb_4bit_use_double_quant=True,       # Saves 0.4 bits/parameter
    bnb_4bit_compute_dtype=compute_dtype  # 16-bit for faster training
)

model = AutoModelForCausalLM.from_pretrained(
    model_id, 
    quantization_config=nf4_config
)
```

### QLoRA Fine-tuning with PEFT

```python
from peft import prepare_model_for_kbit_training, LoraConfig, get_peft_model

# Prepare quantized model for training
model = prepare_model_for_kbit_training(model)

# Configure LoRA
config = LoraConfig(
    r=16,
    lora_alpha=32,
    lora_dropout=0.1,
    task_type=TaskType.CAUSAL_LM,
    target_modules=['q_proj', 'k_proj', 'v_proj', 'o_proj'],
    bias="none",
)

# Get trainable model
model = get_peft_model(model, config)
```

### Training Arguments

```python
training_args = TrainingArguments(
    model_name,
    eval_strategy="epoch",
    save_strategy="epoch",
    learning_rate=2e-5,
    per_device_train_batch_size=224,
    per_device_eval_batch_size=224,
    num_train_epochs=3,
    weight_decay=0.01,
    lr_scheduler_type="cosine",
    warmup_ratio=0.1,
    fp16=True,
    load_best_model_at_end=True,
    metric_for_best_model="accuracy",
    push_to_hub=True,
    optim="paged_adamw_32bit"  # Paged optimizer for memory management
)
```

---

## Supported Architectures

```
['bigbird_pegasus', 'blip_2', 'bloom', 'bridgetower', 'codegen', 'deit', 'esm',
'gpt2', 'gpt_bigcode', 'gpt_neo', 'gpt_neox', 'gpt_neox_japanese', 'gptj', 'gptsan_japanese',
'lilt', 'llama', 'longformer', 'longt5', 'luke', 'm2m_100', 'mbart', 'mega', 'mt5', 'nllb_moe',
'open_llama', 'opt', 'owlvit', 'plbart', 'roberta', 'roberta_prelayernorm', 'rwkv', 'switch_transformers',
't5', 'vilt', 'vit', 'vit_hybrid', 'whisper', 'xglm', 'xlm_roberta']
```

---

## Hardware Requirements

| Feature | Minimum Hardware |
|---------|-----------------|
| 8-bit optimizers | NVIDIA Pascal (GTX 10X0, P100) or newer |
| LLM.int8() | **NVIDIA Turing (RTX 20X0, T4) or newer** |
| NF4/FP4 quantization | NVIDIA Pascal (GTX 10X0, P100) or newer |

---

## Blog Posts & Technical Explanations

| Source | Link |
|--------|------|
| **Hugging Face Blog** | [Making LLMs even more accessible with bitsandbytes, 4-bit quantization and QLoRA](https://huggingface.co/blog/4bit-transformers-bitsandbytes) |
| **HuggingFace Docs** | [BitsAndBytes Documentation](https://huggingface.co/docs/transformers/en/quantization/bitsandbytes) |
| **Manal El Aidouni** | [Mastering QLoRa: A Deep Dive](https://manalelaidouni.github.io/4Bit-Quantization-Models-QLoRa.html) |
| **MaximoFN Blog** | [QLoRA Explained: NF4, Double Quantization & Paged Optimizers](https://www.maximofn.com/en/qlora/) |
| **Chris McCormick** | [QLoRA and 4-bit Quantization](https://mccormickml.com/2024/09/14/qlora-and-4bit-quantization/) |

---

*Last updated: 2026-04-19*
