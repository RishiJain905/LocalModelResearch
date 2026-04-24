# Unsloth AI

## 📌 Overview

| Field | Value |
|-------|-------|
| **Organization Name** | Unsloth AI (commonly: "Unsloth") |
| **NOT** | Unsloth Studios |
| **Founded** | 2023 |
| **Founders** | Daniel Han (Software/Algorithms) & Michael Han (Design/Product) — Australian brothers |
| **Funding** | ~$500K-$540K seed; Y Combinator Summer 2024 batch |
| **GitHub** | [unslothai/unsloth](https://github.com/unslothai/unsloth) — 56.5k stars |
| **Downloads** | 10M+ monthly |
| **License** | Apache 2.0 (core) / AGPL-3.0 (Studio UI) |

> "Hey there! We started as a team of two brothers!"

— [Unsloth AI About Page](https://unsloth.ai/about)

---

## Identity & History

### Previous Work: HyperLearn

Before Unsloth, the founders created **HyperLearn** — a NumPy/SciPy replacement library that was trusted by Microsoft, NVIDIA, Meta, NASA, and others. This established their reputation for optimized algorithms before moving into LLM training.

### What They Build

> "Train 500+ models up to 2x faster with up to 70% less VRAM."

— [Unsloth GitHub README](https://github.com/unslothai/unsloth)

> "Our goal is first and foremost open-source."

— [Daniel & Michael Han, Unsloth Blog Feb 2025](https://unsloth.ai/blog/reintroducing)

### Funding & Investors

| Source | Details |
|--------|---------|
| Y Combinator | Summer 2024 batch |
| Investors | Pioneer Fund, Transpose Platform, Microsoft M12 |
| Total Raised | ~$500K-$540K seed |

— [Startup Intros](https://startupintros.com/orgs/unsloth-ai), [Crunchbase](https://www.crunchbase.com/organization/unsloth-ai)

---

## Products

### Unsloth Core

- **Install**: `pip install unsloth`
- **License**: Apache 2.0
- **What it does**: Fine-tune LLMs 2-5x faster with 30-90% less VRAM
- **Key features**:
  - Custom Triton kernels (RoPE, MLP, RMSNorm)
  - 4-bit / 16-bit LoRA / QLoRA fine-tuning
  - Smart padding-free packing
  - GRPO reinforcement learning support
  - 500+ supported models

### Unsloth Studio

> "Open source, no code web UI."

— [Junia AI Review](https://www.junia.ai/blog/unsloth-studio-local-ai-fine-tuning)

- **Repository**: [unslothai/unsloth-studio](https://github.com/unslothai/unsloth-studio)
- **License**: AGPL-3.0
- **Status**: Beta (web UI)
- Browser-based fine-tuning interface — no code required

### Docker

- **Image**: [unsloth/unsloth on Docker Hub](https://hub.docker.com/r/unsloth/unsloth)

### HuggingFace

- [huggingface.co/unsloth](https://huggingface.co/unsloth) — pre-trained adapters and model ports

---

## Performance Claims

| Model | Speedup | VRAM Reduction |
|-------|---------|----------------|
| Qwen3.5 (4B) | 1.5x | 60% |
| gpt-oss (20B) | 2x | 70% |
| gpt-oss (20B) + GRPO | 2x | 80% |
| Llama 3.1 (8B) | 2x | 70% |
| Gemma 3 (4B) Vision | 1.7x | 60% |

> "GRPO is an optimized PPO variant."

— [Unsloth Blog: RL Environments (Mar 2026)](https://unsloth.ai/blog/rl-environments)

---

## Community & Support

| Platform | Link |
|----------|------|
| GitHub Discussions | [unslothai/unsloth/discussions](https://github.com/unslothai/unsloth/discussions) |
| Discord | [discord.com/invite/unsloth](https://discord.com/invite/unsloth) |
| Reddit | [r/unsloth](https://www.reddit.com/r/unsloth/) |

---

## Controversies & Debates

### Quantization Quality Debate

> "Unsloth team we need to talk."

— [Reddit r/LocalLLaMA](https://www.reddit.com/r/LocalLLaMA/comments/1rfe1l6/unsloth_team_we_need_to_talk/)

Community noted that some Unsloth quantization variants have higher perplexity despite larger model size — raising questions about the quality of certain dynamic quantization approaches vs their file size.

### Windows BitDefender Blocking Install

Some Windows users reported BitDefender flagging the Unsloth installation package — likely due to the bundled Triton kernels triggering false positives.

### MoE Licensing Questions

Community discussions around licensing implications when applying Unsloth's optimized kernels to Mixture-of-Experts models under various open-source licenses.

---

## Related Topics

- [Custom-Kernels.md](./Custom-Kernels.md) — The Triton kernels powering Unsloth
- [Unsloth.md](./Unsloth-AI.md) — (this file)
- [Acceleration-Frameworks](./Acceleration-Frameworks/) — Parent folder; covers Axolotl, QAT, FSDP2 as alternatives

---

## References

- [Unsloth GitHub](https://github.com/unslothai/unsloth)
- [Unsloth About](https://unsloth.ai/about)
- [Unsloth Blog — Reintroducing](https://unsloth.ai/blog/reintroducing)
- [Unsloth Blog — RL Environments](https://unsloth.ai/blog/rl-environments)
- [Junia AI — Unsloth Studio Review](https://www.junia.ai/blog/unsloth-studio-local-ai-fine-tuning)
