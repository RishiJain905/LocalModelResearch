# VLM (Vision-Language Models)

## 📌 Overview

| Field | Value |
|-------|-------|
| **Full Name** | Vision-Language Model |
| **Type** | Multimodal AI system |
| **Input** | Images + Text |
| **Output** | Text (typically) |
| **Key Task** | Joint image-text understanding and generation |

> "A vision-language model (VLM) is a type of artificial intelligence system that can jointly interpret and generate information from both images and text."

— [Wikipedia](https://en.wikipedia.org/wiki/Vision-language_model)

> "Vision language models are broadly defined as multimodal models that can learn from images and text. They are a type of generative models that take image and text inputs, and generate text outputs."

— [Hugging Face Blog](https://huggingface.co/blog/vlms)

---

## Definitions & Standards

### Formal Definition

> "Following the recent popularity of Large Language Models (LLMs), several attempts have been made to extend them to the visual domain."

— [Bordes et al., arXiv:2405.17247](https://arxiv.org/html/2405.17247v1) (Meta AI)

### Architecture Categories

| Type | Examples | Description |
|------|----------|-------------|
| **Dual-encoder** | CLIP, SigLIP | Contrastive learning with InfoNCE loss |
| **Fusion transformer** | Flamingo | Cross-attention for fine-grained grounding |
| **Unified single-stream** | GPT-4V, Qwen-VL | Visual tokens within language model |
| **Modular bridge** | BLIP-2, Q-Former | Query-based adapters connect frozen encoders to LLM |

---

## Key Sources

### Foundational Papers

| Paper | Year | URL | Key Quote |
|-------|------|-----|-----------|
| BLIP-2: Bootstrapping Language-Image Pre-Training | 2023 | [arXiv:2301.12597](https://arxiv.org/html/2301.12597) | "188M trainable params vs 10.2B Flamingo; 54x fewer parameters" |
| LLaVA: Visual Instruction Tuning (NeurIPS 2023 Oral) | 2023 | [arXiv](https://proceedings.neurips.cc/paper_files/paper/2023/file/6dcf277ea32ce3288914faf369fe6de0-Paper-Conference.pdf) | "First attempt to use language-only GPT-4 to generate multimodal language-image instruction-following data" |
| InstructBLIP (NeurIPS 2023) | 2023 | [arXiv:2305.06500](https://arxiv.org/abs/2305.06500) | "Systematic and comprehensive study on vision-language instruction tuning" |

### Efficient VLMs Survey (2025)

> "Vision-language models (VLMs) integrate visual and textual information for tasks such as image captioning and visual question answering (VQA). However, their high computational demands hinder real-time and edge deployment."

— [arXiv:2504.09724](https://arxiv.org/html/2504.09724v3)

### MLLM Survey (CVPR 2025)

> "A typical vision-language MLLM comprises three modules: (1) Modality encoder (e.g., vision encoder); (2) Connector (aligns vision features to text embedding space); (3) LLM (generates responses from concatenated multimodal inputs)."

— [MME-Survey arXiv:2411.15296](https://arxiv.org/html/2411.15296v1)

---

## Repositories & Tools

### Fine-tuning Frameworks

| Repository | URL | Description |
|------------|-----|-------------|
| LLaVA | [GitHub](https://github.com/haotian-liu/LLaVA) | Most popular open VLM; visual instruction tuning |
| MiniGPT-4 | [GitHub](https://github.com/catid/minigpt4) | Lightweight GPT-4 equivalent |
| BLIP-2/LAVIS | [GitHub](https://github.com/salesforce/LAVIS) | Salesforce VLM framework |
| Qwen-VL-finetune | [GitHub](https://github.com/cognitedata/Qwen-VL-finetune) | Qwen VL fine-tuning |
| Qwen3-VL | [GitHub](https://github.com/QwenLM/Qwen3-VL) | Next-gen Qwen VL |
| MLX-VLM | [GitHub](https://github.com/Blaizzy/mlx-vlm) | Apple Silicon (MLX) optimized |
| LLaMA-Factory | [GitHub](https://github.com/hiyouga/LLaMA-Factory) | Unified fine-tuning framework |

### Benchmark & Evaluation

| Tool | URL |
|------|-----|
| MMBench | [GitHub](https://github.com/open-compass/MMBench) |
| MMMU Benchmark | [GitHub](https://github.com/MMMU-Benchmark/MMMU) |
| Open VLM Leaderboard | [HuggingFace](https://huggingface.co/spaces/opencompass/open_vlm_leaderboard) |
| VLMEvalKit | [GitHub](https://github.com/open-compass/VLMEvalKit) |

---

## Key Performance Data

### Benchmark Results

| Benchmark | Model | Score |
|-----------|-------|-------|
| VQAv2 (zero-shot) | BLIP-2 ViT-g FlanT5-XXL | 65.0% |
| Science QA | LLaVA + GPT-4 | 92.53% |
| MMMU | GPT-5.1 | 0.854 |

### Model Specifications

| Model | Size | Image Resolution |
|-------|------|-----------------|
| LLaVA 1.6 (Hermes 34B) | 34B | 672×672 |
| DeepSeek-VL-7B | 7B | 384×384 |
| CogVLM-Chat | 17B | 490×490 |
| MiniGPT-4 | 7B | 224×224 |

---

## Debates & Controversies

### Visual Hallucination Problem (CVPR 2024)

> "LVLMs suffer from more hallucinations when tasked to focus on multiple objects, compared to focusing on a single object."

— [DAMO Academy/Alibaba, CVPR 2024](https://openaccess.thecvf.com/content/CVPR2024/papers/Leng_Mitigating_Object_Hallucinations_in_Large_Vision-Language_Models_through_Visual_Contrastive_CVPR_2024_paper.pdf)

### Fine-Tuning Encourages Hallucinations (EMNLP 2024)

> "When large language models are aligned via supervised fine-tuning, they may encounter new factual information that was not acquired during pre-training."

— [Gekhman et al., EMNLP 2024](https://aclanthology.org/2024.emnlp-main.444/)

### Multi-Object Confusion (NeurIPS 2024)

> "LVLMs may be following shortcuts and spurious correlations."

— [NeurIPS 2024](https://proceedings.neurips.cc/paper_files/paper/2024/file/4ea4a1ea4d9ff273688c8e92bd087112-Paper-Conference.pdf)

---

## Key Challenges

| Challenge | Description |
|-----------|-------------|
| **Visual hallucination** | Generating objects not present in images |
| **Multi-object confusion** | More hallucinations when focusing on multiple objects |
| **Attribute/ordering ignorance** | Struggles with spatial relationships and counting |
| **Cross-modal alignment** | Image and text feature mismatch |
| **Edge deployment** | High computational demands for real-time use |
| **New knowledge overfitting** | Fine-tuning on new facts encourages hallucinations |

---

## Related Topics

- [Projector-Alignment](./Projector-Alignment.md) — Bridge between vision encoder and LLM
- [Vision-Encoding](./Vision-Encoding.md) — Vision encoding strategies (CLIP, DINO, ViT)
- [CLIP](../weight-quantization/CLIP.md) — Contrastive Language-Image Pretraining
- [LLaVA](../model-cards/LLaVA.md) — Visual instruction tuning approach

---

## References

- [Wikipedia — Vision-language model](https://en.wikipedia.org/wiki/Vision-language_model)
- [Hugging Face — Vision Language Models Explained](https://huggingface.co/blog/vlms)
- [BLIP-2 (arXiv:2301.12597)](https://arxiv.org/html/2301.12597)
- [LLaVA (NeurIPS 2023)](https://proceedings.neurips.cc/paper_files/paper/2023/file/6dcf277ea32ce3288914faf369fe6de0-Paper-Conference.pdf)
- [Efficient VLMs Survey (arXiv:2504.09724)](https://arxiv.org/html/2504.09724v3)
