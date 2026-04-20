# GraphRAG

> **Paper:** [arXiv:2404.16130](https://arxiv.org/abs/2404.16130) (Microsoft, 2024)
> **GitHub:** ⭐32,353 [microsoft/graphrag](https://github.com/microsoft/graphrag) · ⭐33,850 [HKUDS/LightRAG](https://github.com/HKUDS/LightRAG) · ⭐3,800 [gusye1234/nano-graphrag](https://github.com/gusye1234/nano-graphrag)
> **Blogs:** [Microsoft Research — Unlocking LLM Discovery](https://www.microsoft.com/en-us/research/blog/graphrag-unlocking-llm-discovery-on-narrative-private-data/) · [Introducing DRIFT Search](https://www.microsoft.com/en-us/research/blog/introducing-drift-search-combining-global-and-local-search-methods-to-improve-quality-and-efficiency/) · [LazyGraphRAG](https://www.microsoft.com/en-us/research/blog/lazygraphrag-setting-a-new-standard-for-quality-and-cost/)
> **Docs:** [microsoft.github.io/graphrag](https://microsoft.github.io/graphrag/)

---

## Overview

> *"GraphRAG leads to substantial improvements over a conventional RAG baseline for both the comprehensiveness and diversity of generated answers."* — Edge et al., 2024

GraphRAG (Graph-based Retrieval-Augmented Generation) is Microsoft's approach to question answering over private text corpora that uses **knowledge graphs + community summaries + graph ML** to augment LLM prompts. Standard RAG fails on "connecting the dots" across disparate information and on holistic understanding of large collections — GraphRAG explicitly addresses both.

---

## Key Papers

### 1. From Local to Global: A Graph RAG Approach to Query-Focused Summarization (2024) ★ Primary

> **Paper:** [arXiv:2404.16130](https://arxiv.org/abs/2404.16130) · **Authors:** Darren Edge, Ha Trinh, Newman Cheng, Joshua Bradley, Alex Chao, Apurva Mody, Steven Truitt, Jonathan Larson (Microsoft)
> **GitHub:** ⭐32,353 [microsoft/graphrag](https://github.com/microsoft/graphrag)

The foundational GraphRAG paper. Builds a knowledge graph from source documents using LLM extraction, detects hierarchical communities via Leiden algorithm, generates community summaries bottom-up, and answers queries via map-reduce over community summaries.

> *"GraphRAG uses knowledge graphs to provide substantial improvements in question-and-answer performance when reasoning about complex information."*

### 2. Retrieval-Augmented Generation with Graphs — Survey (2025)

> **Paper:** [arXiv:2501.00309](https://arxiv.org/abs/2501.00309) · **Authors:** Haoyu Han and 17 others

Proposes a holistic GraphRAG framework with **5 components**: Query Processor, Retriever, Organizer, Generator, Data Source. Covers domains: Knowledge Graphs, Document-level Graphs, Table/Schema Graphs, Code/AST Graphs, Social Networks, Molecular/Scientific Graphs.

### 3. LightRAG: Simple and Fast Retrieval-Augmented Generation (2025)

> **Paper:** [arXiv:2410.05779](https://arxiv.org/abs/2410.05779) · **Venue:** EMNLP 2025 · **GitHub:** ⭐33,850 [HKUDS/LightRAG](https://github.com/HKUDS/LightRAG)

Dual-level retrieval (low-level entities + high-level topics) over knowledge graphs. Outperforms NaiveRAG, RQ-RAG, HyDE, and GraphRAG across multiple domains. Latest v1.4.13 (Apr 2026) supports OpenSearch, MongoDB, PostgreSQL, Neo4j backends.

> **Minimum LLM:** 32B parameters, ≥32KB context length recommended.

### 4. LazyGraphRAG (Microsoft Evolution)

> **Blog:** [LazyGraphRAG — Setting a New Standard](https://www.microsoft.com/en-us/research/blog/lazygraphrag-setting-a-new-standard-for-quality-and-cost/)

Radically different approach requiring **no prior summarization** of source data:
- **Indexing costs:** Identical to vector RAG; **0.1% of full GraphRAG** costs
- Uses **NLP noun phrase extraction** instead of LLM extraction
- Defers all LLM use until query time ("lazy")
- **At 4% of GraphRAG query cost:** significantly outperforms ALL competing methods on both local and global queries
- **At comparable cost to vector RAG:** outperforms all methods on local queries
- Now integrated into **Microsoft Discovery** (Azure) and **Azure Local**

---

## Papers Table

| Paper | Year | Venue | URL | Key Contribution |
|-------|------|-------|-----|------------------|
| **From Local to Global (GraphRAG)** | 2024 | arXiv | [arXiv:2404.16130](https://arxiv.org/abs/2404.16130) | Foundational paper — KG + communities + map-reduce |
| **GraphRAG Survey** | 2025 | arXiv | [arXiv:2501.00309](https://arxiv.org/abs/2501.00309) | Holistic 5-component framework |
| **LightRAG** | 2025 | EMNLP | [arXiv:2410.05779](https://arxiv.org/abs/2410.05779) | Dual-level retrieval, outperforms GraphRAG |

---

## Graph Indexing Pipeline

From the paper (arXiv:2404.16130):

| Step | Process |
|------|---------|
| **1. Chunking** | Source documents → TextUnits (default: 600-token chunks with 100-token overlap). Longer chunks = fewer LLM calls but degraded recall |
| **2. Entity/Relationship/Claim Extraction** | LLM extracts **Entities** ("NeoChip — publicly traded company specializing in low-power processors"), **Relationships** ("Quantum Systems owned NeoChip"), **Claims** ("NeoChip's shares surged during their first week"). Supports **self-reflection/gleaming**: LLM re-checks for missed entities |
| **3. Knowledge Graph Construction** | Extracted elements → nodes + edges. Entity descriptions aggregated/summarized per node. Relationship duplicates become edge weights. Uses **exact string matching** for entity resolution |
| **4. Community Detection** | **Leiden algorithm** (hierarchical) recursively detects sub-communities until leaf communities can't be further partitioned. Each hierarchy level = mutually exclusive, collectively exhaustive partition |
| **5. Community Summaries** | Leaf communities prioritized by combined source/target node degree. Higher-level summaries substitute shorter sub-summaries until context fits |
| **6. Embedding Generation** | Embeddings created for text units, entity descriptions, and full community content |

---

## Search Modes

| Mode | Purpose | Method |
|------|---------|--------|
| **Global Search** | Holistic questions about entire corpus | Map-reduce over community summaries → shuffle into fixed-size chunks → parallel intermediate answers (scored 0-100 helpfulness) → reduce to final global answer |
| **Local Search** | Specific entity-focused questions | Fans out from matching entities to neighbors and associated concepts |
| **DRIFT Search** | Local + community context | **D**ynamic **R**easoning and **I**nference with **F**lexible **T**raversal. (1) Primer: HyDE + top-K community reports → initial answer + follow-ups; (2) Follow-up: local search per question; (3) Ranked hierarchy. Outperforms Local Search **78%** on comprehensiveness, **81%** on diversity |
| **Basic Search** | Simple queries | Standard top-k vector search (baseline RAG) |

---

## GitHub Repos

| Repo | ⭐ | Description |
|------|-----|-------------|
| [microsoft/graphrag](https://github.com/microsoft/graphrag) | 32,353 | Official Microsoft GraphRAG — data pipeline + transformation suite. Python 88.2%, Jupyter 11.8%. MIT license. Latest v3.0.9 |
| [HKUDS/LightRAG](https://github.com/HKUDS/LightRAG) | 33,850 | EMNLP 2025 — dual-level retrieval. Supports OpenSearch, MongoDB, PostgreSQL, Neo4j. WebUI + RAGAS eval |
| [gusye1234/nano-graphrag](https://github.com/gusye1234/nano-graphrag) | 3,800 | ~1100 lines of code. Supports incremental insert, naive/local/global search, async, custom LLM/embedding backends |
| [neo4j-contrib/ms-graphrag-neo4j](https://github.com/neo4j-contrib/ms-graphrag-neo4j) | 86 | Neo4j backend implementation. Uses OpenAI models + Neo4j GDS library. Experimental |
| [microsoft/graphrag-benchmarking-datasets](https://github.com/microsoft/graphrag-benchmarking-datasets) | 4 | Benchmarking datasets for graph-based RAG |

---

## Graph RAG vs Vector RAG

| Aspect | Vector RAG | Graph RAG |
|--------|-----------|-----------|
| **Representation** | Dense vector embeddings | Nodes (entities) + edges (relationships) |
| **Retrieval method** | Semantic similarity search | Graph traversal + community summaries |
| **Multi-hop reasoning** | Weak — no native entity relationship understanding | Strong — explicitly models relationships |
| **Global/holistic queries** | Fails — can't summarize across corpus | Strong — community hierarchy covers entire dataset |
| **Explainability** | Low — similarity scores opaque | High — can trace query path through graph |
| **Setup complexity** | Low | High — requires KG construction + community detection |
| **Cost** | Lower (embedding + retrieval) | Higher (LLM extraction + summarization) |
| **Best for** | Large-scale unstructured data, simple similarity | Complex relationships, structured reasoning, global questions |

---

## Benchmarking & Evaluation

**Datasets:** Podcast transcripts (~1M tokens), News articles (~1.7M tokens), AP News (5,590 articles for LazyGraphRAG)

**Metrics:** Comprehensiveness, Diversity, Empowerment, Directness, Faithfulness (via SelfCheckGPT)

**Evaluation method:** LLM-as-judge head-to-head comparisons

**Key result (from launch blog):**
- Query: *"What has Novorossiya done?"*
- **Baseline RAG:** *"The text does not provide specific information."*
- **GraphRAG:** Provided detailed answer with provenance citations

---

## LazyGraphRAG Cost Comparison

| Method | Indexing Cost | Query Cost | Local Quality | Global Quality |
|--------|-------------|-----------|--------------|----------------|
| **Vector RAG** | Baseline | Baseline | Baseline | Baseline |
| **Full GraphRAG** | ~1000× vector RAG | ~4× vector RAG | High | Very High |
| **LazyGraphRAG @ 4% GraphRAG cost** | Same as vector RAG | 4% of GraphRAG | Outperforms all | Outperforms all |
| **LazyGraphRAG @ vector RAG cost** | Same as vector RAG | Same as vector RAG | Outperforms all | High |

---

## Blog Posts & Technical Reports

| Title | Source | URL |
|-------|--------|-----|
| **GraphRAG: Unlocking LLM Discovery on Narrative Private Data** | Microsoft Research | [Link](https://www.microsoft.com/en-us/research/blog/graphrag-unlocking-llm-discovery-on-narrative-private-data/) |
| **GraphRAG: New Tool on GitHub** | Microsoft Research | [Link](https://www.microsoft.com/en-us/research/blog/graphrag-new-tool-for-complex-data-discovery-now-on-github/) |
| **GraphRAG Auto-Tuning for New Domains** | Microsoft Research | [Link](https://www.microsoft.com/en-us/research/blog/graphrag-auto-tuning-provides-rapid-adaptation-to-new-domains/) |
| **Introducing DRIFT Search** | Microsoft Research | [Link](https://www.microsoft.com/en-us/research/blog/introducing-drift-search-combining-global-and-local-search-methods-to-improve-quality-and-efficiency/) |
| **LazyGraphRAG: New Standard for Quality & Cost** | Microsoft Research | [Link](https://www.microsoft.com/en-us/research/blog/lazygraphrag-setting-a-new-standard-for-quality-and-cost/) |
| **Intro to GraphRAG** | Neo4j (graphrag.com) | [Link](https://graphrag.com/concepts/intro-to-graphrag/) |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **What it is** | Graph-based RAG that builds knowledge graphs from documents, detects communities, and answers via community summaries |
| **Key advantage** | Holistic understanding + multi-hop reasoning that vector RAG cannot achieve |
| **Search modes** | Global (map-reduce), Local (entity-centric), DRIFT (hybrid), Basic (vector baseline) |
| **Latest evolution** | LazyGraphRAG — 0.1% indexing cost of full GraphRAG, outperforms all methods at 4% query cost |
| **Best competitor** | LightRAG (⭐33,850) — dual-level retrieval, EMNLP 2025, outperforms GraphRAG on multiple metrics |
| **Tradeoff** | Higher setup cost and complexity vs. significantly better global reasoning and explainability |

---

*Last updated: 2026-04-20*