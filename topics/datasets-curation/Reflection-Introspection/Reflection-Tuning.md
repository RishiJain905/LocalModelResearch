# Reflection-Tuning

## 📌 Overview

> **Reflection-Tuning** is a data recycling method that uses an oracle LLM (e.g., GPT-4) to introspect and enhance instruction-response pairs in existing training data — improving LLM instruction-tuning without additional data collection.

> *"This approach utilizes an oracle LLM to recycle the original training data by introspecting and enhancing the quality of instructions and responses in the data."* — Li et al., NeurIPS 2023

---

## 📚 Key Papers

### V1: Reflection-Tuning — NeurIPS 2023 Workshop

**Paper:** *"Reflection-Tuning: Data Recycling Improves LLM Instruction-Tuning"*  
**URL:** [arXiv:2310.11716](https://arxiv.org/abs/2310.11716)  
**Authors:** Ming Li, Lichang Chen, Jiuhai Chen, Shwai He, Heng Huang (UMD), Jiuxiang Gu (Adobe Research)

**Core Process:**
```
Instruction → Oracle reflects on instruction-response → Refined instruction
                      ↓
Refined Response → Oracle reflects on response → Final recycled pair (x_ins, y_res)
```

**Reflection Criteria for Instructions:**
1. Complexity of the Topic
2. Level of Detail Required
3. Knowledge Required
4. Ambiguity of the Instruction
5. Logical Reasoning Involved

**Reflection Criteria for Responses:**
1. Helpfulness
2. Relevance
3. Accuracy
4. Level of Details

---

### V2: Selective Reflection-Tuning — ACL 2024 Findings

**Paper:** *"Selective Reflection-Tuning: Student-Selected Data Recycling for LLM Instruction-Tuning"*  
**URL:** [arXiv:2402.10110](https://arxiv.org/abs/2402.10110) | [ACL Anthology](https://aclanthology.org/2024.findings-acl.958/)

**Key Innovation:** Introduces teacher-student collaboration where the **student model actively selects** which enhanced samples to incorporate.

**Two New Metrics:**
- **IFD (Instruction-Following Difficulty):** Selects instructions with higher difficulty for the student
- **r-IFD (Reversed-IFD):** Selects responses with lower feasibility cost (easier for student to learn)

> *"With less than 1,000 selective recycled data, our 'sRecycled WizardLM 7B (2%) (926)' outperforms most existing 7B models, including LIMA, which is trained with manually curated data samples."*

---

## 📊 Benchmark Results

### AlpacaEval Leaderboard
| Model | Win Rate |
|-------|----------|
| GPT-4 | 95.28% |
| Claude 2 | 91.36% |
| **sRecycled WizardLM 7B** | **83.48%** |
| **Recycled Alpaca 7B** | **76.99%** |
| Vicuna 7B v1.3 | 76.84% |
| Alpaca 7B (original) | 26.46% |

### Open LLM Leaderboard (7B models)
| Model | Average |
|-------|---------|
| Vicuna 7B v1.3 | 55.63 |
| **sRecycled WizardLM 7B** | **56.79** |
| **sRecycled Alpaca 7B** | **56.05** |
| Alpaca 7B (original) | 50.21 |

---

## 🛠️ Repos & Datasets

**GitHub:** [tianyi-lab/Reflection_Tuning](https://github.com/tianyi-lab/Reflection_Tuning)

**Huggingface Datasets:**
| Dataset | Link |
|---------|------|
| Recycled Alpaca V1 | [umd-zhou-lab/recycled_alpaca_v1](https://huggingface.co/datasets/umd-zhou-lab/recycled_alpaca_v1) |
| Recycled WizardLM70k V1 | [umd-zhou-lab/recycled_wiz70_v1](https://huggingface.co/datasets/umd-zhou-lab/recycled_wiz70_v1) |
| sRecycled Alpaca V2 | [umd-zhou-lab/sRecycled_Alpaca](https://huggingface.co/datasets/umd-zhou-lab/sRecycled_Alpaca) |
| sRecycled WizardLM70k V2 | [umd-zhou-lab/sRecycled_Wiz70](https://huggingface.co/datasets/umd-zhou-lab/sRecycled_Wiz70) |

**IFD Score Datasets:**
| Dataset | Link |
|---------|------|
| Alpaca IFD (llama2-7b) | [MingLiiii/Alpaca_Analysis_llama2_7b](https://huggingface.co/datasets/MingLiiii/Alpaca_Analysis_llama2_7b) |
| Alpaca IFD (llama2-13b) | [MingLiiii/Alpaca_Analysis_llama2_13b](https://huggingface.co/datasets/MingLiiii/Alpaca_Analysis_llama2_13b) |
| Wiz70 IFD (llama2-7b) | [MingLiiii/Wiz70_Analysis_llama2_7b](https://huggingface.co/datasets/MingLiiii/Wiz70_Analysis_llama2_7b) |
| Wiz70 IFD (llama2-13b) | [MingLiiii/Wiz70_Analysis_llama2_13b](https://huggingface.co/datasets/MingLiiii/Wiz70_Analysis_llama2_13b) |

---

## 🔗 Related Work

- **ALPAGASUS:** Data filtering approach (similar goal, different method)
- **Cherry LLM:** Proposed IFD metric that Selective Reflection-Tuning builds upon
- **LIMA:** Manual data curation comparison point
