# LocalModelResearch

> Wiki-style UI research repository covering topics related to local AI models — setup, fine-tuning, inference, deployment, and real-world usage.

---

## 📚 Topics

### Models & Architecture
- [Model Cards](./topics/models/) — Architecture comparisons, benchmarks, capability analysis
- [Quantization](./topics/quantization/) — INT4/INT8/GGUF methods, quality vs size tradeoffs
- [Context Windows](./topics/context-windows/) — Attention mechanisms, sliding windows, retrieval-augmented generation

### Inference & Serving
- [Inference Engines](./topics/inference/) — llama.cpp, vLLM, Ollama, text-generation-webui
- [API Servers](./topics/api-servers/) — FastAPI backends, OpenAI-compatible endpoints, WebSocket streaming
- [Hardware Acceleration](./topics/hardware/) — GPU/CPU optimization, Apple Silicon, local cluster setups

### Fine-Tuning & Training
- [Fine-Tuning Guides](./topics/fine-tuning/) — LoRA, QLoRA, Axolotl, DPO/GRPO
- [Datasets](./topics/datasets/) — Curation, filtering, synthetic data generation
- [Training Infrastructure](./topics/training/) — Distributed training, FSDP, cloud vs local

### UX & Interfaces
- [Web UIs](./topics/web-ui/) — Interface comparisons, custom frontends, real-time features
- [Chatbot Implementations](./topics/chatbots/) — Prompt engineering, persona systems, memory management
- [Mobile & Edge](./topics/mobile/) — On-device inference, iOS/Android, embedded systems

### Research & Evaluation
- [Benchmarks](./topics/benchmarks/) — MMLU, HellaSwag, HumanEval, custom evals
- [Safety & Alignment](./topics/safety/) — RLHF, constitutional AI, jailbreak resistance
- [Monitoring & Observability](./topics/monitoring/) — Latency tracking, token usage, cost analysis

### Community & Ecosystem
- [Hugging Face](./topics/huggingface/) — Hub navigation, model versioning, Spaces
- [Open Source Projects](./topics/projects/) — Notable repos, contributor guides, licensing
- [News & Papers](./topics/news/) — arxiv summaries, release notes, industry updates

---

## 🛠️ Contributing

1. **Pick a topic folder** (or propose a new one via Issue)
2. **Follow the wiki format** — structured headers, code examples, citations
3. **Keep it practical** — real benchmarks, working commands, actionable guides
4. **Cite sources** — link to papers, docs, and original implementations

> This repo is research-oriented. If you hit something outdated or wrong, open an Issue or PR.

---

## 📂 Repo Structure

```
topics/           # All research documents (one folder per topic)
├── models/
├── quantization/
├── inference/
├── web-ui/
... (more as topics grow)

scripts/          # Utility scripts for benchmarks, setup, etc.
references/       # Raw paper links, external docs, data exports
```

---

*Last updated: 2026-04-19*
