# quality-filtering

## 📌 Overview

Methods for filtering and curating high-quality training data using embedding-based techniques. Covers semantic scoring (quality classifiers, similarity thresholds) and embedding-pruning (cluster-level signals, centroid distance, diversity).

> *"Can remove up to 50% of the data with minimal performance loss using embedding-based semantic deduplication."*
> — [SemDeDup ICLR 2023](https://openreview.net/pdf?id=IRSesTQUtb)

---

## Topics

| Topic | Description |
|-------|-------------|
| [Semantic-Scoring.md](./Semantic-Scoring.md) | Semantic similarity, FineWeb-Edu classifier, CQF, reward models |
| [Embedding-Pruning.md](./Embedding-Pruning.md) | Cluster-based pruning, SCIP, D², SIEVE, DBP |

---

## Taxonomy Matrix

| Method | Type | Core Signal | Result |
|-------|------|-------------|--------|
| **SemDeDup** | Semantic dedup | Pairwise cosine similarity | 50% reduction, minimal loss |
| **FineWeb-Edu** | Classifier | Educational quality score (0-5) | F1 82% at threshold 3 |
| **CQF** | Classifier | Likelihood ratio p_HQ/p_LQ | Improves downstream tasks |
| **SCIP** | Pruning | Embedding change under corruption | +3% HumanEval |
| **D² Pruning** | Coreset | Message passing graph | 71.8% at 50% pruning |
| **SIEVE** | Pruning | Synthetic caption similarity | +1.8 avg on DataComp |
| **DBP** | Pruning | Inter/intra cluster distance | 166M subset > 440M full |
| **CLIP-Score** | Filtering | Image-text cosine similarity | DataComp baseline |

---

## Quality Signals

| Signal | Indicates |
|--------|-----------|
| Small cluster membership | Outlier data |
| Large centroid distance | Poor representation |
| Low caption similarity | Misaligned |
| High corruption variance | Syntactic issues |

---

## Related Topics

- [Synthetic-flywheels](./Synthetic-flywheels/) — Data generation and iterative improvement
