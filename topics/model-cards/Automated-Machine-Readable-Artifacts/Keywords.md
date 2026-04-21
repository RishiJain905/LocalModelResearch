# Keywords

> Metadata standards, taxonomy frameworks, automated tagging, and model discovery systems for AI artifacts — covering Schema.org, Croissant, semantic search, and platform-specific approaches.

## Contents

1. [Survey Papers](#survey-papers)
2. [Metadata Standards](#metadata-standards)
3. [Automated Tagging & Extraction](#automated-tagging--extraction)
4. [Model Hubs & Discovery](#model-hubs--discovery)
5. [GitHub Repositories](#github-repositories)
6. [Key Findings](#key-findings)

---

## Survey Papers

### Metadata for ML Models & Datasets: Standards & Harmonization (2024)

> *"A Survey on Metadata for Machine Learning Models and Datasets: Standards, Practices, and Harmonization Challenges"*
> — Gesese, Chen, Zoubia, Limani, Silva, Suryani, Zapilko, Castro, Kutafina, Solanki, Fliegl, Schimmler, Boukhers, Sack
> — FIZ Karlsruhe, KIT, Fraunhofer FOKUS, University of Cologne, ZBW, GESIS, ZBMED, TU Berlin
> — [CEUR-WS Vol. 4065, Paper 05](https://ceur-ws.org/Vol-4065/paper05.pdf)

**Key Findings:**

> *"Platforms (Hugging Face, Zenodo, GitHub, Kaggle, OpenML, MLCommons) vary dramatically in metadata practices — from unstructured README.md to FAIR-aligned structured schemas. No unified framework exists."*

| Finding | Statistic |
|---------|-----------|
| HF datasets lacking language metadata (2023) | ~87% |
| Unique language tag variations on HF | 1,716 variants (`en`, `eng`, `english`, `English`) |

**Best-aligned platforms:** Croissant (MLCommons) and OpenML for FAIR principles. GitHub and Hugging Face rely heavily on community-driven, inconsistent conventions.

---

### BRYT: Automated Keyword Extraction (2024)

> *"BRYT: Automated Keyword Extraction for Open Datasets"*
> — Ahmed, Alexopoulos, Polini
> — *Intelligent Systems with Applications*, July 2024
> — [ScienceDirect](https://www.sciencedirect.com/science/article/pii/S2667305324000954)

**Key Contribution:** Proposes **BRYT** — a hybrid keyword extraction method combining:
- **BERT** — contextual embeddings
- **YAKE** — statistical keyword extraction
- **RAKE** — graph-based extraction
- **TextRank** — ranking-based extraction

Evaluated on European data portals. BRYT consistently outperformed all individual algorithms on keyword match quality.

> *"ChatGPT achieved 157 major matches (best single algorithm) in Science & Technology category, but BRYT hybrid captured complementary signals from all four."*

---

## Metadata Standards

### Croissant: A Metadata Format for ML-Ready Datasets (NeurIPS 2024)

> *"Croissant: A Metadata Format for ML-Ready Datasets"*
> — Akhtar, Benjelloun, Conforti, Foschini, Gijsbers et al. (20+ co-authors)
> — NeurIPS 2024 Datasets & Benchmarks Track
> — [Paper PDF](https://proceedings.neurips.cc/paper_files/paper/2024/file/9547b09b722f2948ff3ddb5d86002bc0-Paper-Datasets_and_Benchmarks_Track.pdf)
> — [GitHub](https://github.com/mlcommons/croissant) ⭐ 812

**Format:** JSON-LD built on **Schema.org** with 4 layers:

1. **Metadata** — name, description, license, URL, citation (via `schema.org/Dataset`)
2. **Resources** — `FileObject` / `FileSet` describing actual data files with SHA256 hashes
3. **Structure** — `RecordSet` + `Field` over heterogeneous formats (CSV, JSON, images, etc.)
4. **ML Semantics** — splits, labels, data types (bounding boxes, categorical data)

**Croissant-RAI extension:** Adds Responsible AI metadata (group fairness, disclaimers, intended use).

**Adoption:**
- Google Dataset Search (crawls and indexes Croissant JSON-LD)
- Kaggle ("Export as Croissant" button)
- OpenML
- Hugging Face (400,000+ datasets)

> *"Croissant provides a common metadata layer that makes datasets machine-readable and interoperable across platforms."*

**Citation:** `10.1145/3650203.3663326`

---

### W3C DCAT Version 3

> [w3.org/TR/vocab-dcat-3](https://www.w3.org/TR/vocab-dcat-3/)

RDF-based vocabulary for describing datasets and data services in catalogs.

- `dcat:Resource` as base class
- `dcat:Dataset`, `dcat:DataService` as specific types
- Extends with provenance (PROV-O), quality info, and data citation properties
- Used in research contexts for mapping platform metadata schemas to a common RDF model

---

### Dublin Core Metadata Initiative

> [nkos.dublincore.org/Metadata-standards.html](https://nkos.dublincore.org/Metadata-standards.html)

Foundational standard used across digital repositories. Referenced by DCAT and other higher-level vocabularies.

---

### MLCommons Model Index

`model_index.yaml` — YAML format for model benchmarking metadata. Scope limited to benchmark results. Precursor to broader Croissant efforts.

---

## Automated Tagging & Extraction

### Huggy Lingo: Automated Language Metadata on Hugging Face (2023)

> *"Huggy Lingo: Using Machine Learning to Improve Language Metadata on the Hugging Face Hub"*
> — Hugging Face Blog, August 2, 2023
> — [huggingface.co/blog/huggy-lingo](https://huggingface.co/blog/huggy-lingo)

**Method:**
- Uses `facebook/fasttext-language-identification` (Meta's No Language Left Behind model, 217 languages)
- Predicts language tags for untagged HF datasets
- Librarian-bot opens PRs to add metadata automatically

**Key Stats:**
- ~50,000 public HF datasets at time of writing
- ~87% lacked language metadata
- 1,716 unique language tag variations existed
- fastText mapped predictions to ISO 639-1 codes for Hub UI compatibility

> *"This is a pioneering example of automated metadata enrichment at scale via ML — a template for how the community can systematically improve discoverability."*

---

### Semantic Search vs. Keyword Search

> *"Beyond Keywords: How Semantic Search is Transforming Discovery"*
> — Merlin Search Technologies
> — [merlin.tech/beyond-keywords/](https://www.merlin.tech/beyond-keywords/)

**Finding:**

> *"Studies show keyword search finds only **20–24% of relevant documents** due to phrasing variability. Semantic search using **embeddings** finds conceptual matches across synonyms."*

Example equivalences semantic search captures:
- "breach of contract" ≈ "failure to perform" ≈ "material default"

**Core mechanism:** Embeddings serve as the preprocessing index for semantic search, much like inverted indexes power keyword search.

---

### PaperSeek: Semantic Search for Academic Papers (2025)

> *"PaperSeek: A Semantic Search Framework for Literature Discovery"*
> — SSRN 2025

Uses semantic embeddings to enhance academic paper discovery beyond keyword matching.

---

## Model Hubs & Discovery

### Hugging Face Hub

Model cards are Markdown files with YAML frontmatter. Fields include:
- `language` — ISO language codes
- `license` — SPDX license identifiers
- `tags` — freeform community tags
- `pipeline_tag` — Hugging Face pipeline identifier
- `datasets` — linked datasets
- `metrics` — evaluation metrics
- `model-index` — benchmarking results

**Tags drive Hub filter UI** — community-contributed, inconsistent.

**Webhook Guide:** *"Setup an Automatic Metadata Quality Review for Models and Datasets"*
— [huggingface.co/docs/hub/en/webhooks-guide-metadata-review](https://huggingface.co/docs/hub/en/webhooks-guide-metadata-review)
— Explains how YAML header blocks power HF's metadata system; how librarian-bots use `ModelCard.load()` / `DatasetCard.load()` for CI/CD-style metadata QA

---

### Ollama

> [docs.ollama.com/api/introduction](https://docs.ollama.com/api/introduction)

- Minimal metadata — models identified by name string only
- No formal tagging schema
- Filtering via API `GET /api/tags` returns bare model listings
- **No keyword/discovery system** — purely name-based

---

### Replicate

> [replicate.com/docs/reference/how-does-replicate-work](https://replicate.com/docs/reference/how-does-replicate-work)

- Public models have metadata: `owner`, `name`, `run_count`, `cover_image_url`, `default_example`
- No user-facing keyword taxonomy
- Discovery via platform browse/search, not structured tags

---

### ModelIndex.org

> [modelindex.org](https://www.modelindex.org/)

Aggregates models from OpenAI, Google, Anthropic, Meta, Mistral.
- Search by provider, modality, capability
- No open keyword taxonomy — provider-specific

---

## GitHub Repositories

| Repository | Description | Stars |
|-----------|-------------|-------|
| [mlcommons/croissant](https://github.com/mlcommons/croissant) | MLCommons Croissant metadata format (JSON-LD on Schema.org). Python lib `mlcroissant`. v1.0.22 | ⭐ 812 |
| [tensorflow/model-card-toolkit](https://github.com/tensorflow/model-card-toolkit) | Google's automated model card generation. Proto → JSON/HTML | Google-maintained |
| [jiarui-liu/AutomatedModelCardGeneration](https://github.com/jiarui-liu/AutomatedModelCardGeneration) | NAACL 2024. CardGen pipeline + CardBench (4.8k model cards) | — |
| [Weixin-Liang/AI-model-card-analysis-HuggingFace](https://github.com/Weixin-Liang/AI-model-card-analysis-HuggingFace) | Analysis of 32K HF model cards. Analyzes `model_info.parquet` | — |
| [bridge2ai/model-card-schema](https://github.com/bridge2ai/model-card-schema) | LinkML rendering of Model Cards schema. URI: `https://w3id.org/linkml/modelcard` | — |
| [rai-project/mlmodelscope](https://github.com/rai-project/mlmodelscope) | Platform for ML model evaluation with standardized taxonomy framework | — |
| [ivylee/model-cards-and-datasheets](https://github.com/ivylee/model-cards-and-datasheets) | Curated collection of model cards, datasheets, DAG cards | — |

---

## Key Findings

1. **Fragmentation is the dominant problem** — No unified metadata schema for AI models. HF uses semi-structured YAML+Markdown; GitHub has none; Kaggle has structured but platform-specific; Zenodo uses DataCite but lacks ML semantics.

2. **Keywords/tags are the weakest metadata layer** — Even on Hugging Face, ~87% of datasets lacked language tags, with 1,716 variant forms coexisting. Keywords are user-supplied, inconsistent, and rarely validated.

3. **Automated enrichment is emerging** — "Huggy Lingo" (fastText + librarian-bot PRs) and "BRYT" (hybrid keyword extraction) represent the leading edge of automated keyword/metadata extraction — but platform-specific and not yet standardized.

4. **Schema.org + Croissant is the closest thing to a standard** — Building on Schema.org's `Dataset` type, Croissant adds ML-specific layers. Adoption by Google Dataset Search, Kaggle, OpenML, and Hugging Face gives it real momentum.

5. **Model cards are human-written and inconsistent** — CardGen (NAACL 2024) and TensorFlow MCT address automation, but generated cards still require validation.

6. **Semantic search for model discovery is underexplored** — Most model hubs rely on keyword filters and hierarchical categories, not embedding-based similarity.

7. **No formal taxonomy framework for AI model types** rivaling, e.g., ImageNet for computer vision. MLModelScope attempted this but hasn't achieved community-wide adoption.

---

## Related Topics

- [Context Windows](../context-windows/) — Attention mechanisms and context
- [Benchmarks](../benchmarks/) — Evaluation frameworks
- [Huggingface](../huggingface/) — Model hub and ecosystem
