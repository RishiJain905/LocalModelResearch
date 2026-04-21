# Copyright & Public Domain Disclosures

> Legal and regulatory landscape of AI training data rights — covering copyright law, fair use, licensing frameworks, court cases, and compliance requirements for disclosing training data provenance.

## Contents

1. [Key Legal References](#key-legal-references)
2. [Court Cases & Regulatory Developments](#court-cases--regulatory-developments)
3. [Licensing Frameworks](#licensing-frameworks)
4. [GitHub Repositories & Tools](#github-repositories--tools)
5. [Organization Positions](#organization-positions)
6. [Key Findings](#key-findings)

---

## Key Legal References

### U.S. Copyright Office: "Copyright and Artificial Intelligence" — Part 3 (May 2025)

> *"Copyright and Artificial Intelligence: Generative AI Training"*
> — U.S. Copyright Office
> — [copyright.gov/ai/](https://www.copyright.gov/ai/)

**Key findings:**

| Finding | Detail |
|---------|--------|
| Training phase | "Will often be transformative" (favors fair use) |
| Expressive copying | Copying entire works for training is "practically necessary for some forms of training" |
| Market harm | Must be analyzed broadly — nascent licensing market for AI training now exists (~$2.5B) |
| Training vs. output | Distinguishes training data (input) from generated content (output) |

> *"The Copyright Office's report acknowledges that AI training 'will often be transformative' but finds that copying entire works 'may be practically necessary for some forms of training.'"*

---

### Generative AI Training and Copyright Law (arXiv:2502.15858)

> *"Generative AI Training and Copyright Law"*
> — [arXiv:2502.15858](https://arxiv.org/html/2502.15858v1)

Argues that fair use/TDM exceptions may **NOT** apply to generative AI training because:
- Generative AI can reproduce **expressive elements** of copyrighted works
- Training data memorization leads to copyright issues **independent** of fair use
- Advocates for **transparent documentation** of training data sources

---

### Does Training AI Violate Copyright Law? (Berkeley Technology Law Journal)

> *"Does Training AI Violate Copyright Law?"*
> — Berkeley Technology Law Journal
> — [btlj.org](https://btlj.org/wp/cont/uploads/2023/02/0003-36-4Quang.pdf)

Covers copyright bias concerns:
> *"Public domain works pre-1923 reflect historical biases — CC-licensed content (Wikipedia) has gender bias issues in editor demographics."*

---

### Copyright and AI Training Data — Transparency to the Rescue? (JIPLP, 2025)

> *"Copyright and AI Training Data—Transparency to the Rescue?"*
> — Journal of Intellectual Property Law & Practice, 2025
> — [Oxford Academic](https://academic.oup.com/jiplp/article/20/3/182/7922541)

Proposes **legal requirement** for AI developers to be transparent about training dataset contents — enabling rightsholders to enforce their rights.

> *"Transparency in training data is not just good practice — it may become a legal requirement that enables the entire rights ecosystem to function."*

---

## Court Cases & Regulatory Developments

### Major U.S. Cases

| Case | Jurisdiction | Status | Key Issue |
|------|-------------|--------|-----------|
| **Andersen v. Stability AI** | N.D. Cal | Direct infringement claims alive; DMCA claims dismissed | Training on LAION dataset; court permitted "similar outputs" claim |
| **NYT v. OpenAI** | S.D.N.Y. | Motion to dismiss denied; MDL consolidation | Training on NYT content; preservation order for ChatGPT logs |
| **Kadrey v. Meta** | N.D. Cal | Ongoing; discovery disputes | Books3 dataset used for LLaMA training |
| **Thomson Reuters v. Ross Intelligence** | D. Del | **First decision finding AI training NOT fair use** (Feb 2025) | Legal search tool trained on rewritten Thomson Reuters annotations |
| **Bartz v. Anthropic** | N.D. Cal | **$1.5B settlement** (largest copyright settlement ever) | Training on pirated books; "library for any purpose" not fair use |
| **Concord Music v. Anthropic** | N.D. Cal | Discovery ongoing | Music lyrics training for Claude |

**Thomson Reuters v. Ross Intelligence — Key Quote:**
> *"The court found that Ross's use was NOT fair use — rewriting Thomson's headnotes to create legal annotations for a competing product was not transformative enough to justify copying."*

**Bartz v. Anthropic — Key Quote:**
> *"$1.5B settlement for training on pirated books — 'library for any purpose' is not a fair use defense."*

---

### EU Developments

**EU AI Act (Article 53) — GPAI Requirements:**
> *"GPAI model providers must put in place a policy to comply with EU copyright law and publish a summary of the training content."*

Specific obligations:
- **Opt-out compliance**: Respect rightsholder opt-outs from TDM exception
- **Transparency reporting**: Summary of training content for models >10^25 FLOPs
- **Downstream liability**: License obligations may flow to downstream deployers

**UK: Getty Images v Stability AI**
- Court rejected secondary copyright claim
- Territoriality matters — overseas-trained models may not be subject to UK copyright

---

## Licensing Frameworks

### Dataset License Landscape (DPI Audit, 2024)

| License Type | Share (approx.) | Key Restriction |
|-------------|----------------|-----------------|
| **CC BY-SA 4.0** | 15.7% | Attribution + Share-alike |
| **CC BY-NC 4.0** | ~10% | Non-commercial only |
| **CC0 / Public Domain** | ~8% | No restrictions |
| **Open AI Terms** | 12.3% | Commercial use often prohibited |
| **Unspecified** | 70%+ | Unknown/missing |

> *"Commercial datasets are systematically less diverse than NC/Academic-only datasets — licensing restrictions limit what can be included."*

---

### RAIL Licenses (Responsible AI Licenses)

> [github.com/Responsible-Adversarial-Collaborations/RAIL](https://github.com/Responsible-Adversarial-Collaborations/RAIL)

Creates taxonomy for data use rights in AI/ML contexts:
- **RAIL-D** — Dataset-specific licenses for AI training
- Defines acceptable use, attribution requirements, and share-alike terms

---

### Common Pile v0.1 (EleutherAI, 2025)

> [eleutherai/news](https://eleutherai.ai)

8TB dataset of **exclusively public domain and openly licensed text** for LLM training.

> *"Common Pile is a dataset designed to avoid copyright issues entirely — only public domain and permissively licensed works are included."*

---

### Open Data Commons License

Addresses **mixed datasets** with multiple rights types:
- Various licenses can coexist in one dataset
- Requires attribution to each component
- Share-alike may apply to portions

---

## GitHub Repositories & Tools

### OpenDataology (LF AI & Data Foundation)

> [github.com/OpenDataology/OpenDataology](https://github.com/OpenDataology/OpenDataology)

Best practices for AI dataset metadata and license compliance:
- Workflow for identifying license compliance risks in datasets for ML
- Methodology for dataset license compliance verification
- Paper on license compliance methodology

---

### Data-Provenance-Initiative/Data-Provenance-Collection

> [github.com/Data-Provenance-Initiative/Data-Provenance-Collection](https://github.com/Data-Provenance-Initiative/Data-Provenance-Collection)

Tools for auto-generating structured provenance and attribution metadata:
- Symbolic Attribution framework (bibliography for data)
- Supports concatenation when datasets are merged

---

### SPDX License Detection

> [github.com/spdx/spdx-license-matcher](https://github.com/spdx/spdx-license-matcher)

Python tool for matching license text against SPDX license list:
- Aids in dataset compliance verification
- Supports license identification from text

---

## Organization Positions

### EFF (Electronic Frontier Foundation)

**"Copyright and AI: The Cases and the Consequences" (Feb 2025)**
> [eff.org](https://www.eff.org/deeplinks/2025/02/copyright-and-ai-cases-and-consequences)

- Argues developers have stronger fair use case
- Warning: Allowing claims based on "similar outputs" conflates style with unlawful copying
- Notes licensing market ($2.5B) emerging despite fair use likely covering training

**"AI and Copyright: Expanding Copyright Hurts Everyone" (Feb 2025)**
> [eff.org](https://www.eff.org/deeplinks/2025/02/ai-and-copyright-expanding-copyright-hurts-everyone-heres-what-do-instead)

- Expanded copyright would centralize AI among wealthy corporations
- Requiring license for fair uses could make ML research "prohibitively complicated"

---

### Creative Commons

**"Fair Use: Training Generative AI" (Feb 2023)**
> [creativecommons.org](https://creativecommons.org/2023/02/17/fair-use-training-generative-ai/)

Argues training generative AI should be **permitted as fair use**:
- Applies four-factor test
- AI uses works as data, not for aesthetic content
- Precedent: Authors Guild v. Google, Kelly v. Arriba Soft

**"Using CC-Licensed Works for AI Training" (May 2025)**
> [creativecommons.org](https://creativecommons.org/wp-content/uploads/2025/05/Using-CC-licensed-Works-for-AI-Training.pdf)

New guidance:
- If training data includes SA works, model/outputs should ideally be CC licensed
- Memorization may require attribution when model stores expressions substantially similar to CC work

---

### Stanford Foundation Model Transparency Index

> [ai.stanford.edu/fmti](https://ai.stanford.edu/fmti)

Graded 10 major AI models on training data transparency (2023–2025):
- **2023 scores**: 58/100 average
- **2025 scores**: 40/100 average (**declining**)

> *"Training data transparency has gotten worse, not better — even as regulatory requirements increase."*

---

## Key Findings

1. **Transparency crisis**: 70%+ of datasets have unspecified licenses; 50%+ of specified licenses are miscategorized (often more permissive than intended)

2. **Emerging licensing market**: ~$2.5B in deals between AI companies and content owners (OpenAI/Google + Reddit, WSJ, HarperCollins) signals that "free and clear" training data is becoming a competitive moat

3. **Legal landscape is unsettled**: Thomson Reuters v. Ross Intelligence (Feb 2025) is the first case to find AI training NOT fair use — but Bartz v. Anthropic ($1.5B settlement) involved clear piracy

4. **EU mandates disclosure**: Article 53 of EU AI Act now requires GPAI providers to publish training content summaries and comply with opt-out requests

5. **Documentation is poor and declining**: Stanford FMTI shows training data transparency scores dropping from 58→40/100 across 2023–2025

6. **Creative Commons is evolving**: CC's 2025 guidance on AI training acknowledges memorization as a factor requiring attribution in some cases

---

## Related Topics

- [Dataset Composition Analysis](./Dataset-Composition-Analysis.md) — Documentation frameworks and provenance
- [Data Contamination Reporting](./Data-Contamination-Reporting.md) — Dataset integrity and test set leakage
- [Regulatory Passports](../Automated-Machine-Readable-Artifacts/Regulatory-Passports.md) — Compliance frameworks
