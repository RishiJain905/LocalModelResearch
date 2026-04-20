# Context Objects & Managed Agents

> **Key Papers:** [MemGPT (arXiv:2310.08560)](https://arxiv.org/abs/2310.08560) · [A-MEM (arXiv:2502.12110)](https://arxiv.org/abs/2502.12110) · [Memoria (arXiv:2512.12686)](https://arxiv.org/abs/2512.12686v1) · [Memory in the Age of AI Agents Survey](https://github.com/Shichun-Liu/Agent-Memory-Paper-List)
> **GitHub:** ⭐102,932 [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent) · ⭐53,553 [mem0ai/mem0](https://github.com/mem0ai/mem0) · ⭐22,167 [letta-ai/letta](https://github.com/letta-ai/letta)
> **Docs:** [Anthropic Memory Tool](https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool) · [Letta Memory Blocks](https://www.letta.com/blog/memory-blocks) · [OpenAI Agents SDK Context](https://openai.github.io/openai-agents-python/context/)

---

## Overview

Context Objects treat memory as a **first-class architectural primitive** — structured, persistent, queryable units that agents can read, write, edit, and evolve across sessions. Unlike KV cache (ephemeral attention scratchpad) or flat vector storage (passive retrieval index), context objects are **active, editable objects** that agents manage directly.

> *"We articulate a forward-looking perspective on emerging research frontiers, including memory automation, reinforcement learning integration, multimodal memory, multi-agent memory, and trustworthiness issues."* — Context Engineering, arXiv:2603.09619

---

## Key Papers

### 1. MemGPT: Towards LLMs as Operating Systems (2023) ★

> **Paper:** [arXiv:2310.08560](https://arxiv.org/abs/2310.08560) · **Authors:** Packer, Wooders, Lin, Vivian Fang, Patil, Stoica, Gonzalez (UC Berkeley)
> **GitHub:** ⭐22,167 [letta-ai/letta](https://github.com/letta-ai/letta) (evolved into Letta platform)

> *"We treat context windows as a constrained memory resource, and design a memory hierarchy for LLMs analogous to memory tiers used in traditional OSes."*

**Core innovation:** Virtual context management — pagination between main context (RAM analogy) and external context (disk analogy). Self-directed memory management via function calls. Evolved into the **Letta** platform.

### 2. A-MEM: Agentic Memory for LLM Agents (2025)

> **Paper:** [arXiv:2502.12110](https://arxiv.org/abs/2502.12110) · **GitHub:** ⭐976 [agiresearch/A-mem](https://github.com/agiresearch/A-mem)

> *"While LLM agents can effectively use external tools for complex real-world tasks, they require memory systems to leverage historical experiences. Current memory systems enable basic storage and retrieval but lack sophisticated memory organization."*

**Core innovation:** Dynamic memory organization based on **Zettelkasten method** — generates comprehensive notes with structured attributes (context, keywords, tags), dynamically links related memories, and evolves existing memories when new experiences are added.

### 3. Memoria: Scalable Agentic Memory Framework (2025)

> **Paper:** [arXiv:2512.12686](https://arxiv.org/abs/2512.12686v1)

> *"Memoria upgrades the LLM and user interaction paradigm from isolated exchanges to ongoing relationships and conversations that evolve, adapt, and remember."*

**Two-component memory:**
- **Short-term:** Dynamic session-level summarization
- **Long-term:** Weighted Knowledge Graph-based user modeling with recency-based exponential decay weighting

### 4. Memory in the Age of AI Agents: A Survey (2025)

> **GitHub:** ⭐1,840 [Shichun-Liu/Agent-Memory-Paper-List](https://github.com/Shichun-Liu/Agent-Memory-Paper-List)

> *"We hope this survey serves not only as a reference for existing work, but also as a conceptual foundation for rethinking memory as a first-class primitive in the design of future agentic intelligence."*

**Three dominant realizations of agent memory:**
1. **Token-level** — KV cache, MemGPT-style paging
2. **Parametric** — weights encode knowledge
3. **Latent** — learned internal representations

### 5. Context Engineering: From Prompts to Corporate Multi-Agent Architecture (2026)

> **Paper:** [arXiv:2603.09619](https://arxiv.org/abs/2603.09619)

Covers emerging frontiers: memory automation, RL integration, multimodal memory, multi-agent memory, and trustworthiness issues.

### 6. Codified Context: Infrastructure for AI Agents (2026)

> **Paper:** [arXiv:2602.20478](https://arxiv.org/abs/2602.20478v1)

> *"A hot-memory constitution encoding conventions, retrieval hooks, and orchestration protocols; 19 specialized domain-expert agents; and a cold-memory knowledge base of 34 on-demand specification documents."*

---

## Framework Implementations

### Hermes Agent (Nous Research) ⭐102,932

> *"The agent that grows with you"* — SQLite-based session storage with FTS5 full-text search

| Feature | Details |
|---------|---------|
| **Storage** | SQLite database (`~/.hermes/state.db`) |
| **Search** | FTS5 virtual table for full-text search across sessions |
| **Memory Tool** | Built-in session_search tool |
| **Persistence** | Sessions table, messages table |
| **Docs** | [mintlify.com/NousResearch/hermes-agent](https://www.mintlify.com/NousResearch/hermes-agent/user-guide/features/memory) |

### Mem0: Universal Memory Layer ⭐53,553

> *"Mem0 enhances AI assistants and agents with an intelligent memory layer, enabling personalized AI interactions."*

| Feature | Details |
|---------|---------|
| **Stars** | 53,553 |
| **Downloads** | 14M+ |
| **Benchmark** | 91.6% on LoCoMo (vs old algorithm 71.4%) |
| **Y Combinator** | S24 |
| **Use case** | Cross-session personalization, user preferences, agent memory |

### Letta (MemGPT Evolution) ⭐22,167

> *"Platform for building stateful agents with advanced memory that can learn and self-improve over time."*

**Memory Blocks:** Discrete, functional units for context window management. Agents can have multiple blocks (e.g., "human", "persona", "knowledge") that are editable and shareable across agents.

> *"Memory blocks provide the memory abstraction needed for LLM agents that are more capable, personalized, and controllable over time."*

### AgentMemory ⭐1,843

> *"4-tier memory consolidation, session replay, real-time viewer, MCP server with 44 tools."*

Integrations: OpenClaw, Hermes Agent, Claude Code, Cursor, any MCP-compatible agent.

---

## Anthropic Claude Memory Tool

> **Docs:** [platform.claude.com](https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool)

> *"This is the key primitive for just-in-time context retrieval: rather than loading all relevant information upfront, agents store what they learn in memory and pull it back on demand."*

| Command | Function |
|---------|----------|
| `view` | Read memory contents |
| `create` | Create new memory |
| `str_replace` | Edit existing memory |
| `insert` | Insert into memory |
| `delete` | Delete memory |
| `rename` | Rename memory |

**Storage Location:** `/memories` directory (client-controlled, file-based)

**Anthropic Internal Evals:**
- **39% improvement** combining memory with context editing on agentic search tasks
- **84% token reduction** in 100-turn web search evaluation
- **Beta released:** September 2025 as `context-management-2025-06-27`

---

## Major Cloud Provider Memory Frameworks

### Microsoft

| System | Description |
|--------|-------------|
| **Azure Cosmos DB AI Agents** | *"Managing agent memories requires weaving together in-memory, relational, and vector databases."* |
| **User-Scoped Persistent Memory** | Azure AI Foundry |
| **Mem0 Integration** | Azure AI Foundry + Mem0 + Azure AI Search |

### OpenAI

> **Docs:** [openai.github.io/openai-agents-python/context](https://openai.github.io/openai-agents-python/context/)

> *"Context available locally to your code: this is data and dependencies you might need when tool functions run."*

**State Object:** Structured state objects that persist across runs, enabling memory, notes, and preferences.

### Neo4j Agent Memory

> *"Graph-native memory system for AI agents — short-term, long-term, and reasoning memory types."*

Graph modeling, vector indexing, and query optimization for agent memory.

---

## KV Cache vs Context Objects

| Aspect | KV Cache | Context Objects |
|--------|----------|----------------|
| **Persistence** | Ephemeral (per-inference) | Persistent across sessions |
| **Agency** | None (inference optimization) | Agent can read/write/edit |
| **Structure** | Raw attention matrices | Structured, typed, queryable |
| **Lifecycle** | GPU memory | External storage (SQL, files) |
| **Purpose** | Speed up inference | Enable learning & continuity |
| **Examples** | PagedAttention, vLLM | MemGPT, Mem0, Claude Memory |

---

## Papers Table

| Paper | Year | Venue | URL | Key Contribution |
|-------|------|-------|-----|------------------|
| **MemGPT** | 2023 | arXiv | [arXiv:2310.08560](https://arxiv.org/abs/2310.08560) | OS-style memory hierarchy, virtual context paging |
| **A-MEM** | 2025 | arXiv | [arXiv:2502.12110](https://arxiv.org/abs/2502.12110) | Zettelkasten-based memory organization |
| **Memoria** | 2025 | arXiv | [arXiv:2512.12686](https://arxiv.org/abs/2512.12686v1) | Session summarization + KG-based personalization |
| **Memory Survey** | 2025 | arXiv | [GitHub ⭐1,840](https://github.com/Shichun-Liu/Agent-Memory-Paper-List) | Token/parametric/latent taxonomy |
| **Context Engineering** | 2026 | arXiv | [arXiv:2603.09619](https://arxiv.org/abs/2603.09619) | Multi-agent memory frontiers |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Core concept** | Memory as a first-class, structured, persistent, queryable object for agents |
| **Best OS analogy** | MemGPT — context as RAM/disk hierarchy, OS-style paging |
| **Best open-source** | Mem0 (⭐53,553), Letta (⭐22,167), Hermes Agent (⭐102,932) |
| **Best production memory** | Anthropic Claude Memory Tool (39% improvement, 84% token reduction) |
| **Key distinction** | Active (agent-editable) vs passive (vector store retrieval-only) |
| **Three memory types** | Token-level, parametric, latent |

---

*Last updated: 2026-04-20*