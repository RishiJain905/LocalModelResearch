# Dataset Composition Analysis

> Frameworks, tools, and standards for documenting what AI training datasets are made of — covering provenance tracking, documentation templates, attribution methods, and transparency practices.

## Contents

1. [Documentation Frameworks](#documentation-frameworks)
2. [Key Papers](#key-papers)
3. [Provenance Tools & Repositories](#provenance-tools--repositories)
4. [Attribution & Watermarking](#attribution--watermarking)
5. [Key Findings](#key-findings)

---

## Documentation Frameworks

### Datasheets for Datasets (Gebru et al., 2021)

> *"Datasheets for Datasets"*
> — Timnit Gebru, Jamie Morgenstern, Brian Goeltz, et al.
> — Communications of the ACM, December 2021 (first posted arXiv:1803.09010, 2018)
> — [arXiv:1803.09010](https://arxiv.org/abs/1803.09010)

Standardized documentation template with **56 questions** across 8 sections:

| Section | Key Questions |
|---------|--------------|
| **Motivation** | Why was the dataset created? Who funded it? |
| **Composition** | What does the dataset contain? How is it structured? |
| **Collection Process** | How was data collected? Who are the annotators? |
| **Preprocessing/Cleaning/Labeling** | What transformations were applied? |
| **Uses** | What is the dataset intended for? What is it NOT for? |
| **Distribution** | How is the dataset distributed? Any restrictions? |
| **Maintenance** | Who maintains it? Will it be updated? |
| **Revisions** | What changes have been made between versions? |

> *"Analogous to datasheets for electronic components, datasheets for datasets document their provenance, composition, collection process, label annotation process, intended uses, maintenance, and other information."*

Directly referenced in EU AI Act practical guidance.

---

### Dataset Nutrition Labels (Holland et al., 2018)

> *"The Dataset Nutrition Label"*
> — Sarah Holland, Nima H. Hosny, et al.
> — arXiv:1805.03677 (2018)
> — [Data Nutrition Project](https://datanutrition.org/)

Diagnostic framework providing standardized "ingredient labels" for datasets before model development.

**Multi-level disclosure:**
- **Dataset-level** — general statistics (size, modalities, diversity)
- **Subgroup-level** — breakdowns by demographic/functional groups
- **Instance-level** — individual example inspection

> *"The Nutrition Label provides a standardized disclosure framework to increase transparency and accountability in dataset creation."*

---

### Data Cards (Pushkarna et al., FAccT 2022)

> *"Data Cards: Purposeful and Transparent Dataset Documentation"*
> — Mahima Pushkarna, Andrew Zaldivar, Oddur Kjartansson
> — FAccT 2022
> — [Google Research](https://arxiv.org/abs/2204.01075)

Structured summaries with **telescope/periscope/microscope** question framework:
- **Telescope** — Who produced this data? What is the broader context?
- **Periscope** — How was data collected, annotated, and processed?
- **Microscope** — What does the data actually contain at the instance level?

Developed at Google over 24 months across 22 datasets. Used in Google Cloud, TensorFlow, and Kaggle.

> *"Data Cards bridge the gap between dataset creators and end users by making documentation a collaborative, multi-stakeholder process."*

---

### Data Statements for NLP (Bender & Friedman, 2018/2023)

> *"Data Statements for NLP"*
> — Emily M. Bender, Batya Friedman, Angelina McMillan-Major
> — [ACL 2018](https://aclanthology.org/Q18-1041/) | Version 2: ACM JR. Responsible Computing (2023)

Framework for documenting speaker demographics, annotator characteristics, and language dataset context.

**Key principle — The Bender Rule:**
> *"Always identify the language and dialect or accent of the people in the dataset, not the language of the dataset."*

Structured fields:
- Language and dialect identification
- Speaker demographics (age, gender, geographic origin)
- Annotator demographics and training
- Text genres and registers

---

### CLeAR Documentation Framework (2024)

> *"The CLeAR Documentation Framework for AI Transparency"*
> — Shorenstein Center, Harvard / 15 researchers
> — Partners: Data Nutrition Project, Harvard, MIT, NYU, Microsoft, IBM, Hugging Face
> — [Microsoft Research](https://www.microsoft.com/en-us/research/publication/the-clear-documentation-framework-for-ai-transparency-recommendations-for-practitioners-context-for-policymakers/)

**CLeAR = Comparable, Legible, Actionable, Robust**

Four documentation principles mandating transparency throughout the AI lifecycle:
- Integrates risk and impact assessments
- Designed for both practitioners and policymakers

> *"Documentation should be mandatory for AI systems — in creation, usage, and sale of AI artifacts."*

---

### Data Provenance Cards (DPI, 2023)

> *"The Data Provenance Initiative"*
> — Longpre et al. — CMU / MIT / Harvard
> — [Nature, 2024](https://www.nature.com/articles/s42256-024-00878-8) | [dataprovenance.org](https://www.dataprovenance.org)

Auto-generated "bibliography for data" combining:
- Attribution — who contributed what
- Provenance — how data was transformed
- Composition — what's in the dataset

> *"Data Provenance Cards enable concatenatable documentation — when datasets are merged, their provenance cards can be merged too."*

---

## Key Papers

### Data Provenance Initiative Audit (Nature, 2024)

> *"The Data Provenance Initiative: A Large Scale Audit of Dataset Licensing & Attribution in AI"*
> — Longpre et al. — CMU, MIT, Harvard
> — [Nature 2024](https://www.nature.com/articles/s42256-024-00878-8)

Audit of **1,858 fine-tuning datasets** across 44 collections. Key findings:

| Finding | Statistic |
|---------|-----------|
| Datasets with missing/unspecified licenses | 72%+ |
| Licenses mislabeled on Hugging Face | 66% |
| Commercial datasets with CC license | 15.7% |
| Commercial datasets with NC restrictions | Higher than academic |

> *"Platform-reported licenses often don't match actual licensing intent — frequently more permissive."*

---

### Data Curation Evaluation Framework (2024)

> *"Machine Learning Data Practices through a Data Curation Lens: An Evaluation Framework"*
> — Ariful Prilient et al.
> — arXiv:2405.02703v1

Rubric evaluating **19 dimensions across 5 groups** applied to 25 NeurIPS datasets:
- Scope (size, modality, domain)
- Ethicality (consent, privacy, bias)
- ML Pipeline (collection, annotation, processing)
- Data Quality (completeness, consistency, accuracy)
- FAIR (findability, accessibility, interoperability, reusability)

---

### Completeness of Dataset Documentation (2025)

> *"Completeness of Datasets Documentation on ML/AI repositories"*
> — arXiv:2503.13463 | EPIA 2023

Empirical investigation of documentation completeness in ML/AI repositories. Finds systematic gaps in:
- Ethical considerations
- Maintenance plans
- Known use cases and limitations

---

### Training Data Attribution for LLMs (EMNLP 2024)

> *"Enhancing Training Data Attribution for Large Language Models with Fitting Error Consideration"*
> — Kangxi Wu et al.
> — EMNLP 2024

Addresses fitting error issues in training data attribution (TDA) for LLMs using influence functions. Extends Koh & Liang (2020) influence function approach.

---

## Provenance Tools & Repositories

### Data Provenance Collection

> [github.com/Data-Provenance-Initiative/Data-Provenance-Collection](https://github.com/Data-Provenance-Initiative/Data-Provenance-Collection)

Multi-disciplinary audit of 44 data collections, 1,800+ fine-tuning datasets. Generates Data Provenance Cards (structured attribution docs).

---

### Data Provenance Tracker

> [github.com/0xReLogic/Data-Provenance-Tracker](https://github.com/0xReLogic/Data-Provenance-Tracker)

Go-based tool for tracking data lineage and provenance in pipelines. Records transformations (filtering, joining, aggregation). Go API + CLI.

---

### IBM ProvLake

> [github.com/IBM/multi-data-lineage-capture-py](https://github.com/IBM/multi-data-lineage-capture-py)

Captures provenance from multiple workflows via code instrumentation. Tracks data transformations across pipelines.

---

### DataDetective: Dataset Watermarking

> *"DataDetective: Easy and Efficient Data Watermarking for Leakage Identification"*
> — IOS Press FAIA 2024
> — [GitHub](https://github.com/NoaWegerhoff/data-detective)

Embeds unique watermark signatures per agent/source:
- **Perfect leaker identification** with only 1% watermarked data
- Black-box approach (no model access required)
- Backdoor-based methodology

> *"DataDetective allows dataset creators to embed agent-specific backdoor signatures — if the dataset leaks, the watermark uniquely identifies the source."*

---

### Data Provenance Explorer

> [dataprovenance.org](https://www.dataprovenance.org)

Interactive tool for filtering and analyzing LLM training datasets. Allows searching across the DPI audit dataset for license, attribution, and composition information.

---

## Attribution & Watermarking

| Method | Approach | Reference |
|--------|----------|-----------|
| **Influence Functions** | Trace predictions back to training samples via gradient-based influence | Koh & Liang (2020) |
| **Dataset Watermarking** | Embed unique signatures per source agent for leak identification | DataDetective (2024) |
| **Symbolic Attribution** | Auto-generate structured "bibliography for data" | Data Provenance Cards (DPI) |
| **Model Inversion Attacks** | Reconstruct training data from trained models | Survey: arXiv:2411.10023v1 |

---

## Key Findings

1. **Documentation debt is systemic** — 72%+ of datasets missing licenses; 66% mislabeled even when present

2. **Frameworks are converging** — Datasheets, Data Cards, Nutrition Labels, and Data Statements share common core elements (provenance, composition, collection process, intended uses, limitations)

3. **Scale complicates attribution** — LLM training data is rarely documented concurrently; most documentation is post-hoc

4. **Multi-stakeholder gap** — Data Cards research found "Producers often assume 'users' have high domain expertise" — creating documentation that only experts can use

5. **Regulatory momentum** — EU AI Act, US Executive Order, and G7 Hiroshima Code of Conduct all mandate training data transparency

---

## Related Topics

- [Copyright & Public Domain Disclosures](./Copyright-Public-Domain-Disclosures.md) — Licensing and rights analysis
- [Data Contamination Reporting](./Data-Contamination-Reporting.md) — Dataset integrity and test set leakage
- [Datasets](../datasets/) — Dataset sources and management
