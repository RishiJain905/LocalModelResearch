# Managed Agents & "Decoupled" Memory

> *"We hope this survey serves not only as a reference for existing work, but also as a conceptual foundation for rethinking memory as a first-class primitive in the design of future agentic intelligence."* — Memory in the Age of AI Agents Survey, 2025

Research on agents with persistent, structured, queryable memory — covering Context Objects (MemGPT, Mem0, Claude Memory) and Memory-Augmented Transformers (Infini-attention, Titans, Memorizing Transformers).

---

## Topics

| Topic | Description | Key Papers |
|-------|-------------|------------|
| [**Context Objects**](./Context-Objects.md) | Structured, persistent, queryable memory units for agents — MemGPT, Mem0, Claude Memory, Hermes Agent | MemGPT (arXiv:2310.08560), A-MEM, Memoria |
| [**Memory-Augmented Transformers**](./Memory-Augmented-Transformers.md) | Transformers with external memory modules — Infini-attention, Titans, RMT, Memorizing Transformers | Infini-attention (Google), Titans, Memorizing Transformers (ICLR 2022) |

---

## Quick Reference

### KV Cache vs Context Objects vs Memory-Augmented Transformers

| Aspect | KV Cache | Context Objects | Memory-Augmented Transformers |
|--------|----------|----------------|------------------------------|
| **Persistence** | Ephemeral (per-inference) | Persistent across sessions | Persistent across inference |
| **Agency** | None | Agent can read/write/edit | Model reads/writes to external memory |
| **Architecture** | Attention mechanism | External storage (SQL, files, KG) | Dedicated memory modules in model |
| **Examples** | PagedAttention, vLLM | MemGPT, Mem0, Claude Memory | Infini-attention, Titans, RMT |
| **Purpose** | Speed up inference | Learning & continuity | Infinite context & continual learning |

### Key Repos Quick Reference

| Repo | ⭐ | Description |
|------|-----|-------------|
| [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent) | 102,932 | SQLite + FTS5 session memory |
| [mem0ai/mem0](https://github.com/mem0ai/mem0) | 53,553 | Universal memory layer for AI agents |
| [letta-ai/letta](https://github.com/letta-ai/letta) | 22,167 | MemGPT → stateful agents platform |
| [rohitg00/agentmemory](https://github.com/rohitg00/agentmemory) | 1,843 | 4-tier memory for coding agents |
| [google-deepmind/dnc](https://github.com/google-deepmind/dnc) | 2,538 | Differentiable Neural Computer |
| [kimiyoung/transformer-xl](https://github.com/kimiyoung/transformer-xl) | 3,699 | Segment-level recurrence (memory foundation) |
| [booydar/recurrent-memory-transformer](https://github.com/booydar/recurrent-memory-transformer) | 778 | RMT wrapper for HuggingFace |

---

*Last updated: 2026-04-20*