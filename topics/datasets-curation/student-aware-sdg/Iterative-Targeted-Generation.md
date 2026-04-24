# Iterative Targeted Generation

## 📌 Overview

> Iterative Targeted Generation refers to the process of **cyclically identifying model weaknesses, generating synthetic training data targeting those weaknesses, and re-training** — creating a feedback loop that progressively narrows the gap between the student model's capabilities and its target performance.

This is a core technique in student-aware synthetic data generation (SDG), where the student model's evolving state continuously informs what data to generate next.

---

## 📚 Key Papers

### 1. Targeted Data Generation (TDG) — ACL 2023

**Paper:** *"Targeted Data Generation: Finding and Fixing Model Weaknesses"*  
**Authors:** He, Ribeiro & Khani (Microsoft)  
**URL:** [https://aclanthology.org/2023.acl-long.474.pdf](https://aclanthology.org/2023.acl-long.474.pdf)

> *"TDG identifies challenging clusters that can benefit from additional data while minimizing potential negative impacts on performance in other regions."*

**Core Method:**
- Automatically identifies **challenging subgroups (clusters)** where models fail systematically
- Generates new training data specifically for those subgroups using LLMs
- Uses two key metrics:
  - **GC (Generalization in Context)** — selects clusters that benefit most from augmentation
  - **IC (Interference in Context)** — avoids clusters where augmentation could hurt other areas

---

### 2. DPE: Diagnostic-Driven Progressive Evolution — 2024

**Paper:** *"From Blind Spots to Gains: Diagnostic-Driven Iterative Training for Large Multimodal Models"*  
**URL:** [https://arxiv.org/abs/2602.22859](https://arxiv.org/abs/2602.22859)

> *"Research on human learning shows that test-driven error exposure followed by targeted correction outperforms repetitive practice."*

**Core Method — The Spiral Loop:**
```
Diagnosis → Generation → Reinforcement → Re-Diagnosis
     ↑                                          |
     └──────────────────────────────────────────┘
```

- Closes the feedback loop between **diagnosing model weaknesses** and **generating targeted training examples**
- Iteratively refines the model by targeting discovered blind spots

---

### 3. Active Synthetic Data Generation for Finetuning — 2024

**URL:** [https://iterative-sd.github.io](https://iterative-sd.github.io)

> *"Active methods are 1.3x-2x more efficient than static generation."*

**Core Method:**
- Generates synthetic data *as training progresses*, guided by the current state of the student model
- Uses **uncertainty (loss)** as the selection criterion
- Key finding: Simple loss-based selection performs as well as complex LLM-judge methods

---

### 4. APT: Adversarial Preference Training

**URL:** [https://www.aimodels.fyi/papers/arxiv/apt-improving-specialist-llm-performance-weakness-case](https://www.aimodels.fyi/papers/arxiv/apt-improving-specialist-llm-performance-weakness-case)

**Three-Stage Process:**
1. **Weakness Discovery** — Identify where the model fails
2. **Preference Collection** — Gather examples of better outputs
3. **Iterative Training** — Fine-tune using the collected preferences

> *"15-25% improvement in specialized task performance."*

---

## 🔧 Comparison Table

| Method | Key Signal | Iteration Style | Application |
|--------|-----------|-----------------|-------------|
| **TDG** | Cluster failure analysis | Batch iterative | NLP text generation |
| **DPE** | Diagnostic evaluation | Spiral loop | Multimodal LMs |
| **Active SDG** | Loss/uncertainty | Per-step active | LLM fine-tuning |
| **APT** | Adversarial weakness probe | 3-stage cycle | Specialist LLM improvement |

---

## 🛠️ Repos & Implementations

| Repo | Description |
|------|-------------|
| [iterative-sd.github.io](https://iterative-sd.github.io) | Active synthetic data generation benchmarks |
| [TDG (ACL 2023)](https://aclanthology.org/2023.acl-long.474.pdf) | Official paper implementation (contact authors) |
| [DPE implementation](https://arxiv.org/abs/2602.22859) | Check paper for official code link |

---

## 💡 Key Insights

1. **Feedback loops are essential** — Single-pass data generation misses many weaknesses; iterative loops progressively uncover and fix more
2. **Loss is a strong signal** — Simpler than LLM-judge, yet equally effective for identifying what to generate
3. **Targeted > generic** — Generating for specific weak clusters outperforms adding more generic data
4. **Minimize interference** — Good iterative methods check that fixing weakness A doesn't introduce weakness B
