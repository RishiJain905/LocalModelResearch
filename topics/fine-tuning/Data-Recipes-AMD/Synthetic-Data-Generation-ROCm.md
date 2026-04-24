# Synthetic Data Generation on ROCm

## 📌 Overview

Synthetic data generation on AMD GPUs (ROCm) enables creating high-quality training datasets for fine-tuning LLMs without relying on human-labeled data. AMD provides several production-grade pipelines, datasets, and tools for synthetic data generation on Instinct MI300X/MI250 GPUs.

> *"SAND democratizes this process by providing a complete ecosystem—pipeline, dataset, and training code—that achieves Qwen3-32B performance using purely synthetic data and standard Supervised Fine-Tuning (SFT). All experiments were conducted on the AMD Instinct™ ROCm™ stack (MI325X/MI300X GPUs)."*
> — [AMD-AGI/sand-pipeline](https://github.com/AMD-AGI/sand-pipeline)

---

## SAND Pipeline (AMD-AGI)

**URL:** [github.com/AMD-AGI/sand-pipeline](https://github.com/AMD-AGI/sand-pipeline)

A complete synthetic data generation ecosystem for building SOTA reasoning models on AMD GPUs.

### Pipeline Stages

| Stage | Description |
|-------|-------------|
| **1. QA Generation & Consistency** | Teacher model generates problems from scratch |
| **2. De-duplication & Decontamination** | Remove duplicate/contaminated samples |
| **3. Difficulty Hiking** | Deeper reasoning chains, added constraints |
| **4. Solution Generation** | Generate step-by-step solutions |

### Hardware

> *"8x AMD Instinct™ MI300X accelerators with LLaMA-Factory + DeepSpeed ZeRO-3."*
> — [SAND Pipeline](https://github.com/AMD-AGI/sand-pipeline)

### Released Datasets

| Dataset | Size | Link |
|---------|------|------|
| **SAND-Post-Training-Dataset** | 27k samples | [HuggingFace](https://huggingface.co/datasets/amd/SAND-Post-Training-Dataset) |
| **SAND-Math-Qwen2.5-32B** | 14k math | [HuggingFace](https://huggingface.co/amd/SAND-Math-Qwen2.5-32B) |
| **SAND-MathScience-DeepSeek-Qwen32B** | 27k combined | [HuggingFace](https://huggingface.co/amd/SAND-MathScience-DeepSeek-Qwen32B) |

---

## LuminaSFT (AMD Official)

**URL:** [rocm.blogs.amd.com — LuminaSFT](https://rocm.blogs.amd.com/artificial-intelligence/luminasft/README.html)

> *"We release LuminaSFT — a supervised fine-tuning dataset comprising both general-purpose and multiple task-specific splits designed to improve small language model (SLM) performance. Task-specific data generation can deliver substantial performance gains. In domains such as reading comprehension, targeted dataset construction boosts performance by as much as ~41%."*
> — [AMD LuminaSFT](https://rocm.blogs.amd.com/artificial-intelligence/luminasft/README.html)

### Dataset Components

| Dataset | Purpose | Teacher Model | Size |
|---------|---------|--------------|------|
| **UltraChat200K-DeepSeek** | Instruction-following | DeepSeek-V3 | 200K |
| **InstructGPT-NaturalQA** | Factual QA | DeepSeek-V3 | ~1M |
| **InstructGPT-TriviaQA** | Factual QA | DeepSeek-V3 | ~1M |
| **CoT-Drop** | Reading comprehension | Qwen3-30B-A3B | DROP-based |
| **InstructGPT-Educational** | Educational QA (fully synthetic) | Qwen3-30B-A3B | Multi-step pipeline |

### Hardware

> *"AMD Instinct™ MI300X and MI250 with ROCm."*
> — [AMD LuminaSFT](https://rocm.blogs.amd.com/artificial-intelligence/luminasft/README.html)

---

## Unsloth + Synthetic-Data-Kit (AMD Tutorial)

**URL:** [amd.com — 10x Model Fine-Tuning Using Synthetic Data with Unsloth](https://www.amd.com/en/developer/resources/technical-articles/2025/10x-model-fine-tuning-using-synthetic-data-with-unsloth.html)

> *"In this blog, we will teach you how to generate synthetic data and then fine-tune the same model on these outputs to help it answer and generate high-quality logical reasoning data."*
> — [AMD Technical Article](https://www.amd.com/en/developer/resources/technical-articles/2025/10x-model-fine-tuning-using-synthetic-data-with-unsloth.html)

### End-to-End Pipeline

```
Source PDFs → Parse to Text → Generate QA/CoT Pairs → Curate with LLM Judge → Convert to Training Format → Fine-tune with Unsloth
```

### Tools Used

| Tool | Purpose |
|------|---------|
| **Synthetic-Data-Kit** | Data generation (vLLM backend on port 8001) |
| **Unsloth** | Memory-efficient fine-tuning |

### Configuration Example

```yaml
# config.yaml for Synthetic-Data-Kit
llm:
  provider: "vllm"
vllm:
  api_base: "http://localhost:8001/v1"
  model: "unsloth/Llama-3.3-70B-Instruct"
```

### Results

> *"Llama-3.3-70B fine-tuned on single MI300X with **41GB VRAM**, batch size 128."*
> — [AMD Technical Article](https://www.amd.com/en/developer/resources/technical-articles/2025/10x-model-fine-tuning-using-synthetic-data-with-unsloth.html)

---

## ROCm Fine-Tuning Documentation

**URL:** [rocm.docs.amd.com — Fine-tuning](https://rocm.docs.amd.com/en/latest/how-to/rocm-for-ai/fine-tuning/index.html)

> *"The ROCm™ software platform helps you optimize this fine-tuning process by supporting various optimization techniques tailored for AMD GPUs. It empowers the fine-tuning of large language models, making them accessible and efficient for specialized tasks."*
> — [ROCm Docs](https://rocm.docs.amd.com/en/latest/how-to/rocm-for-ai/fine-tuning/index.html)

Supports single-GPU and multi-GPU fine-tuning configurations on all AMD GPUs.

---

## Summary Table

| Project/Resource | Type | Hardware | Focus |
|-----------------|------|----------|-------|
| [AMD-AGI/sand-pipeline](https://github.com/AMD-AGI/sand-pipeline) | GitHub Repo | MI325X/MI300X | Math/science reasoning |
| [LuminaSFT](https://rocm.blogs.amd.com/artificial-intelligence/luminasft/README.html) | Blog + Dataset | MI300X/MI250 | Small LM SFT data |
| [Unsloth + Synthetic-Data-Kit](https://www.amd.com/en/developer/resources/technical-articles/2025/10x-model-fine-tuning-using-synthetic-data-with-unsloth.html) | Tutorial | MI300X | General logical reasoning |
| [ROCm Fine-tuning Docs](https://rocm.docs.amd.com/en/latest/how-to/rocm-for-ai/fine-tuning/index.html) | Documentation | All AMD GPUs | LLMs |

---

## Sources

| Type | Citation |
|------|----------|
| **GitHub** | [AMD-AGI/sand-pipeline](https://github.com/AMD-AGI/sand-pipeline) |
| **Blog** | [AMD LuminaSFT](https://rocm.blogs.amd.com/artificial-intelligence/luminasft/README.html) |
| **Article** | [AMD — 10x Model Fine-Tuning with Unsloth](https://www.amd.com/en/developer/resources/technical-articles/2025/10x-model-fine-tuning-using-synthetic-data-with-unsloth.html) |
| **Docs** | [ROCm Fine-tuning Guide](https://rocm.docs.amd.com/en/latest/how-to/rocm-for-ai/fine-tuning/index.html) |
| **Dataset** | [HuggingFace — SAND-Post-Training-Dataset](https://huggingface.co/datasets/amd/SAND-Post-Training-Dataset) |
| **Dataset** | [HuggingFace — SAND-Math-Qwen2.5-32B](https://huggingface.co/amd/SAND-Math-Qwen2.5-32B) |
| **Dataset** | [HuggingFace — SAND-MathScience-DeepSeek-Qwen32B](https://huggingface.co/amd/SAND-MathScience-DeepSeek-Qwen32B) |

---

## Related Topics

- [Unsloth-AMD-Stack](../Unsloth-AMD-Stack/) — ROCm/HIP/bitsandbytes for AMD
- [Synthetic-Data](../Synthetic-Data/) — General synthetic data generation methods
