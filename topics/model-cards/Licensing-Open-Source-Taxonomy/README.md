# Licensing & "True" Open Source Taxonomy

> Research on how AI model licensing is defined, evaluated, and structured — exploring what makes an AI model "truly" open source, where commercial lines are drawn, and how licensing applies to model components.

---

## 📚 Contents

### [OSI-Compliant vs Open-Weights](./OSI-Compliant-vs-Open-Weights.md)

The formal distinction between "open weights" releases and OSI-defined "open source AI." Covers OSAID 1.0, the four freedoms of AI systems, why Llama is not open source, key debates, and the full taxonomy from closed to fully open.

| Subtopic | Description |
|----------|-------------|
| OSAID 1.0 | The Open Source AI Definition's four freedoms and technical requirements |
| Open Weights Definition | What releasing weights means vs releasing code, data, and provenance |
| Llama Case Study | Detailed analysis of why Meta's Llama licenses fail the Open Source Definition |
| Key Debates | OSI, SFC, FSF, Simon Willison, and industry positions on the terminology war |
| Repos & Organizations | Open-Weights/Definition, LLM360, OLMo, EleutherAI, and the Model Openness Tool |

---

### [Commercial Thresholds](./Commercial-Thresholds.md)

Revenue caps, monthly active user thresholds, and field-of-use restrictions across major "open-weight" model licenses.

| Subtopic | Description |
|----------|-------------|
| Model Thresholds | Meta Llama 700M MAU, Qwen 100M MAU, Mistral $20M/month, Stability AI $1M/year |
| Restriction Matrix | Revenue cap vs MAU cap vs competitor ban vs output training ban vs kill switch |
| Legal Analysis | Stanford/Henderson-Lemley on enforceability; WCR Legal deep-dives; OSI position |
| License Tracking Tools | LicenseRec, Black Duck AI SCA, Hugging Face license tags |

---

### [Sub-File & Component-Level Licensing](./Sub-File-Component-Licensing.md)

How licenses apply to separable model components — base weights vs LoRA adapters vs tokenizers vs code — and the metadata standards used to embed licensing into model file formats.

| Subtopic | Description |
|----------|-------------|
| Component Definitions | Sub-file, component-level, and multi-license repository concepts |
| Hugging Face Adapters | PEFT/LoRA as a modular licensing mechanism |
| Metadata Standards | SafeTensors (`modelspec.license`), GGUF (extensible but no standard key), ONNX |
| Standard Comparison | File-level vs model-level vs per-tensor granularity across formats |
| Derivative Works | RAIL, OpenMDW-1.0, and the legal debate over weight copyrightability |

---

## Overview

AI model licensing is a rapidly evolving field at the intersection of open-source philosophy, commercial strategy, and emerging regulation. Three overlapping tensions dominate:

1. **Definitional** — What legally counts as "open source" for AI? OSI says it's OSAID 1.0 (code + data info + weights under OSI-approved terms). Industry often says "open weights" (weights only). The gap matters because EU AI Act carve-outs depend on the definition.

2. **Commercial** — Many "open" model licenses embed hard commercial thresholds (700M MAU for Llama, $1M/year for Stability AI). These are explicitly rejected by OSI as field-of-use restrictions.

3. **Granular** — Most models are released as monolithic artifacts, but the ecosystem is moving toward modular releases (PEFT adapters, separate tokenizer licenses, multi-license repos). Standards for embedding license metadata into weight files are nascent.

| License Pattern | Weights | Code | Data Info | Commercial Limits | OSI-Compliant |
|----------------|---------|------|-----------|-------------------|---------------|
| **API-Only (GPT-4o, Claude)** | ❌ | ❌ | ❌ | Proprietary | ❌ |
| **Open Weights (Llama, Mistral)** | ✅ | ❌ | ❌/Partial | Yes (MAU/revenue caps) | ❌ |
| **Open Source AI (OLMo, LLM360)** | ✅ | ✅ | ✅ Detailed | No | ✅ |
| **Permissive (Apache 2.0, Gemma 4)** | ✅ | ✅ | ✅/Partial | No | ✅ |
| **Research Only (CC BY-NC)** | ✅ | ✅ | ✅ | Non-commercial only | ❌ |

---

## Key Sources

- [Open Source AI Definition 1.0](https://opensource.org/ai/open-source-ai-definition) — OSI
- [Open-Weights/Definition](https://github.com/Open-Weights/Definition) — Heather Meeker / OSS Capital
- [Model Openness Tool](https://github.com/lfai/model_openness_tool) — LF AI & Data
- [ModelSpec](https://github.com/Stability-AI/ModelSpec) — Stability AI
- [SPDX AI Profile 3.0](https://spdx.dev/learn/areas-of-interest/ai/) — Linux Foundation
- [NTIA Open Model Weights Report](https://www.ntia.gov/programs-and-initiatives/artificial-intelligence/open-model-weights-report) — US Department of Commerce

---

## Related Topics

- [Training Data Lineage & Provenance](../Training-Data-Lineage-Provenance/) — Copyright, fair use, and provenance in training data
- [Automated & Machine-Readable Artifacts](../Automated-Machine-Readable-Artifacts/) — Regulatory passports and drift tracking
- [Safety & Alignment](../../safety/) — Responsible AI, red-teaming, and compliance tooling

---

*Last updated: 2026-04-21*
