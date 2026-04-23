# Fine-tuning

Techniques, frameworks, and methodologies for adapting large language models to specific tasks or domains.

---

## Topics

| Topic | Description |
|-------|-------------|
| [Acceleration-Frameworks](./Acceleration-Frameworks/) | Fine-tuning acceleration frameworks: Unsloth, Axolotl, QAT, FSDP2 |
| [Modular-MOE-Merging](./Modular-MOE-Merging/) | Modular MoE merging: BAR, Expert Adaption, Router Tuning |
| [SSM-Hybrid-Tuning](./SSM-Hybrid-Tuning/) | SSM hybrid tuning: Mamba-3, Jamba, State Optimization |
| [PEFT-Consumer-VRAM](./PEFT-Consumer-VRAM/) | PEFT on consumer VRAM: LoRA, QLoRA, DoRA, rsLoRA |
| [Reasoning-r1](./Reasoning-r1/) | Reasoning-r1 post-training techniques: GRPO, RLVR, GIFT, Thinking-CoT |
| [Multimodal-Tuning](./Multimodal-Tuning/) | VLM architectures, projector alignment, vision encoding strategies |

---

## Acceleration-Frameworks

> Fine-tuning acceleration frameworks reduce VRAM usage, training time, or both — via custom kernels (Triton/CUDA), quantization-aware training, distributed sharding, or efficient adapter methods.

| Framework | Type | Key Benefit | GitHub |
|-----------|------|-------------|--------|
| [Unsloth](./Acceleration-Frameworks/Unsloth.md) | Single-GPU optimized | ~2× speed, ~70% less VRAM via Triton kernels + smart packing | [unslothai/unsloth](https://github.com/unslothai/unsloth) ⭐ 56.5k |
| [Axolotl](./Acceleration-Frameworks/Axolotl.md) | Multi-GPU / YAML-driven | All major training methods, FSDP2/DDP/DeepSpeed backends | [axolotl-ai-cloud/axolotl](https://github.com/axolotl-ai-cloud/axolotl) |
| [QAT](./Acceleration-Frameworks/QAT.md) | Quantization technique | Recovers up to 96% of accuracy lost to PTQ at INT4 | [facebookresearch/LLM-QAT](https://github.com/facebookresearch/LLM-QAT) |
| [FSDP2](./Acceleration-Frameworks/FSDP2.md) | Distributed sharding | Per-parameter DTensor sharding, 7% less memory vs FSDP1 | [pytorch/torchtitan](https://github.com/pytorch/torchtitan) |
