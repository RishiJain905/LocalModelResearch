# Sensitivity Analysis: Layer-Wise Error

> **Research Method:** Direct HTTP fetching from arxiv.org/abs/ and GitHub API (April 2026).
>
> **GitHub:** [IST-DASLab/sparsegpt](https://github.com/IST-DASLab/sparsegpt) ⭐878 · [mit-han-lab/llm-awq](https://github.com/mit-han-lab/llm-awq) ⭐3.5K · [cmd2001/KVTuner](https://github.com/cmd2001/KVTuner) ⭐26 · [ggerganov/llama.cpp](https://github.com/ggerganov/llama.cpp) ⭐104.8K

---

## Overview

> *"Quantization errors don't distribute uniformly across transformer layers — certain layers act as information bottlenecks where errors accumulate and propagate."*

Layer-wise sensitivity analysis identifies which transformer layers are most vulnerable to quantization, enabling mixed-precision strategies that allocate more bits to sensitive layers.

---

## Error Propagation Patterns

### By Layer Position

| Layer Position | Typical Sensitivity | Reason |
|---------------|-------------------|--------|
| **Early (Embedding/QKV)** | High | Information bottleneck —失了global context aggregation |
| **Middle (FFN/Attention)** | Medium-High | Error amplification occurs here |
| **Late (Output)** | High | Errors directly affect output quality |

### By Layer Type

| Layer Type | Typical Sensitivity | Reason |
|------------|-------------------|--------|
| **Embedding** | High | Information bottleneck |
| **QKV Projection** | High | Critical for attention quality |
| **Attention Output** | Medium-High | Affects token representations |
| **FFN Layer 1** | Medium | First transformation layer |
| **FFN Layer 2** | Medium | Output directly affects next layer |
| **Output Linear** | High | Directly determines predictions |

---

## Key Papers

### "SparseGPT: One-Pass GPT Sparsification" (Frantar & Rothchild, 2023)

> **arXiv:** [2301.00774](https://arxiv.org/abs/2301.00774) | **GitHub:** [IST-DASLab/sparsegpt](https://github.com/IST-DASLab/sparsegpt)

> *"Shows layer-wise pruning patterns — sensitive layers require different sparsity levels."*

Key finding: Fisher information and Hessian-based methods identify layer sensitivity more accurately than uniform approaches.

---

### Mixed-Precision Quantization

> See: [Bit Allocation Policies](../Mixed-Precision/bit-allocation-policies.md)

Layer-wise sensitivity informs per-layer bit allocation — sensitive layers get higher precision.

---

## Key Insight: Mixed-Precision Wins

> Not all layers are equal — a **W4A8** mixed-precision strategy where sensitive layers use **W8A8** can outperform uniform **W4A4** with the same total bit budget.

### Example Bit Allocation

| Strategy | Sensitive Layers | Robust Layers | Result |
|----------|----------------|--------------|--------|
| Uniform W4A4 | 4-bit | 4-bit | Baseline |
| Mixed W4A8/W8A8 | 8-bit | 4-bit | Better accuracy, same VRAM |
| Mixed W4A4/W6A6 | 6-bit | 4-bit | Moderate improvement |

---

## Methods for Measuring Sensitivity

| Method | Description |
|--------|-------------|
| **Fisher Information** | Measures how much each parameter contributes to the loss |
| **Hessian-based** | Second-order approximations of error sensitivity |
| **Activation outlier** | Layers with more activation outliers need higher precision |
| **Perplexity ablation** | Remove quantize each layer and measure perplexity impact |
| **Task accuracy ablation** | Remove quantize each layer and measure task accuracy |

---

## Repos

| Resource | Link | Description |
|----------|------|-------------|
| SparseGPT | [IST-DASLab/sparsegpt](https://github.com/IST-DASLab/sparsegpt) ⭐878 | Layer-wise sparsity |
| AWQ | [mit-han-lab/llm-awq](https://github.com/mit-han-lab/llm-awq) ⭐3.5K | Activation-aware weight quantization |

---

## Implications

1. **Per-layer precision** is more efficient than uniform precision
2. **Sensitive layers** often cluster in specific regions (early/middle)
3. **Automated search** can find optimal bit allocation given a budget
4. **Hardware-aware** allocation matters — memory-bound vs compute-bound layers

---

## See Also

- [Eval Metrics README](./README.md)
- [Bit Allocation Policies](../Mixed-Precision/bit-allocation-policies.md)
- [AWQ](../weight-quantization/awq.md)
