# VLM (Vision-Language Models) Research Report

## 1. Definitions & Standards

**Wikipedia - Vision-language model:**
"A vision-language model (VLM) is a type of artificial intelligence system that can jointly interpret and generate information from both images and text."
URL: https://en.wikipedia.org/wiki/Vision-language_model

**Hugging Face Blog - Vision Language Models Explained (April 2024):**
"Vision language models are broadly defined as multimodal models that can learn from images and text. They are a type of generative models that take image and text inputs, and generate text outputs."
URL: https://huggingface.co/blog/vlms

**arXiv - An Introduction to Vision-Language Modeling (2024):**
"Following the recent popularity of Large Language Models (LLMs), several attempts have been made to extend them to the visual domain."
URL: https://arxiv.org/html/2405.17247v1
Authors: Florian Bordes et al. (Meta AI)

## 2. Key Sources - Academic Papers

### BLIP-2: Bootstrapping Language-Image Pre-Training
- URL: https://arxiv.org/html/2301.12597
- Authors: Junnan Li, Dongxu Li, Silvio Savarese, Steven Hoi (Salesforce Research)
- Date: January 2023
- Key Quote: "The cost of vision-and-language pre-training has become increasingly prohibitive due to end-to-end training of large-scale models."
- Key Achievement: Only 188M trainable parameters vs 10.2B in Flamingo; 54x fewer parameters

### InstructBLIP: Towards General-purpose Vision-Language Models (NeurIPS 2023)
- URL: https://arxiv.org/abs/2305.06500
- Authors: Salesforce Research
- Key Quote: "Systematic and comprehensive study on vision-language instruction tuning based on BLIP-2 models."

### LLaVA - Visual Instruction Tuning (NeurIPS 2023 Oral)
- URL: https://proceedings.neurips.cc/paper_files/paper/2023/file/6dcf277ea32ce3288914faf369fe6de0-Paper-Conference.pdf
- Authors: Haotian Liu (UW-Madison), Chunyuan Li (Microsoft Research), Qingyang Wu (Columbia), Yong Jae Lee (UW-Madison)
- Key Quote: "First attempt to use language-only GPT-4 to generate multimodal language-image instruction-following data."
- Achievement: Science QA accuracy 92.53% with LLaVA + GPT-4

### A Survey on Efficient Vision-Language Models (2025)
- URL: https://arxiv.org/html/2504.09724v3
- Authors: Gaurav Shinde (UMBC)
- Key Quote: "Vision-language models (VLMs) integrate visual and textual information for tasks such as image captioning and visual question answering (VQA). However, their high computational demands hinder real-time and edge deployment."

### Survey of State of the Art Large Vision Language Models (2025)
- URL: https://arxiv.org/abs/2501.02189
- Note: CVPR 2025 publication on alignment, benchmarks, evaluations and challenges

## 3. Repositories & Tools

### Fine-tuning Repositories
| Repository | URL |
|------------|-----|
| LLaVA | https://github.com/haotian-liu/LLaVA |
| MiniGPT-4 | https://github.com/catid/minigpt4 |
| BLIP-2/LAVIS | https://github.com/salesforce/LAVIS |
| Qwen-VL-finetune | https://github.com/cognitedata/Qwen-VL-finetune |
| Qwen3-VL | https://github.com/QwenLM/Qwen3-VL |
| MLX-VLM (Mac) | https://github.com/Blaizzy/mlx-vlm |
| VLM-Finetune | https://github.com/janhq/VLM-Finetune |
| LLaMA-Factory | https://github.com/hiyouga/LLaMA-Factory |

### Benchmark & Evaluation Tools
| Tool | URL |
|------|-----|
| MMBench | https://github.com/open-compass/MMBench |
| MMMU Benchmark | https://github.com/MMMU-Benchmark/MMMU |
| Open VLM Leaderboard | https://huggingface.co/spaces/opencompass/open_vlm_leaderboard |
| VLMEvalKit | https://github.com/open-compass/VLMEvalKit |

## 4. Debates & Controversies

### Visual Hallucination Problem (CVPR 2024)
- URL: https://openaccess.thecvf.com/content/CVPR2024/papers/Leng_Mitigating_Object_Hallucinations_in_Large_Vision-Language_Models_through_Visual_Contrastive_CVPR_2024_paper.pdf
- Authors: DAMO Academy, Alibaba Group; Nanyang Technological University
- Finding: "LVLMs suffer from more hallucinations when tasked to focus on multiple objects, compared to focusing on a single object."

### Fine-Tuning on New Knowledge Encourages Hallucinations (EMNLP 2024)
- URL: https://aclanthology.org/2024.emnlp-main.444/
- Authors: Gekhman et al. (Ben-Gurion University)
- Key Quote: "When large language models are aligned via supervised fine-tuning, they may encounter new factual information that was not acquired during pre-training."

### Multi-Object Hallucination in Vision Language Models (NeurIPS 2024)
- URL: https://proceedings.neurips.cc/paper_files/paper/2024/file/4ea4a1ea4d9ff273688c8e92bd087112-Paper-Conference.pdf
- Finding: "LVLMs may be following shortcuts and spurious correlations."

## 5. Summary Table / Data Points

### VLM Architecture Categories
| Type | Examples | Description |
|------|----------|-------------|
| Dual-encoder | CLIP, SigLIP | Contrastive learning with InfoNCE loss |
| Fusion transformer | Flamingo | Cross-attention for fine-grained grounding |
| Unified single-stream | GPT-4V, Qwen-VL | Visual tokens within language model |
| Modular bridge | BLIP-2, Q-Former | Query-based adapters connect frozen encoders to LLM |

### Key Performance Numbers
| Benchmark | Model | Score |
|-----------|-------|-------|
| VQAv2 (zero-shot) | BLIP-2 ViT-g FlanT5-XXL | 65.0% |
| Science QA | LLaVA + GPT-4 | 92.53% |
| MMMU | GPT-5.1 | 0.854 |

### Key VLM Model Specifications
| Model | Size | Image Resolution |
|-------|------|------------------|
| LLaVA 1.6 (Hermes 34B) | 34B | 672x672 |
| DeepSeek-VL-7B | 7B | 384x384 |
| CogVLM-Chat | 17B | 490x490 |
| MiniGPT-4 | 7B | 224x224 |

### Key Challenges in VLM Research
1. Visual hallucination - generating objects not present in images
2. Multi-object confusion - more hallucinations when focusing on multiple objects
3. Attribute/ordering ignorance - struggles with spatial relationships and counting
4. Cross-modal alignment - image and text feature mismatch
5. Edge deployment - high computational demands for real-time use
6. New knowledge overfitting - fine-tuning on new facts encourages hallucinations (EMNLP 2024)
