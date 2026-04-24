# Semantic Scoring

## 📌 Overview

**Semantic scoring** uses embedding-based similarity and classifier methods to evaluate and filter data quality. It assigns continuous quality scores to samples based on semantic properties — including educational value, alignment, and likelihood ratios — enabling targeted filtering of high-quality data.

> *"Uses pre-trained embeddings to identify and remove semantic duplicates — data pairs that are highly similar semantically but not exactly identical."*
> — [SemDeDup (ICLR 2023)](https://openreview.net/pdf?id=IRSesTQUtb)

---

## SemDeDup (Semantic Deduplication) — ICLR 2023

### Method

| Stage | Description |
|-------|-------------|
| **1. Embed** | Generate embeddings using pretrained model |
| **2. Cluster** | K-means clustering in embedding space |
| **3. Compute similarity** | Pairwise cosine similarity within clusters |
| **4. Apply threshold** | Keep representative, remove semantic duplicates |

> *"Leverages pre-trained embeddings to identify and remove semantic duplicates — data pairs that are highly similar semantically but not exactly identical."*
> — [SemDeDup Paper](https://openreview.net/pdf?id=IRSesTQUtb)

### Thresholds

| ε Value | Deduplication Level | Data Reduction |
|---------|---------------------|-----------------|
| 0.001 | Less aggressive | ~37% removal, zero performance drop |
| 0.1 | More aggressive | ~50% removal, minimal loss |

> *"Can remove up to 50% of the data with minimal performance loss."*
> — [SemDeDup Paper](https://openreview.net/pdf?id=IRSesTQUtb)

### Results

> *"50% data reduction with minimal performance loss on LAION-440M."*
> — [SemDeDup Paper](https://openreview.net/pdf?id=IRSesTQUtb)

---

## FineWeb-Edu Classifier

### Method

> *"Snowflake-arctic-embed as backbone + classification head trained on 450K Llama3-70B-instruct annotations."*
> — [HuggingFace FineWeb-Edu](https://huggingface.co/HuggingFaceFW/fineweb-edu-classifier)

### Scoring

| Score Range | Meaning |
|-------------|---------|
| 0-5 | Continuous educational quality score |
| ≥3 | Binary threshold for high-quality |

> *"The model achieved an F1 score of 82% when converted to a binary classifier using a score threshold of 3."*
> — [HuggingFace FineWeb-Edu](https://huggingface.co/HuggingFaceFW/fineweb-edu-classifier)

**Application:** Filtering FineWeb dataset to create high-quality educational subset (~1.3T tokens).

---

## Classifier-Based Quality Filtering (CQF)

### Method

> *"Trains binary classifier on high-quality (HQ) vs low-quality (LQ) data; retains top-scoring documents."*
> — [Apple Researchers arXiv:2510.00866](https://arxiv.org/html/2510.00866v1)

### Key Finding

> *"Although CQF consistently improves performance on downstream tasks, it does not necessarily improve language modeling on the HQ set."*
> — [Apple CQF Paper](https://arxiv.org/html/2510.00866v1)

> *"CQF ranks by likelihood ratio p_HQ/p_LQ, not pure quality — filters data far from LQ set."*
> — [Apple CQF Paper](https://arxiv.org/html/2510.00866v1)

---

## NVIDIA NeMo Curator — Semantic Deduplication

### Pipeline

```
Embedding generation → Clustering → Similarity computation → Duplicate identification → Removal
```

> *"Typical reduction: 20-50% dataset size while maintaining/increasing model performance."*
> — [NVIDIA NeMo Semantic Deduplication](https://docs.nvidia.com/nemo-framework/user-guide/25.07/datacuration/semdedup.html)

### Parameters

| Parameter | Description |
|-----------|-------------|
| `eps_to_extract` | Similarity threshold |
| `which_to_keep` | Strategy: hard/easy/random |

---

## Cosine Similarity Thresholds

| Use Case | Typical Threshold |
|----------|------------------|
| Semantic filtering | 0.70-0.80 |
| High-quality matches | ≥0.80 |
| Near-exact deduplication | ≥0.95 |

> *"Minimum threshold (typically 0.70 or above score) to filter weak matches."*
> — [NVIDIA NeMo](https://docs.nvidia.com/nemo-framework/user-guide/25.07/datacuration/semdedup.html)

---

## Reward Model-Based Curation

> *"Small specialized models ('curators') trained to evaluate specific attributes (correctness, reasoning quality, coherence, readability)."*
> — [Collinear Blog](https://blog.collinear.ai/p/post-training-data-curation)

> *"~2× speedup with same downstream accuracy using <50% of curated data."*
> — [Collinear Blog](https://blog.collinear.ai/p/post-training-data-curation)

---

## Summary Table

| Method | Metric | Typical Threshold/Result |
|--------|--------|-------------------------|
| **SemDeDup** | Cosine similarity (1-ε) | ε: 0.001-0.1 → 37-50% reduction |
| **FineWeb-Edu** | Quality score 0-5 | ≥3 binary threshold |
| **CQF** | Likelihood ratio p_HQ/p_LQ | Improves downstream tasks |
| **NeMo SemDedup** | Cosine similarity | 20-50% dataset reduction |
| **Reward Model** | Specialized attribute scores | ~2× speedup |

---

## Sources

| Type | Citation |
|------|----------|
| **Paper** | [SemDeDup ICLR 2023](https://openreview.net/pdf?id=IRSesTQUtb) |
| **Model** | [HuggingFace FineWeb-Edu Classifier](https://huggingface.co/HuggingFaceFW/fineweb-edu-classifier) |
| **Paper** | [Apple CQF arXiv:2510.00866](https://arxiv.org/html/2510.00866v1) |
| **Docs** | [NVIDIA NeMo Semantic Deduplication](https://docs.nvidia.com/nemo-framework/user-guide/25.07/datacuration/semdedup.html) |
| **Blog** | [Collinear — Post-training Data Curation](https://blog.collinear.ai/p/post-training-data-curation) |

---

## Related Topics

- [Embedding-Pruning.md](./Embedding-Pruning.md) — Pruning via embedding space properties
