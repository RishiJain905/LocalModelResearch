# Multimodal-Tuning

## 📌 Overview

Techniques and architectures for fine-tuning multimodal large language models (MLLMs) that process both visual and textual inputs. Covers the three core components of any MLLM: the **vision encoder**, the **projector/connector**, and the **LLM backbone**.

---

## Subtopics

| File | Topic | Type |
|------|-------|------|
| [VLM.md](./VLM.md) | Vision-Language Models | Multimodal architecture taxonomy |
| [Projector-Alignment.md](./Projector-Alignment.md) | Visual Projectors / Modality Connectors | Alignment between vision and language |
| [Vision-Encoding.md](./Vision-Encoding.md) | Vision Encoding Strategies | CLIP, DINO, ViT, tokenization |

---

## Summary Matrix

| Component | Key Question | Answer |
|-----------|-------------|--------|
| **VLM** | What is a Vision-Language Model? | AI system that jointly interprets/generates from images + text |
| **Projector** | How do vision encoders connect to LLMs? | Via trainable connectors (MLP, Q-Former, C-Abstractor) |
| **Vision Encoding** | Which vision encoder is best? | CLIP for text-intensive tasks; DINO for fine-grained perception |

---

## Key Architecture

Every MLLM has three modules:

```
Image → [Vision Encoder] → [Projector/Connector] → [LLM] → Text
```

1. **Vision Encoder**: Converts image → visual tokens (CLIP, DINO, SigLIP, ViT)
2. **Projector**: Maps visual tokens → LLM embedding space (MLP, Q-Former, C-Abstractor)
3. **LLM**: Generates text from multimodal input

---

## Related Topics

- [Reasoning-r1](./Reasoning-r1/) — Reasoning models; orthogonal to multimodal but often combined (e.g., DeepSeek-VL)
- [Acceleration-Frameworks](./Acceleration-Frameworks/) — Fine-tuning speed and memory optimization
- [CLIP](../weight-quantization/CLIP.md) — Foundational vision-language contrastive model
