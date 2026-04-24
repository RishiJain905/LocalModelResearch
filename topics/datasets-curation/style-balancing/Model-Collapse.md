# Model Collapse

## 📌 Overview

**Model collapse** (also called Model Autophagy Disorder or MAD) occurs when models trained on synthetic data iteratively drift from the true data distribution, losing rare patterns, minority perspectives, and edge-case reasoning capabilities.

> *"Repeated training on synthetic data causes model distribution to drift from reality, reinforcing biases, amplifying artifacts."*
> — [Rice DSP](https://dsp.rice.edu/ai-loops/)

---

## The Collapse Mechanism

> *"When a model trained on synthetic data generates more synthetic data, the distribution shifts — low-probability events become increasingly rare, then disappear entirely."*
> — [Shumailov et al. 2023](https://www.nature.com/articles/s41586-023-06647-8)

### What Gets Lost

| Pattern | Effect |
|---------|--------|
| **Rare vocabulary** | Atrophy of uncommon words |
| **Minority perspectives** | Loss of underrepresented views |
| **Edge-case reasoning** | Degradation of complex patterns |
| **Factual accuracy** | Hallucinations and factual drift |

---

## Timeline & Severity

> *"MAD typically manifests after ~5 training cycles."*
> — [Rice DSP](https://dsp.rice.edu/ai-loops/)

| Cycle | Effect |
|-------|--------|
| 1-2 | Minor distribution shift |
| 3-4 | Visible quality degradation |
| 5+ | Catastrophic — recovery increasingly difficult |

---

## Key Papers

### Shumailov et al. 2023 (Nature)

> *"The collapse of generative models — repeated training on model-generated content degrades subsequent generations."*
> — [Nature Paper](https://www.nature.com/articles/s41586-023-06647-8)

### Alemohammad et al. 2024 (ICLR)

> *"Self-Consuming Generative Models Go MAD — model collapse in iterative training loops."*
> — [ICLR Paper](https://arxiv.org/abs/2405.15030)

### Key Finding

> *"Even small amounts of synthetic data in training can lead to measurable performance degradation on rare patterns."*
> — [Alemohammad et al.](https://arxiv.org/abs/2405.15030)

---

## Mitigation Strategies

| Strategy | Description | Source |
|---------|-------------|--------|
| **Mix real + synthetic** | Accumulating real data prevents complete collapse | Rice DSP |
| **Watermarking** | Track and curate data provenance | Research |
| **Active filtering** | Remove low-quality or model-generated content | Research |
| **Constitutional AI** | Synthetic data against human-defined principles | Anthropic |
| **Curated preference alignment** | Ferbach et al. 2024 approach | Ferbach et al. |
| **Minimum real data floor** | Never drop below 50-80% real data | Industry practice |

---

## Relationship to Benchmark Contamination

Model collapse is distinct from but related to benchmark contamination:

| Issue | Cause | Effect |
|-------|-------|--------|
| **Benchmark contamination** | Training on test data | Inflated evaluation scores |
| **Model collapse** | Iterative synthetic data training | Distribution atrophy |

---

## Warning Signs

> *"Performance improves on common cases but degrades on rare patterns — a hallmark of early-stage model collapse."*
> — [Rice DSP](https://dsp.rice.edu/ai-loops/)

### Detection

- Monitor performance on tail distributions
- Track vocabulary diversity metrics
- Compare perplexity on held-out real data
- Check for increasing similarity between consecutive model generations

---

## Sources

| Type | Citation |
|------|----------|
| **Paper** | [Shumailov et al. 2023 Nature](https://www.nature.com/articles/s41586-023-06647-8) |
| **Paper** | [Alemohammad et al. ICLR 2024](https://arxiv.org/abs/2405.15030) |
| **Resource** | [Rice DSP — Self-Consuming AI](https://dsp.rice.edu/ai-loops/) |

---

## Related Topics

- [Dataset-Mix-Optimization.md](./Dataset-Mix-Optimization.md) — Optimal data mixture strategies
- [../Synthetic-flywheels/](../Synthetic-flywheels/) — Synthetic data generation loops
