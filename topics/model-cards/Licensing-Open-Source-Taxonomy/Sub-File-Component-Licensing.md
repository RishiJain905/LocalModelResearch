# Sub-File & Component-Level Licensing

> **Papers:** ["Current Model Licensing Practices are Dragging Us into a Quagmire of Legal Noncompliance"](https://icml.cc/virtual/2025/poster/40180) · ["Copyright for AI Model Weights"](https://www.marble.onl/posts/model_weight_copyrights.html)  
> **GitHub:** [Stability-AI/ModelSpec](https://github.com/Stability-AI/ModelSpec) · [Open-Future-Foundation/AI-model-licensing](https://github.com/Open-Future-Foundation/AI-model-licensing) · [LF AI & Data Blog](https://lfaidata.foundation/blog/2025/07/22/simplifying-ai-model-licensing-with-openmdw/)  
> **Standards:** [SPDX AI Profile 3.0](https://spdx.dev/learn/areas-of-interest/ai/) · [SafeTensors Metadata](https://huggingface.co/docs/safetensors/index) · [GGUF Spec](https://github.com/ggml-org/ggml/blob/master/docs/gguf.md)

---

## Overview

True "sub-file" licensing — applying different legal terms to individual layers or tensors inside a single model artifact — **does not yet exist as a standard practice**. However, the AI ecosystem has robust mechanisms for **component-level licensing at the artifact level**: base model vs adapter vs tokenizer vs inference code can each carry distinct licenses. This modularity is supported by metadata standards in SafeTensors, GGUF, and ONNX, and by legal frameworks like OpenMDW and RAIL.

> *"Different components of an AI model release require different legal frameworks: OSI-approved open source licenses for code; open data licenses for data components (weights/parameters, datasets); Creative Commons for documentation. Managing multiple licenses creates legal complexity."*
> — [LF AI & Data Blog, July 22, 2025](https://lfaidata.foundation/blog/2025/07/22/simplifying-ai-model-licensing-with-openmdw/)

---

## Definitions

| Concept | Definition | Standardized? |
|---------|-------------|---------------|
| **Sub-file licensing** | Applying different terms to different internal parts of a single artifact (e.g. one license per layer) | ❌ Not standardized |
| **Component-level licensing** | Different licenses on externally separable artifacts: base weights, LoRA adapters, tokenizers, code, docs | ✅ Common practice |
| **Multi-license repos** | A single repository with multiple components under distinct licenses | ✅ Seen in the wild |

> *"Code and network weights are released under different licenses, both are dual licenses depending on applications, research or commercial."*
> — [aboulch/tec_prediction/LICENSE.md](https://github.com/aboulch/tec_prediction/blob/master/LICENSE.md)

---

## Hugging Face Ecosystem: Adapters + Model Cards

### PEFT / LoRA Adapters

Adapter weights (`adapter_model.safetensors`) can be published separately from the frozen base model, often under a different license. Users load them dynamically without redistributing the base.

| Mechanism | How It Supports Modular Licensing | Source |
|----------|----------------------------------|--------|
| **PEFT / LoRA Adapters** | Adapter weights published separately from base model; can carry their own license | [Hugging Face PEFT Docs](https://huggingface.co/docs/transformers/v4.47.1/peft) |
| **Multi-LoRA Serving** | Multiple adapters swapped on a single base model at runtime, logically separating license obligations per adapter | [friendli.ai](https://friendli.ai/blog/how-to-use-hugging-face-multi-lora-adapters) |
| **Model Card Metadata** | `base_model` and `base_model_relation` fields trace provenance, clarifying which license applies to the derived artifact | [Model Cards](https://huggingface.co/docs/hub/model-cards) |

> *"If you're evaluating MiniMax models for a project... the restrictive license primarily targets API providers... This highlights a new pattern: model providers using licenses to control the serving layer, not just the weights themselves."*
> — [Dev.to, "Open-Weight AI Model Licenses Compared"](https://dev.to/alanwest/open-weight-ai-model-licenses-compared-what-minimaxs-controversy-means-for-you-37j)

---

## Metadata Standards for License Embedding

### SafeTensors + `__metadata__`

SafeTensors stores a JSON header with a reserved `__metadata__` key, making it the de-facto vehicle for embedding license metadata directly inside weight files.

| Feature | Detail | Source |
|---------|--------|--------|
| **Header structure** | JSON header containing tensor descriptions + `__metadata__` dictionary | [SafeTensors Docs](https://huggingface.co/docs/safetensors/index) |
| **Standardized metadata keys** | Stability AI's `modelspec.` prefix defines `modelspec.license` as a **CAN-level** (optional) field | [ModelSpec](https://github.com/Stability-AI/ModelSpec) |
| **Library support** | `save_file(weights, filename, metadata=metadata)` callable directly from Python API | [SafeTensors Docs](https://huggingface.co/docs/safetensors/index) |

**Key metadata key:**
```
modelspec.license → "CC-BY-SA-4.0", "CreativeML Open RAIL-M", etc.
```
Source: [Stability-AI/ModelSpec](https://github.com/Stability-AI/ModelSpec)

### GGUF Metadata

GGUF (GPT-Generated Unified Format) supports extensive key-value metadata but **currently has no standardized `license` key**.

| Feature | Detail | Standardized? |
|---------|--------|---------------|
| **Key-value map** | Extensible metadata for hyperparameters, architecture, tokenizer info | ✅ |
|`general.license` key | Not currently defined in GGUF spec | ❌ |
| **Naming convention** | Filename components can encode metadata indirectly (e.g., LoRA type) | ⚠️ Structural only |

Source: [GGUF Spec](https://github.com/ggml-org/ggml/blob/master/docs/gguf.md)

### ONNX `metadata_props`

ONNX supports arbitrary string key-value metadata, including custom license keys.

| Feature | Detail | Standardized? |
|---------|--------|---------------|
| **`metadata_props.add()`** | Python API allows custom key-value metadata injection | ✅ Mechanism exists |
| `license` key | No formal standard defined in core ONNX spec | ❌ |
| **Custom usage** | Proven in projects like QGIS Deepness for model provenance | ✅ Achievable |

Sources: [ONNX Python API](https://onnxruntime.ai/docs/api/python/auto_examples/plot_metadata.html) · [MetadataProps](https://onnx.ai/onnx/repo-docs/MetadataProps.html)

---

## Comparison: Metadata Embed Standards

| Format | License Embedding Standard | Granularity | Tooling |
|--------|--------------------------|-------------|---------|
| **SafeTensors** | `__metadata__` dict; `modelspec.license` key (recommended) | **File-level** (one license per `.safetensors`) | Native via `save_file()` |
| **GGUF** | Extensible key-value map; **no standardized license key** | File-level metadata; no per-tensor license | Filename convention only |
| **ONNX** | `metadata_props` allows arbitrary keys; no formal standard | Model/graph-level | Python API |
| **Hugging Face Hub** | YAML model card front-matter; `license:` field in UI | **Repository-level** | Web UI + filtering |

---

## Legal Frameworks for Derivative Works

| Framework | Key Mechanism for Derivatives | Source |
|----------|------------------------------|--------|
| **RAIL / OpenRAIL** | Use restrictions in the original license must be included in downstream re-distributions | [The Turing Way](https://book.the-turing-way.org/reproducible-research/licensing/licensing-ml/) |
| **OpenMDW-1.0** | Grants broad royalty-free permission; outputs not subject to license's attribution | [LF AI & Data](https://lfaidata.foundation/blog/2025/07/22/simplifying-ai-model-licensing-with-openmdw/) |
| **BigScience BLOOM RAIL** | Binds downstream fine-tunes to inherited use restrictions | [RAIL FAQ](https://www.licenses.ai/faq-2) |

**Critical debate:** Are model weights:
1. A **derivative work** of training data (dataset licenses propagate)?
2. **Transformative / not copyrightable** (downstream licensing uncertain)?

> *"The two key arguments I've seen are (1) the weights are a derivative work of the training data and/or (2) they are computer generated so cannot be copyrighted."*
> — [Andrew Marble](https://www.marble.onl/posts/model_weight_copyrights.html)

> *"If you were licensing your model under a permissive open source license, you should be aware that downstream users... will not be able to re-distribute the model or derivatives of it under an open source license, as they will have to place the use restrictions of the initial license."*
> — [RAIL FAQ](https://www.licenses.ai/faq-2)

---

## Community & Academic Discussion

| Source | Key Insight | URL |
|--------|-------------|-----|
| **Open-Future-Foundation** | Analysis of 120k+ Hugging Face repos finds ~70% have no license or use `unknown`/`other` tags; practical complexity at repo level | [GitHub](https://github.com/Open-Future-Foundation/AI-model-licensing) |
| **ICML 2025 Oral** | "Current Model Licensing Practices are Dragging Us into a Quagmire of Legal Noncompliance" — academic recognition that current licensing is broken | [ICML Poster](https://icml.cc/virtual/2025/poster/40180) |
| **Marble.onl** | Argues weights should be copyrightable to prevent free-riding by proprietary systems | [Blog](https://www.marble.onl/posts/model_weight_copyrights.html) |
| **SPDX AI Profile** | SPDX 3.0 now supports AI SBOMs, enabling traceability of component licenses across the model supply chain | [SPDX](https://spdx.dev/learn/areas-of-interest/ai/) |
| **r/StableDiffusion** | Community sentiment: "ideal if SafeTensors itself stored base model, trigger words, and even a license description inside the file header" | [Reddit](https://www.reddit.com/r/StableDiffusion/comments/1459r8e/should_safetensors_files_have_a_model_card/) |

---

## Research Gaps

| Gap | Description | Status |
|-----|-------------|--------|
| **Per-tensor / per-layer licensing** | Different terms for individual layers inside a single file | Not yet a standard |
| **GGUF license key standardization** | Standardized key for license metadata in GGUF files | Spec does not yet define `general.license` |
| **ONNX license metadata convention** | Community convention for `license` key in ONNX metadata | No formal standard exists |
| **Automated compliance checking** | Tools to verify multi-license repo compliance at component level | LicenseRec is academic; enterprise tools exist but are costly |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Current reality** | Modular/component-level licensing (base vs adapter vs code vs docs) is the pragmatic norm. True sub-file (per-tensor) licensing does not exist. |
| **Best format for metadata** | SafeTensors via `modelspec.license` in `__metadata__` is the closest standard. |
| **Best practice** | Use multi-license repos with clear per-component README/LICENSE files and Hugging Face model card metadata. |
| **Emerging standard** | SPDX AI SBOM 3.0 for supply-chain license tracking; OpenMDW-1.0 for simplified "Model Materials" licensing. |
| **Open questions** | Are model weights copyrightable? If not, how enforceable are downstream license restrictions? |

---

*Last updated: 2026-04-21*
