# Commercial Thresholds

> **Papers:** ["From Hugging Face to GitHub: Tracing License Drift in the Open-Source AI Ecosystem"](https://arxiv.org/abs/2509.09873) · ["The Mirage of Artificial Intelligence Terms of Use Restrictions"](https://law.stanford.edu/wp-content/uploads/2025/03/ssrn-5049562.pdf)  
> **Tools:** [Black Duck AI Model Scanning](https://www.blackduck.com/blog/black-duck-ai-model-scanning.html) · [LicenseRec](https://arxiv.org/abs/2509.09873)  
> **License Terms:** [Meta Llama License](https://ai.meta.com/llama/license/) · [Qwen License](https://huggingface.co/Qwen/Qwen2.5-72B-Instruct/blob/main/LICENSE) · [Stability AI License](https://stability.ai/license) · [Mistral License](https://help.mistral.ai/en/articles/347393)

---

## Overview

Many widely used "open-weight" AI model licenses impose **commercial thresholds** — caps on revenue, monthly active users, or usage scale beyond which the license auto-terminates or requires a separate commercial agreement. These thresholds are one of the clearest markers distinguishing truly unrestricted licenses (e.g. Apache 2.0) from custom corporate licenses.

> *"The restrictive and inconsistent licensing of so-called 'open' AI models is creating significant uncertainty, particularly for commercial adoption. The current landscape of AI model licensing is riddled with confusion, restrictive terms, and misleading claims of openness."*
> — **Nick Vidal, Head of Community, Open Source Initiative** — [TechCrunch, February 2025](https://techcrunch.com/2025/03/14/open-ai-model-licenses-often-carry-concerning-restrictions/)

---

## Commercial Thresholds by Model

### Major Model License Thresholds

| Model / Provider | Commercial Threshold | Metric Type | Hard or Soft? | License Type |
|-----------------|----------------------|-------------|--------------|--------------|
| **Meta Llama 3** | **700M Monthly Active Users** | Scale / MAU | **Hard cap** — license auto-expires | Custom Community License |
| **Alibaba Qwen 2.5+** | **100M Monthly Active Users** | Scale / MAU | Hard cap — must request authorization | Custom Qwen License |
| **Mistral AI** | **$20M USD monthly revenue** | Revenue | Must obtain commercial license | Modified MIT |
| **Stability AI (SD3, SDXL)** | **$1M USD annual revenue** | Revenue | Free below threshold; Enterprise required above | Community License |
| **Google Gemma (pre-v4)** | **None** | None | No scale cap, but derivative-data clause applies | Custom Gemma Terms |
| **Google Gemma 4** | **None** | None | No restrictions | Apache 2.0 |
| **DeepSeek-V3** | **None** | None | No revenue cap; use-based restrictions apply | Custom DeepSeek License |
| **Cohere C4AI / Aya** | **Non-commercial only** | Prohibited | CC BY-NC 4.0 — strictly no commercial use | CC BY-NC |
| **BigCode models** | **None** | None | No revenue/MAU cap; use-based restrictions apply | BigCode OpenRAIL-M |

---

## Key License Terms & Quotes

### Meta Llama 3 — 700M MAU Hard Cap

> *"The Llama 3 Community License allows commercial use — but it contains a clause that has no equivalent in any open-source licence: a 700 million monthly active user threshold above which your free licence automatically expires, a competitor restriction that blocks entire industries, and a ban on using Llama to train other models."*
> — [WCR Legal](https://wcr.legal/llama-3-license-700m-mau-limit/)

> *"AI model creators commonly attach restrictive terms of use to both their models and their outputs... Often taken at face value, these terms are positioned by companies as key enforceable tools for preventing misuse... But are these terms truly meaningful, or merely a mirage?"*
> — **Henderson & Lemley**, Stanford Law / Indiana Law Journal (2025) — [PDF](https://law.stanford.edu/wp-content/uploads/2025/03/ssrn-5049562.pdf)

### Alibaba Qwen — 100M MAU

> *"If you are commercially using the Materials, and your product or service has more than 100 million monthly active users, you shall request a license from us. You cannot exercise your rights under this Agreement without our express authorization."*
> — [Qwen License Agreement](https://huggingface.co/Qwen/Qwen2.5-72B-Instruct/blob/main/LICENSE)

### Mistral — $20M/Month Revenue

> *"Companies with monthly revenue exceeding $20M USD must either: Obtain a commercial license by contacting us, or Use the models via Mistral AI Studio."*
> — [Mistral Help Center](https://help.mistral.ai/en/articles/347393-under-which-license-are-mistral-s-open-models-available)

### Stability AI — $1M/Year Revenue

> *"Use of the Core Models (including Stable Diffusion 3 2B) is free for everyone, unless you're using the Core Models for a commercial purpose and you or your organization generate over USD $1M (or local currency equivalent) of annual revenue, regardless of the source of that revenue."*
> — [Stability AI License](https://stability.ai/license)

> *"A model foundry can put out [open] models, wait to see what business cases develop using those models, and then strong-arm their way into successful verticals by either extortion or lawfare."*
> — **Eric Tramel, Gretel**, via [TechCrunch](https://techcrunch.com/2025/03/14/open-ai-model-licenses-often-carry-concerning-restrictions/)

---

## License Restriction Comparison Matrix

| License Family | Revenue Cap | User / MAU Cap | Competitor Ban | Output Training Ban | Remote Kill Switch |
|---------------|-------------|----------------|---------------|--------------------|--------------------|
| **Meta Llama 3** | None | 700M MAU | ✅ Yes | ✅ Yes | No |
| **Mistral Modified MIT** | $20M/month | None | No | No | No |
| **Alibaba Qwen** | None | 100M MAU | ❌ (IP retaliation instead) | Attribution required | No |
| **Stability AI Community** | $1M/year | None | ✅ Yes (foundational) | ✅ Yes | No |
| **Google Gemma (pre-4)** | None | None | No | Derivative-data clause | **Yes** |
| **DeepSeek** | None | None | No | No | No |
| **Apache 2.0 (Gemma 4, etc.)** | None | None | No | No | No |
| **OpenRAIL-M** | None | None | No | No | No |
| **CC BY-NC (Cohere)** | Non-commercial only | N/A | N/A | N/A | No |

---

## Legal Analysis & Industry Commentary

| Publication | Title | Key Findings | URL |
|-------------|-------|--------------|-----|
| **TechCrunch (Feb 2025)** | "'Open' AI model licenses often carry concerning restrictions" | OSI says licenses are "riddled with confusion"; Gemma remote restriction called out; custom licenses exclude small companies | [Link](https://techcrunch.com/2025/03/14/open-ai-model-licenses-often-carry-concerning-restrictions/) |
| **WCR Legal** | "Meta Llama 3 and the 700M MAU Limit" | Deep legal analysis: three critical restrictions; due diligence implications; product-level MAU ambiguity; M&A risk | [Link](https://wcr.legal/llama-3-license-700m-mau-limit/) |
| **Stanford / Indiana Law (2025)** | "The Mirage of Artificial Intelligence Terms of Use Restrictions" | Argues ToU restrictions may be **unenforceable** because model weights/outputs likely uncopyrightable | [PDF](https://law.stanford.edu/wp-content/uploads/2025/03/ssrn-5049562.pdf) |
| **The Verge (Oct 2024)** | "Open-source AI must reveal its training data, per new OSI definition" | OSI's OSAID challenges Meta's Llama; Meta disagrees with definition | [Link](https://www.theverge.com/2024/10/28/24281820/open-source-initiative-definition-artificial-intelligence-meta-llama) |
| **Lawfare** | "Open-Access AI: Lessons From Open-Source Software" | Argues "open-source AI" is a misnomer; distinguishes open-weight vs open-data dynamics | [Link](https://www.lawfaremedia.org/article/open-access-ai--lessons-from-open-source-software) |
| **a16z** | "To Help AI Startups Compete, Use Revenue-based Thresholds" | Argues for revenue-based (not compute-based) regulatory thresholds; ironic that open models may increase calculated training costs | [Link](https://a16z.com/to-help-ai-startups-compete-use-revenue-based-thresholds/) |
| **Black Duck** | "DeepSeek model license review" | Analyzes DeepSeek license as not meeting OSI definition due to use-based restrictions; notes persistence through derivatives | [Link](https://www.blackduck.com/blog/deepseek-license.html) |

---

## Tools & Frameworks for License Tracking

| Tool / Framework | Organization | What It Does | URL |
|-----------------|-------------|-------------|-----|
| **LicenseRec** | Academic (Hugging Face paper, arXiv:2509.09873) | ML-aware license compatibility checker; detects 35.5% violation rate at model-to-repo stage | [arXiv 2509.09873](https://arxiv.org/abs/2509.09873) |
| **Black Duck AI Model Risk Insights** | Black Duck (Synopsys) | Scans code to identify hidden/modified AI models; license compliance checks; supports EU AI Act compliance | [Blog](https://www.blackduck.com/blog/black-duck-ai-model-scanning.html) |
| **Open-Future-Foundation/AI-model-licensing** | Open Future Foundation | Categorizes Hugging Face licenses into 6 categories; analyzes OpenRAIL-M variants | [GitHub](https://github.com/Open-Future-Foundation/AI-model-licensing) |
| **Hugging Face License Tags** | Hugging Face | Model cards display license info; supports SPDX identifiers including OpenRAIL++, OpenMDW | [Docs](https://huggingface.co/docs/hub/repositories-licenses) |

> *"35.5% of model-to-application transitions eliminate restrictive license clauses by relicensing under permissive terms... Our proposed ML-aware compatibility matrix significantly increases the detected license violation rates compared to traditional matrices."*
> — LicenseRec paper, arXiv:2509.09873

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Dominant pattern** | Scale thresholds dominate: Meta (700M MAU), Qwen (100M MAU), Mistral ($20M/month), Stability AI ($1M/year) |
| **OSI stance** | Custom licenses with field-of-use restrictions are **not open source** regardless of threshold generosity |
| **Legal risk** | Stanford/Henderson & Lemley argue many ToU restrictions are unenforceable due to copyright preemption on uncopyrightable weights |
| **Gold standard** | Apache 2.0 — fully permissive, no revenue/MAU caps, no competitor bans (Gemma 4, many community models) |
| **Tooling gap** | No widely adopted open-source "license checker" exists; LicenseRec is academic research; Black Duck is enterprise SCA |

---

*Last updated: 2026-04-21*
