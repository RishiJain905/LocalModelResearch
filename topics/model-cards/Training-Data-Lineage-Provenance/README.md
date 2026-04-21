# Training Data Lineage & Provenance

> Research on how AI training data is documented, traced, licensed, and audited — covering dataset composition analysis, copyright disclosures, and contamination reporting.

## 📚 Topics

| Topic | Description |
|-------|-------------|
| [Dataset Composition Analysis](./Dataset-Composition-Analysis.md) | Dataset documentation frameworks, provenance tracking, and composition transparency |
| [Copyright & Public Domain Disclosures](./Copyright-Public-Domain-Disclosures.md) | Training data licensing, copyright law, fair use, and rights disclosures |
| [Data Contamination Reporting](./Data-Contamination-Reporting.md) | Benchmark contamination detection, test set leakage, and model evaluation integrity |

---

## Overview

Training data lineage and provenance addresses the fundamental question: *where did the data come from, how was it processed, and what rights exist around it?*

Key events in this space:

### The Data Provenance Initiative (2023–2024)

> *"The Data Provenance Initiative: A Large Scale Audit of Dataset Licensing & Attribution in AI"*
> — Longpre et al. — CMU / MIT / Harvard
> — [Nature, 2024](https://www.nature.com/articles/s42256-024-00878-8) | [dataprovenance.org](https://www.dataprovenance.org)

Audit of **1,858 fine-tuning datasets** across 44 collections. Found:
- **72%+ missing licenses** on crowdsourced platforms
- **Up to 66% mislabeled** (often as more permissive than intended)
- Created Data Provenance Cards — structured attribution docs similar to a bibliography for data

### EU AI Act Requirements

> *"GPAI model providers must put in place a policy to comply with EU copyright law and publish a summary of the training content."* — Article 53(1)(d)

---

## Key Findings

1. **Documentation debt**: The majority of AI training datasets lack adequate provenance documentation — missing licenses, unclear sources, and undocumented filtering steps

2. **Copyright as a scaling bottleneck**: The emerging ~$2.5B licensing market for training data signals that free-and-clear training data is becoming a competitive advantage

3. **Contamination is widespread but uneven**: Contamination rates range from 1–45% across benchmarks; larger models gain more advantage from contaminated data

4. **Transparency is regulatory**: EU AI Act Article 53 now mandates training content summaries for GPAI models; US Copyright Office Report (2025) favors training-phase fair use but requires documentation

5. **Documentation frameworks converge**: Datasheets, Data Cards, Nutrition Labels, and Data Statements all share similar core elements (provenance, composition, collection process, intended uses, limitations)

---

## Related Topics

- [Regulatory Passports](../Automated-Machine-Readable-Artifacts/Regulatory-Passports.md) — Compliance frameworks and documentation standards
- [Safety](../safety/) — Responsible AI and data ethics
- [Datasets](../datasets/) — Dataset sources and management
