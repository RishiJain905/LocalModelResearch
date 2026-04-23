# Loss-Seeding

## 📌 Overview

> **Loss-seeding** is a technique where training examples with **high loss** are identified as "weak areas," and synthetic or augmented data is generated specifically for those regions — seeding new training data from the model's own error signal.

The term "seeding" reflects the idea that **high-loss examples are seeds** from which targeted training data grows.

---

## 📚 Key Papers & Concepts

### 1. Max-Loss Subset Selection + Data Augmentation — NeurIPS 2022

**Paper:** *"Data-Efficient Augmentation for Training Neural Networks"*  
**URL:** [https://arxiv.org/pdf/2210.08363](https://arxiv.org/pdf/2210.08363)

> *"Max-loss baseline selects a new subset at every subset selection step to capture training dynamics."*

**Core Method:**
- Identifies data points where the model has **highest loss**
- Generates augmented versions of those high-loss examples
- Uses **gradient-based coreset selection** to find examples that, when augmented, closely capture full data augmentation dynamics

> *"Up to 6.3x speedup while maintaining performance."*

**Key Insight:** Instead of generating entirely new data, augmenting high-loss existing examples is highly efficient.

---

### 2. USAGE: Unified Seed Area Generation — ICCV 2023

**Paper:** *"USAGE: A Unified Seed Area Generation Paradigm for Weakly Supervised Semantic Segmentation"*  
**URL:** [https://openaccess.thecvf.com/content/ICCV2023/papers/Peng_USAGE_A_Unified_Seed_Area_Generation_Paradigm_for_Weakly_Supervised_ICCV_2023_paper.pdf](https://openaccess.thecvf.com/content/ICCV2023/papers/Peng_USAGE_A_Unified_Seed_Area_Generation_Paradigm_for_Weakly_Supervised_ICCV_2023_paper.pdf)

> *"Uses generation loss, which controls the shape of seed areas by a temperature parameter, plus regularization loss for consistency."*

**Core Method:**
- Generates initial seed areas using **loss optimization** in neural networks
- The "seed" is the region of the input space where the model is most uncertain
- Generalizable framework: originally for image segmentation, applicable to any domain

---

### 3. Synthetic Data Generation for SDG Classification — Swiss-Text 2024

**Paper:** *"Synthetic Data Generation for SDG Classification"*  
**URL:** [https://aclanthology.org/2024.swisstext-1.49.pdf](https://aclanthology.org/2024.swisstext-1.49.pdf)

> *"Synthetic data significantly enhances model performance for multi-class SDG classification."*

**Core Method:**
- Uses LLMs to generate synthetic training data for **Sustainable Development Goal (SDG) classification**
- Generates data targeting classes where the model is weakest
- Uses **AttributePrompt-based generation** with research domains, sub-topics, and abstract styles

**Key Finding:** AttributePrompt-based loss-weighted generation improves classification on weak classes.

---

## 🔧 Methods Breakdown

| Method | Loss Signal Used | What Gets Generated | Domain |
|--------|-----------------|---------------------|--------|
| **Max-Loss Coreset** | Highest-loss examples | Augmented versions of high-loss examples | General ML |
| **USAGE** | Generation loss + regularization | Seed areas via optimization | Image segmentation |
| **SDG AttributePrompt** | Per-class performance | Synthetic texts for weak classes | NLP classification |

---

## 🛠️ Repos & Implementations

| Repo | Description |
|------|-------------|
| [arxiv.org/pdf/2210.08363](https://arxiv.org/pdf/2210.08363) | Max-Loss Coreset paper + links to code |
| [USAGE ICCV 2023](https://openaccess.thecvf.com/content/ICCV2023/papers/Peng_USAGE_A_Unified_Seed_Area_Generation_Paradigm_for_Weakly_Supervised_ICCV_2023_paper.pdf) | Seed area generation paper |
| [SDG Classification Benchmark](https://github.com/SDGClassification/benchmark) | SDG classification dataset + benchmarks |
| [LLM UN SDG Awareness](https://github.com/marscod/Examining_LLM_UN_SDG) | LLM awareness of UN SDGs |

---

## 💡 Key Insights

1. **Loss as GPS** — The model's loss surface acts as a map pointing to exactly where it needs more training data
2. **Seed, don't blanket** — Instead of generating data for all classes equally, seed new data from high-loss regions
3. **Augment vs. Generate** — Can either augment existing high-loss examples OR generate entirely new ones targeting the same loss region
4. **Temperature controls coverage** — As seen in USAGE, the temperature parameter in generation loss controls how broad vs. narrow the seeded area is
