# RAG-to-Runtime Evolution

> *"Indiscriminately retrieving and incorporating a fixed number of retrieved passages, regardless of whether retrieval is necessary, or passages are relevant, diminishes LM versatility or can lead to unhelpful response generation."* — Self-RAG, ICLR 2024

Research on the evolution from static RAG pipelines to adaptive, graph-augmented, and context-engineered retrieval systems. Covers GraphRAG (knowledge graph–augmented generation) and Adaptive Retrieval (dynamic when/what/how-much-to-retrieve).

---

## Topics

| Topic | Description | Key Papers |
|-------|-------------|------------|
| [**GraphRAG**](./GraphRAG.md) | Graph-based RAG using knowledge graphs + community summaries for holistic reasoning | GraphRAG (Microsoft), LightRAG (EMNLP 2025), LazyGraphRAG |
| [**Adaptive Retrieval**](./Adaptive-Retrieval.md) | Systems that dynamically decide when, what, and how much to retrieve | Self-RAG (ICLR 2024), FLARE (EMNLP 2023), CRAG, DRAGIN, Adaptive-RAG |

---

## Quick Reference

### RAG Evolution Timeline

| Generation | Approach | Key Limitation |
|------------|----------|----------------|
| **Naive RAG** | Fixed retrieve-then-generate | Retrieves even when unnecessary; can't handle multi-hop or global questions |
| **Graph RAG** | Knowledge graph + community summaries | Better global reasoning but high indexing cost |
| **Adaptive RAG** | Dynamic retrieval decisions | Solves *when* to retrieve, but doesn't solve global reasoning |
| **Corrective RAG** | Evaluate + correct retrieved docs | Solves *what* to retrieve, but doesn't address global questions |
| **LazyGraphRAG** | Deferred LLM + NLP extraction | 0.1% indexing cost of GraphRAG, best quality at 4% query cost |

### Graph RAG vs Adaptive RAG vs Corrective RAG

| Aspect | Graph RAG | Adaptive RAG | Corrective RAG |
|--------|-----------|-------------|----------------|
| **What it solves** | Global reasoning, multi-hop | When to retrieve | Quality of retrieved docs |
| **Key mechanism** | KG + community summaries | Complexity-based routing | Confidence gating + correction |
| **Training needed** | No (pipeline) | Yes (Self-RAG) or No (DRAGIN) | No (plug-and-play) |
| **Cost profile** | High indexing, moderate query | Low or moderate per-query | Low per-query |
| **Best combined with** | Adaptive retrieval | Graph retrieval | Both |

### Key Repos Quick Reference

| Repo | ⭐ | Method |
|------|-----|--------|
| [microsoft/graphrag](https://github.com/microsoft/graphrag) | 32,353 | Microsoft GraphRAG |
| [HKUDS/LightRAG](https://github.com/HKUDS/LightRAG) | 33,850 | LightRAG (EMNLP 2025) |
| [gusye1234/nano-graphrag](https://github.com/gusye1234/nano-graphrag) | 3,800 | Minimal GraphRAG (~1100 LOC) |
| [AkariAsai/self-rag](https://github.com/AkariAsai/self-rag) | 2,361 | Self-RAG (ICLR 2024) |
| [jzbjyb/FLARE](https://github.com/jzbjyb/FLARE) | 667 | FLARE (EMNLP 2023) |
| [HuskyInSalt/CRAG](https://github.com/HuskyInSalt/CRAG) | 453 | Corrective RAG |
| [starsuzi/Adaptive-RAG](https://github.com/starsuzi/Adaptive-RAG) | 383 | Adaptive-RAG (NAACL 2024) |
| [oneal2000/DRAGIN](https://github.com/oneal2000/DRAGIN) | 191 | DRAGIN (ACL 2024) |

---

*Last updated: 2026-04-20*