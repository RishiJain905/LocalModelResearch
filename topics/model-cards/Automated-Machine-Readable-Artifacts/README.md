# Model Cards

> Standardized, machine-readable documentation for AI models — covering model identity, evaluation, compliance, and operational metadata.

## 📚 Topics

| Topic | Description |
|-------|-------------|
| [Dynamic Drift Tracking](./Dynamic-Drift-Tracking.md) | Real-time model monitoring, concept/data drift detection, and automated artifact generation |
| [Regulatory Passports](./Regulatory-Passports.md) | EU AI Act, NIST AI RMF, ISO standards, model cards, and compliance documentation |
| [Keywords](./Keywords.md) | Metadata standards, taxonomy frameworks, automated tagging, and model discovery |

---

## Overview

**Model Cards** are short documents accompanying trained ML models that provide benchmarked evaluation across demographic/functional subgroups. Originating from Google's 2019 FAT* paper:

> *"Model Cards for Model Reporting" — Mitchell, Wu, Zaldivar, Barnes, Vasserman, Hutchinson, Spitzer, Raji, Gebru — [arXiv:1810.03993](https://arxiv.org/abs/1810.03993)*

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
