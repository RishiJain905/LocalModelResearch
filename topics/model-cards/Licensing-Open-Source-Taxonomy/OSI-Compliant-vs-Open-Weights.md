# OSI-Compliant vs Open-Weights

> **Papers:** ["Beyond the Binary: A nuanced path for open-weight advanced AI"](https://arxiv.org/abs/2602.19682) · ["Opening the Scope of Openness in AI"](https://arxiv.org/abs/2505.06464) · ["LLM360: Towards Fully Transparent Open-Source LLMs"](https://arxiv.org/abs/2312.06550)  
> **GitHub:** [Open-Weights/Definition](https://github.com/Open-Weights/Definition) · [allenai/awesome-open-source-lms](https://github.com/allenai/awesome-open-source-lms) · [LLM360](https://www.llm360.ai/)  
> **Regulatory:** [OSI OSAID 1.0](https://opensource.org/ai/open-source-ai-definition) · [NTIA Open Model Weights Report](https://www.ntia.gov/programs-and-initiatives/artificial-intelligence/open-model-weights-report/policy-approaches-recommendations/policy-approaches)

---

## Overview

The terms "open source" and "open weights" are frequently conflated in AI model releases. The **Open Source Initiative (OSI)** maintains that releasing model weights alone does **not** constitute open source. This distinction became codified in October 2024 with the release of the **Open Source AI Definition (OSAID) 1.0**.

> *"An Open Source AI is an AI system made available under terms and in a way that grant the freedoms to: Use for any purpose, Study how it works, Modify as needed, and Share — for others to use with or without modifications, for any purpose."*
> — [Open Source AI Definition 1.0](https://opensource.org/ai/open-source-ai-definition)

---

## OSAID 1.0: The Four Freedoms

OSAID extends the four fundamental freedoms of open source software to AI systems:

| Freedom | Description | Requirement |
|---------|-------------|-------------|
| **Use** | Use the system for any purpose | No field-of-use restrictions |
| **Study** | Inspect how the system works | Full source code must be available |
| **Modify** | Change the system for any purpose | Preferred form for modifications |
| **Share** | Distribute for others to use | Redistribution under same terms |

To satisfy the "preferred form to make modifications," three elements must be made available under OSI-approved terms:

1. **Data Information** — *"sufficiently detailed information about the data used to train the system so that a person can build a substantially equivalent system"*
2. **Code** — *"the complete source code used to train and run the system"*
3. **Parameters** — *"model weights and configuration settings"*

> *"The Open Source AI Definition does not require a specific legal mechanism for assuring that the model parameters are freely available to all. We expect this will become clearer over time, once the legal system has had more opportunity to address Open Source AI systems."*
> — [OSAID FAQ](https://opensource.org/ai/faq)

---

## Open Weights: What It Actually Means

> *"Open Weights refer to the final weights and biases of a trained neural network. These values, once locked in, determine how the model interprets input data and generates outputs."*
> — [OSI: Open Weights: not quite what you've been told](https://opensource.org/ai/open-weights)

Open weights releases (like Meta Llama, Mistral, etc.) provide the trained model parameters but typically **withhold** training code, training data, and full provenance. This is fundamentally different from OSAID "Open Source AI."

| Feature | Open Weights Release | Open Source AI (OSAID) |
|---------|---------------------|----------------------|
| Weights & Biases | ✅ Released | ✅ Released |
| Training Code | ❌ Not Shared | ✅ Fully Shared |
| Intermediate Checkpoints | ❌ Withheld | ✅ Nice to have |
| Training Dataset | ❌ Not/Partially Disclosed | ✅ Disclosed* |
| Training Data Composition | ⚠️ Partially/Not Disclosed | ✅ Fully Disclosed |

> *"When legally allowed. See OSAID FAQ for caveats on training data that cannot be freely shared."*

---

## Why Llama Is Not Open Source

Meta's Llama models are the most prominent example of an "open weights" release that is frequently mislabeled as open source.

> *"Meta's license for the LLaMa models and code does not meet this standard; specifically, it puts restrictions on commercial use for some users and also restricts the use of the model and software for certain purposes (the Acceptable Use Policy). An Open Source license ensures that developers and users are able to decide for themselves how and where to use the technology without the need to engage with another party."*
> — [OSI: Meta's LLaMa License is not Open Source](https://opensource.org/blog/metas-llama-2-license-is-not-open-source)

On Llama 3.x specifically:

> *"Fails at freedom 0, the freedom to use the model for any purpose. Fails at the Open Source Definition point 5, discriminates against users. Fails at the Open Source Definition point 6, restricts fields of endeavour."*
> — [OSI: Meta's LLaMa license is still not Open Source](https://opensource.org/blog/metas-llama-license-is-still-not-open-source)

> *"The FSF Blog — The Llama 3.1 Community License agreement... is not a free software license and you should not use it if you care about software freedom."*
> — [FSF Blog](https://www.fsf.org/blogs/licensing/llama-3-1-community-license-is-not-a-free-software-license)

---

## Key Debates & Blogs

| Source | Position / Quote | URL |
|---|---|---|
| **OSI (Stefano Maffulli)** | *"'Open Source' means software under a license with specific characteristics, defined by the Open Source Definition."* | [TechCrunch, 2024](https://techcrunch.com/2024/10/28/we-finally-have-an-official-definition-for-open-source-ai/) |
| **Software Freedom Conservancy** | *"The OSAID fails to require reproducibility by the public of the scientific process of building these systems... OSI has endorsed demonstrably bad LLM-backed generative AI systems as 'open source' by definition!"* | [SFC Blog, Oct 2024](https://sfconservancy.org/blog/2024/oct/31/open-source-ai-definition-osaid-erodes-foss/) |
| **Simon Willison** | *"The only janky license among them is Kimi K2, which uses a non-OSI-compliant modified MIT. Qwen's models are all Apache 2 and Z.ai's are MIT."* | [simonwillison.net, July 2025](https://simonwillison.net/2025/Jul/30/chinese-models/) |
| **Simon Willison** | *"Is it possible that Meta are deliberately misusing the term 'open source' because upcoming legislation has carve-outs for open source models?"* | [X/Twitter](https://x.com/simonw/status/1816131242452213993) |
| **Open Future** | *"It would have been preferable for the OSI to choose a name for the definition that makes it clear that the level of openness required to meet the definition corresponds to what many observers call open weights models – and reserve the term 'open source' for AI systems that include open data."* | [openfuture.eu](https://openfuture.eu/blog/the-open-source-ai-definition-is-a-step-forward-in-defining-openness-in-ai/) |
| **Duke / Gabriel Toscano** | *"Preliminary exploratory analysis of the Hugging Face data points to differing practices in how developers signal openness in AI."* | [The State of "Open" Source AI](https://sites.duke.edu/gabrieltoscano/2026/02/05/the-state-of-open-source-ai/) |

---

## Taxonomy: From Closed to Fully Open

| Category | Weights | Code | Data Info | License | Examples |
|----------|---------|------|-----------|---------|----------|
| **Closed / API-Only** | ❌ | ❌ | ❌ | Proprietary | GPT-4o, Claude 3.5 Sonnet |
| **Open Weights (NOT OSI)** | ✅ | ❌ | ❌/Partial | Custom/Restricted | Meta Llama 3.x, Mistral (some), Kimi K2 |
| **Open Source AI (OSAID)** | ✅ | ✅ | ✅ Detailed | OSI-Approved | OLMo, LLM360 Amber, GPT-NeoX, Pythia |
| **Open Weights (OSS Capital defn)** | ✅ | ❌ | ❌ Not required | Open Weights Defn | Various industry releases |

> *"The critical distinction is that Open Weights releases (like Meta Llama, most Mistral models) provide the model parameters but withhold the training code and full training data provenance, making them not OSI-compliant. True Open Source AI per OSAID 1.0 requires the full 'preferred form to make modifications.'"*

---

## Key Repos & Organizations

| Repository / Org | Description | URL |
|---|---|---|
| **Open-Weights/Definition** | Community-driven definition of Open Weights by Heather Meeker / OSS Capital. Explicitly does NOT require training data disclosure. | [github.com/Open-Weights/Definition](https://github.com/Open-Weights/Definition) |
| **Model Openness Tool (MOT)** | LF AI & Data's practical scoring tool for model openness based on the MOF framework. | [github.com/lfai/model_openness_tool](https://github.com/lfai/model_openness_tool) |
| **allenai/awesome-open-source-lms** | Curated list of truly open-source language models (training code + data + weights). OLMo is the flagship. | [github.com/allenai/awesome-open-source-lms](https://github.com/allenai/awesome-open-source-lms) |
| **LLM360** | Initiative releasing all training code, data, model checkpoints, and intermediate metrics. | [llm360.ai](https://www.llm360.ai/) |
| **EleutherAI** | Open science organization releasing fully open models (GPT-NeoX, Pythia). | [eleuther.ai](https://www.eleuther.ai/) |

---

## Papers & Reports

| Paper / Report | Key Finding | URL |
|---|---|---|
| **"Beyond the Binary: A nuanced path for open-weight advanced AI"** (arXiv:2602.19682) | Moves beyond binary open/closed framing to assess the current landscape of open-weight advanced AI. | [arXiv](https://arxiv.org/abs/2602.19682) |
| **"Opening the Scope of Openness in AI"** (arXiv:2505.06464) | *"The practices and benefits of open source software are not fully transferable to AI."* Develops a taxonomy of 98 openness concepts. | [arXiv](https://arxiv.org/abs/2505.06464) |
| **"LLM360: Towards Fully Transparent Open-Source LLMs"** (arXiv:2312.06550) | Framework for fully transparent LLMs including all checkpoints, data, metrics, and code. | [arXiv](https://arxiv.org/abs/2312.06550) |
| **Open-Weights Definition (Heather Meeker, 2023)** | Formalizes Open Weights as distinct from Open Source AI. | [heathermeeker.com](https://heathermeeker.com/2023/06/08/toward-an-open-weights-definition/) |
| **NTIA Open Model Weights Report** | *"Open-weights models allow more researchers than just the small number at industry labs to investigate how to improve model safety."* | [NTIA](https://www.ntia.gov/programs-and-initiatives/artificial-intelligence/open-model-weights-report/policy-approaches-recommendations/policy-approaches) |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Core distinction** | "Open Weights" = weights available, code/data withheld. "Open Source AI" (OSAID) = weights + code + data info, under OSI-approved terms. |
| **Why it matters** | EU AI Act and other regulations have carve-outs for "open source" models; mislabeling can affect compliance and liability. |
| **Best for researchers** | OSAID-compliant models (OLMo, LLM360) for reproducibility and study. |
| **Best for practitioners** | Open-weights models (Llama, Mistral) for production inference with licensing caveats. |
| **Legal risk** | Custom licenses (Llama, Qwen) have usage thresholds, competitor bans, and output-training restrictions not present in OSI definitions. |

---

*Last updated: 2026-04-21*
