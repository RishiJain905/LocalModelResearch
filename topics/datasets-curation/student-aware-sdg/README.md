# student-aware-sdg

> Synthetic data generation techniques that are **aware of the student model's knowledge state** and generate targeted training data accordingly.

## 📂 Contents

| Document | Description |
|----------|-------------|
| [Iterative-Targeted-Generation.md](./Iterative-Targeted-Generation.md) | Cyclical feedback loop: identify weaknesses → generate targeted data → re-train → repeat |
| [Loss-Seeding.md](./Loss-Seeding.md) | Using high-loss examples as "seeds" for targeted synthetic data generation |

## 🔑 Key Themes

### Iterative Targeted Generation
- **TDG (ACL 2023):** Cluster-based weakness identification → LLM generation for weak subgroups
- **DPE:** Spiral loop — Diagnosis → Generation → Reinforcement → Re-Diagnosis
- **Active SDG:** Per-step loss-guided generation, 1.3x–2x more efficient than static

### Loss-Seeding
- **Max-Loss Coreset:** Augment highest-loss examples → 6.3x speedup
- **USAGE:** Seed area generation via loss optimization
- **SDG AttributePrompt:** Loss-weighted synthetic data for weak classes

## 📊 Taxonomy Matrix

| Technique | Core Signal | Iteration | Domain |
|-----------|-------------|-----------|--------|
| TDG | Cluster failure analysis | Batch | NLP |
| DPE | Diagnostic evaluation | Spiral | Multimodal |
| Active SDG | Loss/uncertainty | Per-step | LLM finetuning |
| Max-Loss Coreset | Highest loss examples | Per-step | General ML |
| USAGE | Generation loss | Optimization | Image seg |
| SDG AttributePrompt | Per-class loss | Generation | NLP classif |

## 🔗 Related Folder

← Back to [datasets-curation](../README.md)
