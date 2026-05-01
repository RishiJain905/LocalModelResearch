# Bandwidth-Efficient Decentralized Training

> Bandwidth-efficient decentralized training refers to methods that reduce the communication overhead of distributed SGD — the dominant bottleneck in multi-node training — through gradient compression, sparsification, quantization, topology optimization, and adaptive compression strategies.

## Contents

1. [Key Papers](#key-papers)
2. [Open-Source Libraries & Tools](#open-source-libraries--tools)
3. [GitHub Repositories](#github-repositories)
4. [Blog Posts & Articles](#blog-posts--articles)
5. [Key Findings](#key-findings)
6. [Related Topics](#related-topics)

---

## Key Papers

### Deep Gradient Compression: Reducing the Communication Bandwidth for Distributed Training (ICLR 2018)

> *"We find 99.9% of the gradient exchange in distributed SGD is redundant, and propose Deep Gradient Compression (DGC) to greatly reduce the communication bandwidth."*
> — Yujun Lin, Song Han, Huizi Mao, Yu Wang, William J. Dally
> — [arXiv:1712.01887](https://arxiv.org/abs/1712.01887v3)

**Key Contributions:**
- DGC achieves a gradient compression ratio from **270× to 600×** without losing accuracy
- Cuts the gradient size of ResNet-50 from 97 MB to 0.35 MB
- Employs four preservation methods: momentum correction, local gradient clipping, momentum factor masking, and warm-up training

---

### Communication-Efficient Training Workload Balancing for Decentralized Multi-Agent Learning / ComDML (IEEE ICDCS 2024)

> *"ComDML can significantly reduce the overall training time while maintaining model accuracy, compared to state-of-the-art methods."*
> — Seyed Mahmoud Sajjadi Mohammadabadi, Lei Yang, Feng Yan, Junshan Zhang
> — [arXiv:2405.00839](https://arxiv.org/abs/2405.00839) | [DOI](https://doi.org/10.1109/ICDCS60910.2024.00069)

**Key Contributions:**
- Heterogeneity-aware workload balancing for decentralized training
- Minimizes end-to-end training time in compute/network-heterogeneous environments

---

### Hop: Heterogeneity-Aware Decentralized Training (ACM ASPLOS 2019)

> *"This paper proposes Hop, the first heterogeneity-aware decentralized training protocol."*
> — Qinyi Luo, Jinkun Lin, Youwei Zhuo, Xuehai Qian
> — [arXiv:1902.01064](https://arxiv.org/abs/1902.01064) | [DOI](https://doi.org/10.1145/3297858.3304009)

**Key Contributions:**
- Identifies the **"iteration gap"** as the core challenge in heterogeneous decentralized training
- Proposes a queue-based synchronization mechanism
- Delivers significant speedup over standard decentralized SGD in heterogeneous settings

---

### MATCHA: Speeding Up Decentralized SGD via Matching Decomposition Sampling (NeurIPS 2019)

> *"The main idea of MATCHA is to parallelize inter-node communication by decomposing the topology into matchings."*
> — Jianyu Wang, Anit Kumar Sahu, Zhouyi Yang, Gauri Joshi, Soummya Kar
> — [arXiv:1905.09435](https://arxiv.org/abs/1905.09435)

**Key Contributions:**
- MATCHA takes up to **5× less time** than vanilla decentralized SGD to reach the same training loss
- Decomposes communication graphs into matchings to exploit network parallelism

---

### Train Where the Data is: A Case for Bandwidth Efficient Coded Training (arXiv 2019)

> *"We demonstrate a 50% reduction in total required communication bandwidth compared to MDS coded computation."*
> — Zhifeng Lin, Krishna Giri Narra, Mingchao Yu, Salman Avestimehr, Murali Annavaram
> — [arXiv:1910.10283](https://arxiv.org/abs/1910.10283)

**Key Contributions:**
- Uses Random Linear Network Coding (RLNC) to reduce cross-device data exchange in mobile training
- Reduces bandwidth requirement by **50%** vs. MDS-coded computation

---

### Energy-efficient Decentralized Learning via Graph Sparsification (IEEE ICASSP 2024)

> *"The proposed solution can lower the energy consumption at the busiest node by 54%-76% while maintaining the quality of the trained model."*
> — Xusheng Zhang, Cho-Chun Chiu, Ting He
> — [arXiv:2401.03083](https://arxiv.org/abs/2401.03083)

**Key Contributions:**
- Optimizes the mixing matrix which controls communication demands
- Reduces energy at the busiest node by **54%–76%**

---

### MoTEF: Towards Faster Decentralized Stochastic Optimization with Communication Compression (arXiv 2024)

> *"Our analysis demonstrates that MoTEF achieves most of the desired properties, and significantly outperforms existing methods under arbitrary data heterogeneity."*
> — Rustem Islamov, Yuan Gao, Sebastian U. Stich
> — [arXiv:2405.20114](https://arxiv.org/abs/2405.20114)

**Key Contributions:**
- Integrates **Momentum Tracking** and **Error Feedback** with communication compression
- First work to combine all three desired properties (compression, momentum tracking, error feedback)

---

### GraVAC: Adaptive Compression for Communication-Efficient Distributed DL Training (IEEE CLOUD 2023)

> *"GraVAC reduces end-to-end training time for ResNet101, VGG16 and LSTM by 4.32x, 1.95x and 6.67x respectively."*
> — Sahil Tyagi, Martin Swany
> — [arXiv:2305.12201](https://arxiv.org/abs/2305.12201) | [DOI](https://doi.org/10.1109/CLOUD60044.2023.00045)

**Key Contributions:**
- **Adaptive compression** — dynamically adjusts compression factor based on model progress and gradient information loss
- End-to-end training speedups of **1.95×–6.67×** depending on model architecture

---

### CESAR: Secure Aggregation Meets Sparsification in Decentralized Learning (arXiv 2024)

> *"With random subsampling, CESAR is always within 0.5% accuracy of decentralized parallel stochastic gradient descent (D-PSGD), while adding only 11% of data overhead."*
> — Sayan Biswas, Anne-Marie Kermarrec, Rafael Pires, Rishi Sharma, Milos Vujasinovic
> — [arXiv:2405.07708](https://arxiv.org/abs/2405.07708)

**Key Contributions:**
- Novel secure aggregation protocol compatible with existing sparsification mechanisms
- Adds only **11% data overhead** while preserving accuracy within **0.5%**

---

### Decentralized Federated Learning: A Segmented Gossip Approach (FML 2019)

> *"We propose a model segment level decentralized federated learning... which not only makes full utilization of node-to-node bandwidth, but also has good training convergence."*
> — Chenghao Hu, Jingyan Jiang, Zhi Wang
> — [arXiv:1908.07782](https://arxiv.org/abs/1908.07782)

**Key Contributions:**
- Segmented gossip approach for federated learning
- Maximizes node-to-node bandwidth utilization

---

### Decentralized Training of Foundation Models in Heterogeneous Environments (arXiv 2022)

> *"Across 8 different cities spanning 3 continents, our approach is 4.8X faster than prior state-of-the-art training systems (Megatron)."*
> — Binhang Yuan et al.
> — [arXiv:2206.01288](https://arxiv.org/abs/2206.01288)

**Key Contributions:**
- First study of training large foundation models with model parallelism in a decentralized regime
- **4.8× faster** than Megatron across geo-distributed nodes (8 cities, 3 continents)

---

### On the Surprising Effectiveness of a Single Global Merging in Decentralized Learning (ICLR 2026 Oral)

> *"Fully connected communication at the final step, implemented by a single global merging, can significantly improve the performance of decentralized learning under high data heterogeneity."*
> — Tongtian Zhu, Tianyu Zhang, Mingze Wang, Zhanpeng Zhou, Can Wang
> — [arXiv:2507.06542](https://arxiv.org/abs/2507.06542)

**Key Contributions:**
- First to establish that a **single global merge** at the final step matches the convergence rate of parallel SGD
- Surprisingly effective even under high data heterogeneity

---

## Open-Source Libraries & Tools

### Flower (FLWR)

> [flower.ai](https://flower.ai/) | `pip install flwr`

A comprehensive Federated Learning framework supporting large-scale FL experiments (up to 15M clients on a pair of high-end GPUs) and heterogeneous device scenarios. Researchers can migrate experiments to real devices seamlessly.

---

### Deep Gradient Compression (DGC)

> [GitHub: synxlin/deep-gradient-compression](https://github.com/synxlin/deep-gradient-compression)

Official PyTorch/TensorFlow reference implementation implementing gradient sparsification with momentum correction and local gradient clipping.

---

### ChocoSGD

> [GitHub: epfml/ChocoSGD](https://github.com/epfml/ChocoSGD)

Reference implementation for "Decentralized SGD and Consensus with Communication Compression." Supports Python/PyTorch decentralized training with quantized and sparsified communication.

---

### DisPFL

> [GitHub: rong-dai/DisPFL](https://github.com/rong-dai/DisPFL) | ICML 2022

Implements "DisPFL: Towards Communication-Efficient Personalized Federated Learning via Decentralized Sparse Training." Decentralized sparse training for personalized FL.

---

### GCFL

> [GitHub: hushyangqing/GCFL](https://github.com/hushyangqing/GCFL)

Gradient Compression in Federated Learning — implements various gradient compression and quantization methods for FL experiments.

---

### PyTorch Distributed / `torch.distributed`

> [PyTorch Docs](https://pytorch.org/docs/stable/distributed.html) | `pip install torch`

Core PyTorch distributed training package. Includes built-in gradient compression hooks (e.g., PowerSGD, quantization) and supports both parameter-server and decentralized (DDP) modes.

---

## GitHub Repositories

| Repository | Stars | Description |
|---|---|---|
| [epfml/ChocoSGD](https://github.com/epfml/ChocoSGD) | ⭐ 74 | Decentralized SGD with communication compression (quantization + sparsification). |
| [rong-dai/DisPFL](https://github.com/rong-dai/DisPFL) | ⭐ 83 | ICML 2022 code for DisPFL: decentralized sparse personalized federated learning. |
| [hushyangqing/GCFL](https://github.com/hushyangqing/GCFL) | ⭐ 1 | Gradient Compression in Federated Learning experiments. |
| [ltd-ARYAN-pvt/federated-learning-client](https://github.com/ltd-ARYAN-pvt/federated-learning-client) | ⭐ 1 | Flower-based FL client with quantization, pruning, gradient compression. |
| [jiajunsong811/FedLuck_Euro_Par](https://github.com/jiajunsong811/FedLuck_Euro_Par) | ⭐ 0 | Euro-Par 2024: joint local updating and gradient compression for async FL. |
| [sadafsk/FL_RD](https://github.com/sadafsk/FL_RD) | ⭐ 0 | Federated learning with rate-distortion gradient compression experiments. |
| [VinkuraAI/Docxo](https://github.com/VinkuraAI/Docxo) | ⭐ 7 | Efficient training framework for LLMs on GCP, focused on minimizing communication overhead. |
| [Space-Paragon1/Federated-learning-for-privacy](https://github.com/Space-Paragon1/Federated-learning-for-privacy) | ⭐ 0 | FL system with differential privacy, Byzantine-robust aggregation, gradient compression. |
| [ItsDarker/fl_smarthome_privacy](https://github.com/ItsDarker/fl_smarthome_privacy) | ⭐ 0 | FL privacy attack framework with gradient compression defenses. |

> Star counts retrieved May 2026. GitHub API rate limits mean this list is representative, not exhaustive.

---

## Blog Posts & Articles

### Flower Blog — Federated Learning Research Framework

> *"Flower can perform FL experiments up to 15M in client size using only a pair of high-end GPUs. Researchers can then seamlessly migrate experiments to real devices to examine other parts of the design space."*
> — [flower.ai](https://flower.ai/) | Paper: [arXiv:2007.14390](https://arxiv.org/abs/2007.14390)

---

### PyTorch Documentation — Distributed Communication Package

Official PyTorch docs covering DDP, RPC, collective communication, and gradient compression hooks.
> — [pytorch.org/docs/stable/distributed.html](https://pytorch.org/docs/stable/distributed.html)

---

### Weights & Biases — Communication-Efficient Federated Learning

Inferred resource. Direct extraction was limited by access restrictions during research.
> — [wandb.ai (citation needed for exact permalink)](https://wandb.ai)

---

### Google AI Blog — Federated Learning

Google's introduction to federated learning. Direct extraction was limited by bot mitigation.
> — [blog.google/technology/ai/federated-learning/](https://blog.google/technology/ai/federated-learning/)

---

## Key Findings

1. **Gradient redundancy is massive in distributed SGD.**
   > *"We find 99.9% of the gradient exchange in distributed SGD is redundant"* — Deep Gradient Compression (ICLR 2018).

2. **Compression ratios of 270×–600× are achievable without accuracy loss.**
   > *"Deep Gradient Compression achieves a gradient compression ratio from 270x to 600x without losing accuracy, cutting the gradient size of ResNet-50 from 97MB to 0.35MB"* — Deep Gradient Compression (ICLR 2018).

3. **Decentralized learning can outperform centralized baselines when bandwidth is limited, if the topology is tuned.**
   > *"MATCHA takes up to 5x less time than vanilla decentralized SGD to reach the same training loss"* — MATCHA (NeurIPS 2019).

4. **Graph sparsification of the communication topology reduces energy and bandwidth costs while maintaining model quality.**
   > *"The proposed solution can lower the energy consumption at the busiest node by 54%-76% while maintaining the quality of the trained model"* — Energy-efficient Decentralized Learning via Graph Sparsification (ICASSP 2024).

5. **Heterogeneity (compute, network, data) is the dominant bottleneck in decentralized training.**
   > *"Inherent heterogeneity in agents' resources (computation, communication, and task size) may lead to substantial variations in training time"* — ComDML (ICDCS 2024).

6. **A single global merge at the end of decentralized training can match parallel SGD convergence.**
   > *"Fully connected communication at the final step, implemented by a single global merging, can significantly improve the performance of decentralized learning under high data heterogeneity"* — Zhu et al., ICLR 2026.

7. **Secure aggregation and sparsification can be combined with only ~11% overhead.**
   > *"With random subsampling, CESAR is always within 0.5% accuracy of D-PSGD, while adding only 11% of data overhead"* — CESAR (arXiv 2024).

8. **Adaptive compression (GraVAC) yields 1.95×–6.67× end-to-end training speedups.**
   > *"GraVAC reduces end-to-end training time for ResNet101, VGG16 and LSTM by 4.32x, 1.95x and 6.67x respectively"* — GraVAC (IEEE CLOUD 2023).

9. **Foundation-model training in heterogeneous geo-distributed settings is viable with careful scheduling.**
   > *"Across 8 different cities spanning 3 continents, our approach is 4.8X faster than prior state-of-the-art training systems (Megatron)"* — Yuan et al. (arXiv 2022).

10. **Decentralized learning does not automatically provide privacy advantages over FL; it can increase attack surface.**
    > *"Decentralized learning does not offer any security advantage over federated learning. Rather, it increases the attack surface"* — Pasquini et al., IEEE S&P 2023.

---

## Research Gaps

- Most decentralized methods assume static network topologies; dynamic/intermittent connectivity remains understudied in practice
- Adaptive and online compression (e.g., GraVAC) is newer and needs broader benchmarks across model scales
- Joint optimization of bandwidth + energy + accuracy is still an emerging area
- Secure aggregation + compression (e.g., CESAR) has only been evaluated at modest scales (< 100 nodes)

---

## Related Topics

- [Federated Learning (section index)](README.md)
- `topics/quantization/Multimodal-Quantization/` — gradient quantization methods overlap with compression
- `topics/training/fsdp2-vescale/` — distributed training infrastructure
