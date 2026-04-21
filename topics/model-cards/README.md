# Model Cards

> Standardized documentation for AI models — covering model identity, evaluation, compliance, monitoring, and discovery metadata.

## 📚 Contents

### [Automated & Machine-Readable Artifacts](./Automated-Machine-Readable-Artifacts/)

Research on automated, machine-readable documentation standards and tooling for AI models — including drift tracking, regulatory compliance, and metadata schemas.

| Subtopic | Description |
|----------|-------------|
| [Dynamic Drift Tracking](./Automated-Machine-Readable-Artifacts/Dynamic-Drift-Tracking.md) | Real-time model monitoring, concept/data drift detection, and automated artifact generation |
| [Regulatory Passports](./Automated-Machine-Readable-Artifacts/Regulatory-Passports.md) | EU AI Act, NIST AI RMF, ISO standards, model cards, and compliance documentation |
| [Keywords](./Automated-Machine-Readable-Artifacts/Keywords.md) | Metadata standards, taxonomy frameworks, automated tagging, and model discovery |

---

### [Training Data Lineage & Provenance](./Training-Data-Lineage-Provenance/)

Research on how AI training data is documented, traced, licensed, and audited — covering dataset composition, copyright disclosures, and contamination reporting.

| Subtopic | Description |
|----------|-------------|
| [Dataset Composition Analysis](./Training-Data-Lineage-Provenance/Dataset-Composition-Analysis.md) | Dataset documentation frameworks, provenance tracking, and composition transparency |
| [Copyright & Public Domain Disclosures](./Training-Data-Lineage-Provenance/Copyright-Public-Domain-Disclosures.md) | Training data licensing, copyright law, fair use, and rights disclosures |
| [Data Contamination Reporting](./Training-Data-Lineage-Provenance/Data-Contamination-Reporting.md) | Benchmark contamination detection, test set leakage, and model evaluation integrity |

---

## Overview

**Model Cards** originated from Google's 2019 FAT* paper — short documents accompanying trained ML models that provide structured evaluation across demographic/functional subgroups:

> *"Model Cards for Model Reporting" — Mitchell, Wu, Zaldivar, Barnes, Vasserman, Hutchinson, Spitzer, Raji, Gebru — [arXiv:1810.03993](https://arxiv.org/abs/1810.03993)

Model cards serve as **boundary objects** between diverse stakeholders: researchers, deployers, auditors, and end users.

### Key Findings

- **Adoption is high, completeness is low**: Only ~44% of Hugging Face models have cards; critical sections severely under-filled — Limitations (17.4%), Evaluation (15.4%), Environmental Impact (2.0%)
- **Machine-readable formats emerging**: JSON Schema, YAML frontmatter, Protocol Buffers, and RDF/OWL Semantic Web specs
- **Automation is active research**: LLMs (GPT-4) outperform humans on completeness but lag on accuracy due to hallucination ([CardGen, NAACL 2024](https://arxiv.org/html/2405.06258v1))
- **Regulatory driver**: EU AI Act Annex IV requirements are shaping documentation standards for high-risk AI systems

---

## Related Topics

- [Quantization](../quantization/) — Model size, quality/size tradeoffs
- [Inference Engines](../inference/) — Serving, monitoring, and runtime
- [Benchmarks](../benchmarks/) — Evaluation frameworks and metrics
- [Safety](../safety/) — Responsible AI, red-teaming, and compliance
