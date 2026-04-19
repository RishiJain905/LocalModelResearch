# Structured Pruning

> **Papers:** [LLM-Pruner](https://arxiv.org/abs/2305.11627) · [Sheared LLaMA](https://arxiv.org/abs/2310.06694) · [SliceGPT](https://arxiv.org/abs/2401.15024) · [SlimGPT](https://arxiv.org/abs/2412.18110) · [Bonsai](https://arxiv.org/abs/2402.05406) · [2SSP](https://arxiv.org/abs/2501.17771)  
> **GitHub:** [horseee/LLM-Pruner](https://github.com/horseee/LLM-Pruner) · [princeton-nlp/LLM-Shearing](https://github.com/princeton-nlp/LLM-Shearing) · [microsoft/TransformerCompression](https://github.com/microsoft/TransformerCompression) · [VainF/Torch-Pruning](https://github.com/VainF/Torch-Pruning)  
> **Blogs:** [Rasa — Pruning BERT](https://rasa.com/blog/pruning-bert-to-accelerate-inference) · [Hugging Face — GLU-Aware Pruning](https://huggingface.co/blog/oopere/making-llms-smaller-without-breaking-them)

---

## Overview

> *"LLM-Pruner adopts structural pruning that selectively removes non-critical coupled structures based on gradient information."* — Ma et al., [NeurIPS 2023](https://arxiv.org/abs/2305.11627)

Structured pruning removes entire architectural components (neurons, attention heads, layers, channels) from LLMs. Unlike unstructured pruning which creates sparse weight matrices requiring specialized hardware, structured pruning produces dense, compact models that run efficiently on standard hardware.

| Aspect | Details |
|--------|---------|
| **Method** | Remove entire neurons, heads, layers, or channels |
| **Resulting Format** | Dense, compact model |
| **Hardware Compatibility** | ✅ Works on standard GPU/CPU matrix operations |
| **Sparsity Level** | Typically 20–50% parameter reduction |
| **Accuracy** | May be lower than unstructured at equivalent sparsity |
| **Speedup** | 2–3× practical speedup |

---

## Types of Structured Pruning

| Type | What Gets Removed | Hardware Benefit |
|------|-------------------|------------------|
| **Channel/Neuron Pruning** | Individual neurons in MLP layers | Dense speedup |
| **Head Pruning** | Entire attention heads | Reduced sequence length |
| **Layer Pruning** | Entire transformer layers | Fewer computation passes |
| **GLU-Aware Pruning** | Paired neurons in GLU structures | Preserves model function |
| **Width Pruning** | MLP width dimension | Smaller hidden states |
| **Depth Pruning** | Number of layers | Shorter computation chain |

---

## Key Methods

### 1. LLM-Pruner (NeurIPS 2023)

> **Paper:** [arXiv:2305.11627](https://arxiv.org/abs/2305.11627) · **GitHub:** [horseee/LLM-Pruner](https://github.com/horseee/LLM-Pruner)  
> **Authors:** Xinyin Ma, Gongfan Fang, Xinchao Wang — National University of Singapore

**TL;DR:** First structural pruning framework specifically designed for LLMs. Uses DepGraph for automatic dependency grouping and Taylor expansion for importance estimation.

> *"LLM-Pruner adopts structural pruning that selectively removes non-critical coupled structures based on gradient information."*

**Supported Models:** Llama-3/3.1, Llama-2, LLaMA, BLOOM, Vicuna, Baichuan, TinyLlama, ChatGLM

**Workflow:** Pruning → Post-Training (LoRA) → Generation

**Key Features:**
- Uses Taylor or Hessian-based importance for structured removal
- Block-wise pruning for MLP and attention layers
- Post-training with LoRA for accuracy recovery

**Example Command:**
```bash
python hf_prune.py --pruning_ratio 0.25 --block_wise \
--block_mlp_layer_start 4 --block_mlp_layer_end 30 \
--block_attention_layer_start 4 --block_attention_layer_end 30 \
--pruner_type taylor
```

### 2. Sheared LLaMA (ICLR 2024)

> **Paper:** [arXiv:2310.06694](https://arxiv.org/abs/2310.06694) · **GitHub:** [princeton-nlp/LLM-Shearing](https://github.com/princeton-nlp/LLM-Shearing)  
> **Blog:** [xiamengzhou.github.io/sheared-llama](https://xiamengzhou.github.io/sheared-llama/)  
> **Authors:** Mengzhou Xia, Tianyu Gao, Zhiyuan Zeng, Danqi Chen — Princeton NLP

**TL;DR:** Dynamically prunes transformer layers, attention heads, and FFN neurons using L0 regularization. Requires continued pre-training.

> *"Pruning strong base models is an extremely cost-effective way to get strong small-scale language models compared to pre-training them from scratch. Pruning Llama-2-7B produces a model as strong as OpenLLaMA with **only 3% of its pre-training cost**."*

**Available Models:**

| Model | Size |
|-------|------|
| Sheared-LLaMA-1.3B | 1.3B params |
| Sheared-LLaMA-2.7B | 2.7B params |
| Sheared-LLaMA-1.3B-ShareGPT | Instruction-tuned |

### 3. SliceGPT (ICLR 2024)

> **Paper:** [arXiv:2401.15024](https://arxiv.org/abs/2401.15024) · **GitHub:** [microsoft/TransformerCompression](https://github.com/microsoft/TransformerCompression)  
> **Blog:** [MarkTechPost](https://www.marktechpost.com/2024/02/02/researchers-from-eth-zurich-and-microsoft-introduce-slicegpt-for-efficient-compression-of-large-language-models-through-sparsification/)

**TL;DR:** Computational-level pruning that replaces weight matrices with smaller dense matrices via PCA. Maintains dense format for efficient inference.

> *"SliceGPT is a new post-training sparsification scheme which replaces each weight matrix with a smaller (dense) matrix."*

**Method:**
1. Applies orthogonal transformations (RMSNorm compatibility) to each transformer layer
2. Uses Principal Component Analysis (PCA) to identify principal components
3. Slices off minor rows/columns of weight matrices, reducing embedding dimension

**Results:**
- Up to **25% of model parameters cut** (including embeddings)
- Inference compute reduced to **64%** (consumer GPUs) and **66%** (high-end GPUs)
- Outperforms SparseGPT across OPT, LLAMA-2, and Phi-2 models

**Usage:**
```bash
python run_slicegpt.py --model microsoft/phi-2 --sparsity 0.25
```

### 4. SlimGPT (NeurIPS 2024)

> **Paper:** [arXiv:2412.18110](https://arxiv.org/abs/2412.18110)

> *"SlimGPT presents a low-cost and fast structured pruning method for LLMs named based on the Optimal Brain Surgeon framework."*

**Key Innovation:**
- Uses **Optimal Brain Surgeon (OBS)** framework for importance estimation
- Proposes **Batched Greedy Pruning** for rapid near-optimal pruning
- Introduces **Incremental Pruning Ratio** (non-uniform strategy) to combat error accumulation
- Layer-wise pruning with parameter compensation

**Advancement over LLM-Pruner:** LLM-Pruner lacks parameter compensation — layers pruned at the output end have larger impact. SlimGPT reduces this through parameter compensation.

### 5. Bonsai — Gradient-Free Structured Pruning (2024)

> **Paper:** [arXiv:2402.05406](https://arxiv.org/abs/2402.05406)

> *"Bonsai, a gradient-free structured pruning method that eliminates the need for backpropagation, significantly reducing memory."*

**Key Innovation:** Only requires forward passes, no gradients needed:
- Enables pruning on memory-constrained devices
- Eliminates gradient storage overhead
- Uses magnitude-based importance with structured groups

### 6. 2SSP — Two-Stage Structured Pruning (TMLR 2025)

> **Paper:** [arXiv:2501.17771](https://arxiv.org/abs/2501.17771) · **GitHub:** [FabrizioSandri/2SSP](https://github.com/FabrizioSandri/2SSP)

**Two Stages:**
1. **Width Pruning:** Removes entire neurons and corresponding rows/columns
2. **Depth Pruning:** Removes entire transformer layers

> *"TRSP outperforms strong layer-wise structured pruning methods without requiring retraining."*

### 7. GLU-Aware Pruning (Hugging Face, 2025)

> **Blog:** [Hugging Face — Making LLMs Smaller Without Breaking Them](https://huggingface.co/blog/oopere/making-llms-smaller-without-breaking-them)

> *"The MLP layers represent more than 50% of the model's size, making them clear candidates for pruning."*

**Critical Discovery:** GLU-based models (LLaMA 3.2, Gemma, Mistral, QWen) require **paired pruning** of `gate_proj`, `up_proj`, and `down_proj` layers. Non-GLU-aware pruning causes catastrophic output degradation even at 20% pruning.

**Example of failure:**
- Non-GLU-aware pruning: *"Paris is the capital of of France. This is the the the the main the area of..."*
- GLU-aware pruning: *"Paris is the capital of France. It is also one of the most beautiful cities in the world..."*

---

## DepGraph: Foundation for Structural Pruning

> **Paper:** [arXiv:2301.12900](https://arxiv.org/abs/2301.12900) · **GitHub:** [VainF/Torch-Pruning](https://github.com/VainF/Torch-Pruning)

> *"DepGraph automatically identifies dependencies and collects groups for pruning."*

**Core Contribution:** Solves the coupling problem — when pruning one layer, automatically identifies and prunes all connected layers that depend on it. Powers LLM-Pruner's structural pruning capabilities.

---

## Structured vs. Unstructured Tradeoffs

| Aspect | Structured Pruning | Unstructured Pruning |
|--------|-------------------|---------------------|
| **What is removed** | Entire neurons, heads, layers | Individual weights |
| **Resulting format** | Dense, compact model | Sparse matrix |
| **Hardware compatibility** | ✅ Standard matrix ops | ❌ Requires sparse kernels |
| **Speedup potential** | Moderate (~2–3×) | High (theoretical 10×+) |
| **Accuracy retention** | May be lower (constrained) | Higher (fine-grained) |
| **Implementation complexity** | Lower | Higher |
| **Example methods** | LLM-Pruner, Sheared LLaMA, SliceGPT | Magnitude, SparseGPT, Wanda |

---

## Papers Table

| Paper | Year | Venue | URL | Key Contribution |
|-------|------|-------|-----|-----------------|
| **LLM-Pruner** | 2023 | NeurIPS 2023 | [arXiv](https://arxiv.org/abs/2305.11627) | First structural pruning framework for LLMs with DepGraph |
| **Sheared LLaMA** | 2023 | ICLR 2024 | [arXiv](https://arxiv.org/abs/2310.06694) | Dynamic L0 regularization pruning, 3% of pre-training cost |
| **SliceGPT** | 2024 | ICLR 2024 | [arXiv](https://arxiv.org/abs/2401.15024) | PCA-based matrix size reduction, 25% params cut |
| **SlimGPT** | 2024 | NeurIPS 2024 | [arXiv](https://arxiv.org/abs/2412.18110) | OBS-based batched greedy pruning with parameter compensation |
| **Bonsai** | 2024 | — | [arXiv](https://arxiv.org/abs/2402.05406) | Gradient-free structured pruning |
| **2SSP** | 2025 | TMLR 2025 | [arXiv](https://arxiv.org/abs/2501.17771) | Two-stage width + depth pruning |
| **DepGraph** | 2023 | CVPR 2023 | [arXiv](https://arxiv.org/abs/2301.12900) | Automatic dependency grouping for arbitrary architecture pruning |

---

## GitHub Repos

| Repo | Description |
|------|-------------|
| [horseee/LLM-Pruner](https://github.com/horseee/LLM-Pruner) | Structural pruning for LLMs (NeurIPS 2023) |
| [princeton-nlp/LLM-Shearing](https://github.com/princeton-nlp/LLM-Shearing) | Sheared LLaMA (ICLR 2024) |
| [microsoft/TransformerCompression](https://github.com/microsoft/TransformerCompression) | SliceGPT implementation |
| [VainF/Torch-Pruning](https://github.com/VainF/Torch-Pruning) | DepGraph: general structural pruning toolkit |
| [FabrizioSandri/2SSP](https://github.com/FabrizioSandri/2SSP) | Two-stage structured pruning |
| [horseee/Awesome-Efficient-LLM](https://github.com/horseee/Awesome-Efficient-LLM/blob/main/pruning.md) | Comprehensive pruning resources list |

---

## Blog Posts & Technical Reports

| Title | Source | URL |
|-------|--------|-----|
| **Pruning BERT to Accelerate Inference** | Rasa | [Link](https://rasa.com/blog/pruning-bert-to-accelerate-inference) |
| **Making LLMs Smaller Without Breaking Them: A GLU-Aware Approach** | Hugging Face Blog | [Link](https://huggingface.co/blog/oopere/making-llms-smaller-without-breaking-them) |
| **SliceGPT: Efficient Compression of LLMs** | MarkTechPost | [Link](https://www.marktechpost.com/2024/02/02/researchers-from-eth-zurich-and-microsoft-introduce-slicegpt-for-efficient-compression-of-large-language-models-through-sparsification/) |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Problem solved** | Reduces model size while maintaining hardware efficiency — produces dense compact models |
| **Key innovation** | Automatic dependency grouping (DepGraph), GLU-aware paired pruning, L0 regularization for learned sparsity |
| **Best for** | Deployment scenarios requiring real hardware speedups on standard GPUs; structured compression |
| **Trade-off** | Less flexible than unstructured — constrained to removing whole structural units |
| **Cost efficiency** | Sheared LLaMA: 3% of pre-training cost to match OpenLLaMA performance |

---

*Last updated: 2026-04-19*
