# Adaptive Retrieval

> **Papers:** [Self-RAG (ICLR 2024)](https://arxiv.org/abs/2310.11511) · [FLARE (EMNLP 2023)](https://arxiv.org/abs/2305.06983) · [Adaptive-RAG (NAACL 2024)](https://arxiv.org/abs/2403.14403) · [CRAG](https://arxiv.org/abs/2401.15884) · [DRAGIN (ACL 2024)](https://arxiv.org/abs/2403.10081) · [UAR (EMNLP 2024)](https://arxiv.org/abs/2406.12534)
> **GitHub:** ⭐2,361 [AkariAsai/self-rag](https://github.com/AkariAsai/self-rag) · ⭐667 [jzbjyb/FLARE](https://github.com/jzbjyb/FLARE) · ⭐453 [HuskyInSalt/CRAG](https://github.com/HuskyInSalt/CRAG) · ⭐383 [starsuzi/Adaptive-RAG](https://github.com/starsuzi/Adaptive-RAG)
> **Blogs:** [LangGraph Adaptive RAG Tutorial](https://github.com/langchain-ai/langgraph/blob/main/examples/rag/langgraph_adaptive_rag_cohere.ipynb) · [Learn Prompting — CRAG](https://learnprompting.org/docs/retrieval_augmented_generation/corrective-rag) · [Learn Prompting — FLARE](https://learnprompting.org/docs/retrieval_augmented_generation/flare-active-rag)

---

## Overview

> *"Indiscriminately retrieving and incorporating a fixed number of retrieved passages, regardless of whether retrieval is necessary, or passages are relevant, diminishes LM versatility or can lead to unhelpful response generation."* — Self-RAG, ICLR 2024

Adaptive Retrieval is a family of RAG methods that **dynamically decide when, what, and how much to retrieve** during generation, rather than always retrieving a fixed set of documents. Standard RAG retrieves once based on input — adaptive methods retrieve on-demand, iteratively, or conditionally based on the model's real-time needs.

---

## Key Papers

### 1. Self-RAG: Learning to Retrieve, Generate, and Critique through Self-Reflection (2024) ★

> **Paper:** [arXiv:2310.11511](https://arxiv.org/abs/2310.11511) · **Authors:** Akari Asai, Zeqiu Wu, Yizhong Wang, Avirup Sil, Hannaneh Hajishirzi
> **Venue:** ICLR 2024 (Oral, top 1%) · **GitHub:** ⭐2,361 [AkariAsai/self-rag](https://github.com/AkariAsai/self-rag) · **Website:** [selfrag.github.io](https://selfrag.github.io/)

Trains a single LM that **adaptively retrieves passages on-demand** using special `[Retrieve]` tokens, then generates and self-reflects using four types of **reflection tokens**:

| Token | What It Signals |
|-------|----------------|
| `isrelay` | Whether the generation is supported by retrieved passages |
| `isfact` | Whether the output is factually grounded |
| `iscomplete` | Whether the response is complete |
| `ishas` | Whether retrieval was necessary |

> *"Self-RAG retrieves on demand (can retrieve multiple times or skip entirely) and criticizes its own generation by predicting reflection tokens as part of generation."*

**Key mechanism:** Tree-based decoding with segment-wise beam search to maximize utility for diverse preferences.

### 2. FLARE: Active Retrieval Augmented Generation (2023)

> **Paper:** [arXiv:2305.06983](https://arxiv.org/abs/2305.06983) · **Authors:** Zhengbao Jiang, Frank F. Xu, Luyu Gao, Zhiqing Sun, Qian Liu, Jane Dwivedi-Yu, Yiming Yang, Jamie Callan, Graham Neubig
> **Venue:** EMNLP 2023 · **GitHub:** ⭐667 [jzbjyb/FLARE](https://github.com/jzbjyb/FLARE)

> *"FLARE is a generic retrieval-augmented generation method that actively decides when and what to retrieve. It achieves this by predicting the upcoming sentence to anticipate future content, then using that prediction as a query to retrieve relevant documents if it contains low-confidence tokens."*

**Core idea:** Iterative forward-looking retrieval:
1. Generate a tentative next sentence
2. Check for low-probability tokens
3. If found → use tentative sentence as search query → retrieve documents → regenerate the sentence with context
4. If not found → accept the generated sentence

### 3. Adaptive-RAG: Learning to Adapt Retrieval through Question Complexity (2024)

> **Paper:** [arXiv:2403.14403](https://arxiv.org/abs/2403.14403) · **Authors:** Soyeong Jeong, Jinheon Baek, Sukmin Cho, Sung Ju Hwang, Jong C. Park
> **Venue:** NAACL 2024 · **GitHub:** ⭐383 [starsuzi/Adaptive-RAG](https://github.com/starsuzi/Adaptive-RAG)

> *"We propose a novel adaptive QA framework that can dynamically select the most suitable strategy for (retrieval-augmented) LLMs from the simplest to the most complex, based on question complexity."*

A **classifier trained on query complexity** routes among three retrieval strategies:

| Strategy | When | Action |
|----------|------|--------|
| **No retrieval** | Simple questions | Answer directly from model knowledge |
| **Single-step retrieval** | Moderate questions | Retrieve once, then generate |
| **Multi-step retrieval** | Complex questions | Iterate retrieve-generate cycles |

### 4. CRAG: Corrective Retrieval Augmented Generation (2024)

> **Paper:** [arXiv:2401.15884](https://arxiv.org/abs/2401.15884) · **Authors:** Shi-Qi Yan, Jia-Chen Gu, Yun Zhu, Zhen-Hua Ling
> **GitHub:** ⭐453 [HuskyInSalt/CRAG](https://github.com/HuskyInSalt/CRAG)

> *"CRAG improves robustness by introducing a retrieval evaluator that assesses the quality of retrieved documents and triggers different knowledge retrieval actions: Correct (refine), Incorrect (web search), Ambiguous (combine both)."*

**Core mechanism:** A lightweight retrieval evaluator categorizes documents into three confidence levels:

| Confidence | Action |
|-----------|--------|
| ✅ **Correct** | Refine retrieved docs via decompose-then-recompose |
| ❌ **Incorrect** | Fall back to web search |
| ⚠️ **Ambiguous** | Combine both retrieved + web search |

Also introduces a **decompose-then-recompose** algorithm to filter noise from retrieved documents. **Plug-and-play** — can be combined with any RAG approach.

### 5. DRAGIN: Dynamic Retrieval Based on LLM Information Needs (2024)

> **Paper:** [arXiv:2403.10081](https://arxiv.org/abs/2403.10081) · **Authors:** Yutao Su, Zhiyuan Tang, Qiutong Su, et al.
> **Venue:** ACL 2024 Main Conference (Oral) · **GitHub:** ⭐191 [oneal2000/DRAGIN](https://github.com/oneal2000/DRAGIN)

> *"DRAGIN can be incorporated into any Transformer-based LLMs without further training, fine-tuning, or prompt engineering."*

Uses the LLM's **own internal signals** (attention weights + token probabilities) during generation to:
1. **Determine when to retrieve** — based on real-time information needs
2. **Formulate what to retrieve** — queries derived from model's internal state, not just input text

### 6. UAR: Unified Active Retrieval for RAG (2024)

> **Paper:** [arXiv:2406.12534](https://arxiv.org/abs/2406.12534) · **Authors:** Qinyuan Cheng et al.
> **Venue:** EMNLP 2024 Findings

> *"UAR contains four orthogonal criteria and casts them into plug-and-play classification tasks, which achieves multifaceted retrieval timing judgements with negligible extra inference cost."*

Introduces **UAR-Criteria** — a standardized procedure for four orthogonal active retrieval dimensions. Also curates an **Active Retrieval benchmark (AR-Bench)** for evaluation.

### 7. RAGate: Adaptive RAG for Conversational Systems (2024)

> **Paper:** [arXiv:2407.21712](https://arxiv.org/abs/2407.21712) · **Authors:** Xuan Wang et al.

> *"We develop RAGate, a gating model, which models conversation context and relevant inputs to predict if a conversational system requires RAG for improved response quality."*

A **gating model** trained on human judgments to decide per conversational turn whether external knowledge retrieval is needed.

---

## Papers Table

| Paper | Year | Venue | URL | Key Mechanism |
|-------|------|-------|-----|---------------|
| **Self-RAG** | 2024 | ICLR 2024 (Oral) | [arXiv:2310.11511](https://arxiv.org/abs/2310.11511) | `[Retrieve]` tokens + 4 reflection tokens + beam search |
| **FLARE** | 2023 | EMNLP 2023 | [arXiv:2305.06983](https://arxiv.org/abs/2305.06983) | Low-confidence tokens trigger forward-looking retrieval |
| **Adaptive-RAG** | 2024 | NAACL 2024 | [arXiv:2403.14403](https://arxiv.org/abs/2403.14403) | Question-complexity classifier → route to strategy |
| **CRAG** | 2024 | arXiv | [arXiv:2401.15884](https://arxiv.org/abs/2401.15884) | Retrieval evaluator → Correct/Incorrect/Ambiguous actions |
| **DRAGIN** | 2024 | ACL 2024 (Oral) | [arXiv:2403.10081](https://arxiv.org/abs/2403.10081) | Internal attention + perplexity signals trigger retrieval |
| **UAR** | 2024 | EMNLP 2024 | [arXiv:2406.12534](https://arxiv.org/abs/2406.12534) | Four orthogonal retrieval criteria + AR-Bench |
| **RAGate** | 2024 | arXiv | [arXiv:2407.21712](https://arxiv.org/abs/2407.21712) | Human-judgment-trained gate per conversational turn |

---

## GitHub Repos

| Repo | ⭐ | Paper/Project | Description |
|------|-----|----------------|-------------|
| [AkariAsai/self-rag](https://github.com/AkariAsai/self-rag) | 2,361 | Self-RAG (ICLR 2024) | Original Self-RAG implementation with reflection tokens + beam search |
| [jzbjyb/FLARE](https://github.com/jzbjyb/FLARE) | 667 | FLARE (EMNLP 2023) | Forward-Looking Active REtrieval — iterative confidence-based retrieval |
| [HuskyInSalt/CRAG](https://github.com/HuskyInSalt/CRAG) | 453 | CRAG | Corrective RAG — retrieval evaluator + decompose-then-recompose |
| [starsuzi/Adaptive-RAG](https://github.com/starsuzi/Adaptive-RAG) | 383 | Adaptive-RAG (NAACL 2024) | Question-complexity classifier routes retrieval strategy |
| [oneal2000/DRAGIN](https://github.com/oneal2000/DRAGIN) | 191 | DRAGIN (ACL 2024) | Dynamic retrieval from internal LLM signals — no training needed |
| [alonsoir/crag_srag_arag](https://github.com/alonsoir/crag_srag_arag) | — | CRAG + Self-RAG + Adaptive-RAG | LangGraph implementations of all three methods |

---

## Taxonomy of Strategies

| Strategy | When to Retrieve? | What to Retrieve? | Key Mechanism |
|----------|-------------------|-------------------|---------------|
| **Self-RAG** | Model decides via `[Retrieve]` token | Any relevant passage (Contriever) | Reflection tokens + beam search |
| **FLARE** | Low-confidence tokens detected | Tentative next sentence as query | Probability-threshold triggering + sentence regeneration |
| **Adaptive-RAG** | Classifier routes by question complexity | No/single/multi-step retrieval based on complexity | Question-complexity classifier → strategy |
| **CRAG** | Always retrieves, then filters/corrects | Correct: decompose-filter; Incorrect: web search; Ambiguous: both | Confidence gating → corrective action |
| **DRAGIN** | LLM's real-time attention/probability signals | Formulated from LLM's information needs | Internal model signals → dynamic retrieval triggers |
| **UAR** | Four orthogonal criteria (classification tasks) | Plug-and-play across multiple dimensions | Unified Active Retrieval Criteria |
| **RAGate** | Per conversational turn | Context-dependent gating | Human-judgment-trained gate model |

---

## Implementation Resources

| Resource | URL |
|----------|-----|
| **LangGraph Adaptive RAG Tutorial** | [langgraph_adaptive_rag_cohere.ipynb](https://github.com/langchain-ai/langgraph/blob/main/examples/rag/langgraph_adaptive_rag_cohere.ipynb) |
| **Analytics Vidhya — Adaptive RAG with LangGraph** | [analyticsvidhya.com](https://www.analyticsvidhya.com/blog/2025/03/adaptive-rag-systems-with-langgraph/) |
| **Learn Prompting — CRAG** | [learnprompting.org](https://learnprompting.org/docs/retrieval_augmented_generation/corrective-rag) |
| **Learn Prompting — FLARE** | [learnprompting.org](https://learnprompting.org/docs/retrieval_augmented_generation/flare-active-rag) |
| **Athina AI — Active RAG** | [blog.athina.ai](https://blog.athina.ai/active-retrieval-augmented-generation) |
| **Meilisearch — CRAG** | [meilisearch.com](https://www.meilisearch.com/blog/corrective-rag) |
| **Self-RAG Website** | [selfrag.github.io](https://selfrag.github.io/) |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Problem** | Standard RAG retrieves indiscriminately — wastes capacity on easy queries, fails on complex ones |
| **Core idea** | Dynamically decide when, what, and how much to retrieve during generation |
| **Spectrum** | No retrieval (simple) → single-step (moderate) → multi-step (complex) → corrective (unreliable sources) |
| **Best for self-reflection** | Self-RAG — learns when to retrieve via special tokens, critiques its own output |
| **Best for forward-looking** | FLARE — predicts upcoming content, retrieves when confidence is low |
| **Best for plug-and-play** | CRAG / DRAGIN — no training needed, works with any LLM |
| **Best for conversation** | RAGate — per-turn gating trained on human judgments |
| **Key insight** | Different query complexities need different retrieval strategies — one-size-fits-all RAG leaves performance on the table |

---

*Last updated: 2026-04-20*