# Carbon Tracking and Carbon-Aware Computing for ML Training

> **Note:** This document is part of the [k8s-elastic-training](../k8s-elastic-training.md) infrastructure series under [green-ai-thermals](../README.md).

## Table of Contents

1. [Overview](#overview)
2. [Carbon Footprint of LLM Training](#carbon-footprint-of-llm-training)
3. [Carbon Intensity APIs](#carbon-intensity-apis)
4. [Carbon-Aware Scheduling on Kubernetes](#carbon-aware-scheduling-on-kubernetes)
5. [Carbon Tracking Tools: CodeCarbon](#carbon-tracking-tools-codecarbon)
6. [ML CO2 Impact Calculator](#ml-co2-impact-calculator)
7. [Experiment Tracking with Carbon Logs](#experiment-tracking-with-carbon-logs)
8. [Green GPU Cloud Providers](#green-gpu-cloud-providers)
9. [Carbon Offsets for GPU Clusters](#carbon-offsets-for-gpu-clusters)
10. [Key Papers and Research](#key-papers-and-research)

---

## Overview

Carbon-aware computing for ML training integrates real-time carbon intensity data into workload scheduling decisions, enabling organizations to reduce emissions without sacrificing performance. This approach is particularly relevant for GPU clusters running ML workloads, where energy consumption—and thus carbon emissions—can be substantial.

Key drivers:
- **Regulatory pressure**: EU Energy Efficiency Act mandates 100% renewable electricity for German data centers by 2027
- **Cost optimization**: Carbon-aware scheduling often aligns with lower electricity prices during off-peak renewable energy periods
- **Research transparency**: Growing community expectation that ML papers report carbon emissions

---

## Carbon Footprint of LLM Training

### Key Findings

| Model | Parameters | Energy (MWh) | CO₂eq (tonnes) | Equivalent |
|-------|-----------|--------------|----------------|------------|
| GPT-3 | 175B | 1,287 | ~502 | 112 cars/year |
| BERT | 340M | ~0.0006 | ~1,400 lbs CO₂ | Trans-American flight |
| BLOOM | 176B | ~433 | ~230 | — |
| Llama 2 | 70B | 1,273 | ~539 | — |
| OPT-175B | 175B | ~324 | ~90 | — |

> **Source:** [The Carbon Footprint of Large Language Models | Cutter Consortium](https://www.cutter.com/article/large-language-models-whats-environmental-impact)
>
> "Figures for the training of Llama 2 are similar: 1,273,000 kWh, with 539 tonnes of CO2e. Analysis of the open source model BLOOM suggests that..."

### GPU Energy Consumption

| GPU | TDP (W) | Annual kWh (24/7) | Notes |
|-----|---------|-------------------|-------|
| NVIDIA A100 | 400W | 3,504 kWh | Common for ML training |
| NVIDIA H100 | 350-700W | 3,066-6,132 kWh | 700W = German household |
| NVIDIA H100 (cluster of 3.5M) | 700W × 3.5M | 21,420 GWh | Equivalent to entire nations |

> **Source:** [Massed Compute FAQ](https://massedcompute.com/faq-answers/?question=What%20are%20the%20estimated%20carbon%20emissions%20of%20NVIDIA%27s%20A100%20and%20H100%20GPUs%20in%20a%20typical%20data%20center%20per%20year); [Kaggle: Nvidia H100 GPUs uses more electricity than some countries](https://www.kaggle.com/general/494791)

### Google's Research: Training Carbon Will Plateau and Shrink

Google researchers (including David Patterson) found that four best practices can reduce ML training energy by up to **100x** and CO₂ emissions up to **1000x**:

1. **Model efficiency** (Primer, etc.)
2. **Hardware upgrades** (TPUv4 vs P100)
3. **Data center efficiency** (PUE improvements)
4. **Location selection** (cleaner energy grids)

> **Source:** [The Carbon Footprint of Machine Learning Training Will Plateau, Then Shrink | IEEE Computer](https://research.google/blog/good-news-about-the-carbon-footprint-of-machine-learning-training/)

---

## Carbon Intensity APIs

### Electricity Maps

Real-time, historical, and forecasted carbon-intensity data for electricity grids worldwide.

**Key Features:**
- Grid-specific carbon intensity (gCO2eq/kWh)
- Power mix breakdown by technology
- 1-hour temporal granularity
- Free tier available

**API Endpoint Example:**
```
GET /v3/carbon-intensity
Headers: auth-token: my-api-token
```

> **Source:** [Electricity Maps API Docs](https://app.electricitymaps.com/developer-hub/api/getting-started)
>
> "The API key should be included as a header on the request: `auth-token: my-api-token`. To calculate the **carbon intensity**, we match the flow-traced electricity mix with technology-specific emission factors."

### WattTime

Open-source-based API providing real-time marginal emissions data for power grids.

**Key Features:**
- Marginal Operating Emissions Rate (MOER) in lbs CO2/MWh
- 5-minute and hourly granularity
- Location-based mapping
- Free tier available

> **Source:** [WattTime API Documentation](https://legacy-docs.watttime.org/#:~:text=The%20WattTime%20API%20provides%20access,e.g.%20CO2%20lbs/MWh)
>
> "Provides a real-time signal indicating the marginal carbon intensity for the local grid for the current time (updated every 5 minutes)."

### Green Software Foundation: API-Based Carbon Intensity

> **Source:** [SCI Guidance - API Based](https://sci-guide.greensoftware.foundation/I/APIBased/)
>
> "Providers like WattTime API and Electricity Maps provide an API to get the carbon intensity based on location parameters."

| Provider | Data Type | Granularity | Free Tier |
|----------|-----------|-------------|-----------|
| WattTime | Marginal emissions | 5-min, hourly | Yes |
| Electricity Maps | Carbon intensity, power mix | 1-hour | Yes (limited) |
| Carbon Aware API (Azure) | Grid carbon intensity | Hourly | Via Azure |

---

## Carbon-Aware Scheduling on Kubernetes

### Compute Gardener

Open-source Kubernetes scheduler for carbon-aware ML workload scheduling.

> **Source:** [Compute Gardener](https://www.compute-gardener.com/)
>
> "Automatically delay deferrable training jobs (batch retraining, experimental iterations, overnight pipelines) until grid carbon intensity is lower, without any code changes."

**Features:**
- Real-time carbon intensity integration
- Hardware power profiling (GPU inference, training, rendering)
- GPU workload classification
- Energy budgets
- Free open-source tier; Advanced Signals API for real-time optimization

**Key Quote:**
> "At high carbon intensity times (300+ gCO2eq/kWh) versus low intensity times (~100 gCO2eq/kWh), the same training job can produce 3x more carbon emissions simply due to when it runs."

### kube-green

Kubernetes operator that automatically shuts down resources during off-hours to reduce CO2 footprint.

> **Source:** [kube-green GitHub](https://github.com/kube-green/kube-green)
>
> "kube-green is a simple k8s addon that automatically shuts down (some of) your resources when you don't need them."

**Features:**
- Automatic resource shutdown (pods, CronJobs, deployments)
- Timezone support (IANA identifiers)
- Exclusion support (e.g., exclude api-gateway deployments)
- Auto-restore or leave suspended

### Carbon-Aware Kubernetes (Green Software Foundation)

Extends Kubernetes Scheduler with carbon intensity data from WattTime API.

> **Source:** [Carbon-Aware Kubernetes | Green Software Foundation](https://greensoftware.foundation/articles/carbon-aware-kubernetes/)
>
> "Kubernetes is built in a way that it can make carbon-aware decisions balanced against the technical requirements of the system."

**Architecture:**
1. Kubernetes Scheduler assigns Pods to Nodes
2. Carbon Intensity Data from WattTime (MOER values)
3. Weighting Algorithm calculates normalized percentages

**US Regional Comparison:**

| Region | Cost/hour (5 D4as v4) | lbs CO2/MWh |
|--------|----------------------|-------------|
| East US | $0.96 | 1585 |
| West US | $1.12 | 865 |

> "Small premium (~17%) to cut carbon emissions roughly in half"

### Zemdom Carbon-Aware Scheduling

Custom Kubernetes scheduler that minimizes cluster carbon emissions by assigning tasks to most energy-efficient bare-metal servers.

> **Source:** [zemdom/carbon-aware-scheduling GitHub](https://github.com/zemdom/carbon-aware-scheduling)
>
> "Carbon-aware Kubernetes scheduler, which minimizes cluster carbon emissions by assigning computational tasks to most energy-efficient bare-metal servers."

### Research Paper: Carbon Emission-Aware Job Scheduling

> **Source:** [Carbon emission-aware job scheduling for Kubernetes deployments | Springer](https://link.springer.com/article/10.1007/s11227-023-05506-7)
>
> "In this paper, we overcome this deficit by proposing a novel practical CO2-aware workload scheduling algorithm implemented in the Kubernetes orchestrator to shift non-critical jobs in time."

### ARXIV Survey: Carbon-Aware Container Orchestration

> **Source:** [A Survey on Task Scheduling in Carbon-Aware Container Orchestration | arXiv:2508.05949](https://arxiv.org/html/2508.05949v1)
>
> "This paper proposes a cloud-based scheduling framework that integrates real-time carbon intensity data to optimize the execution of energy intensive tasks in cloud data centers."

---

## Carbon Tracking Tools: CodeCarbon

Open-source Python library for tracking carbon emissions from ML training.

> **Source:** [mlco2/codecarbon GitHub](https://github.com/mlco2/codecarbon)
>
> "Track emissions from compute and recommend ways to reduce their impact on the environment."

### Installation

```bash
pip install codecarbon
```

### Quick Start

```python
from codecarbon import EmissionsTracker

tracker = EmissionsTracker()
tracker.start()

# Your ML training code here

emissions = tracker.stop()
print(f"Emissions: {emissions} kg CO₂")
```

### CLI Usage

```bash
codecarbon monitor --no-api -- python train.py
```

### How It Works

1. Estimates hardware electricity power consumption (GPU + CPU + RAM)
2. Applies carbon intensity of the region where computing occurs
3. Outputs emissions in kg CO₂

### Project Statistics

| Metric | Value |
|--------|-------|
| Stars | 1.8k |
| Forks | 266 |
| Releases | 65+ |
| License | MIT |

### Visualization

- **Dashboard:** [dashboard.codecarbon.io](https://dashboard.codecarbon.io/)
- **Local:** Use `carbonboard`

### Citation

```bibtex
@software{benoit_courty_2024_11171501,
  author = {Benoit Courty, Victor Schmidt, Sasha Luccioni, and many others},
  title = {mlco2/codecarbon: v2.4.1},
  year = {2024},
  publisher = {Zenodo},
  doi = {10.5281/zenodo.11171501}
}
```

---

## ML CO2 Impact Calculator

Web-based tool for estimating carbon footprint of ML training.

> **Source:** [ML CO2 Impact Calculator](https://mlco2.github.io/impact/)

**Purpose:**
- Calculate GPU carbon emissions
- Publish results in research papers/blog posts
- Integrate tracking via CodeCarbon package

### Key Metrics

| Metric | Description |
|--------|-------------|
| Raw Emissions | Carbon produced based on local power grid |
| Offset Emissions | Carbon offset by cloud provider |

> **Note:** Estimates do **not** include datacenter PUE. Multiply by PUE to account for cooling/overhead.

### Recommendations

1. Choose Cloud Provider with clean energy commitments
2. Select Region strategically (different grids, different carbon intensity)
3. Buy Carbon Offsets
4. Avoid Grid Search (major source of ML emissions)
5. Choose Clean Energy Sources
6. Publish Emissions Data in papers

> **Source:** [ML CO2 Impact Calculator - Key Concepts](https://mlco2.github.io/impact/)

---

## Experiment Tracking with Carbon Logs

### Weights & Biases + CodeCarbon

> **Source:** [Tracking CO2 Emissions with CodeCarbon and W&B](https://wandb.ai/amanarora/codecarbon/reports/Tracking-CO2-Emissions-of-Your-Deep-Learning-Models-with-CodeCarbon-and-Weights-Biases--VmlldzoxMzM1NDg3)
>
> "In this report, we showcase how to use codecarbon and W&B to track the CO2 emission of your computing resources."

### MLflow + CodeCarbon Integration

> **Source:** [dataroots: Tracking Carbon Emission During MLOps](https://dataroots.io/blog/tracking-carbon-emission-during-mlops)
>
> "mlflow-emissions-sdk: Integrating CodeCarbon into MLFlow"

---

## Green GPU Cloud Providers

### Gensyn (Protocol Labs)

Decentralized ML compute network connecting distributed computing resources.

> **Source:** [Meet Gensyn: The Machine Learning Compute Network | Protocol Labs](https://www.protocol.ai/blog/meet-gensyn-the-machine-learning-compute-network/)

**Key Value Proposition:**
- Connects data centers, personal laptops, smartphones into virtual cluster
- Peer-to-peer, on-demand access
- Reduces costs for developers
- Open network for open-source AI builders

**Quote:**
> "Machine learning is revolutionizing the way we work and the way we live, but the biggest roadblock to continuing this revolution is access to compute power."
> — Harry Grieve, Co-founder

**Funding:** $50M+ (Series A led by a16z, June 2023)

### NexGen Cloud

> **Source:** [NexGen Cloud Sustainability](https://www.nexgencloud.com/sustainability)
>
> "We integrate the world's most energy-efficient GPU hardware with 100% green energy while also emphasising data sovereignty and security."

### Lambda (Hydrogen-Powered GPU Cluster)

> **Source:** [Lambda fires up hydrogen-powered Nvidia GPU cluster | The Register](https://www.theregister.com/2025/09/24/lambda_nvidia_hydrogen/)
>
> "Lambda says its latest Nvidia GB300 NVL72 system is not only powered entirely by hydrogen fuel cells but doesn't consume a single ounce of water."

### Flux Core Data Systems

> **Source:** [Flux Core Data Systems](https://fluxcoredatasystems.com/sustainable-cloud-computing-solutions/)
>
> "Flux Core Data Systems delivers secure, high-performance cloud computing solutions and cloud hosting services powered entirely by renewable energy."

**Features:**
- Modular, renewable-powered data centers
- Deploy in 90 days
- Solar-powered AI infrastructure
- Independent from traditional grid

### Aethir

> **Source:** [Aethir Green Compute](https://ecosystem.aethir.com/blog-posts/green-compute-building-energy-efficient-strategic-gpu-reserves-for-sustainable-ai)
>
> "Aethir's decentralized GPU cloud is leveraging energy-efficient compute infrastructure, powered by the ATH Strategic Compute Reserve, to support sustainable AI evolution."

### Major Cloud Providers

| Provider | Commitment | Target Date |
|----------|------------|-------------|
| Google | 24/7 Carbon-Free Energy | 2030 |
| Microsoft Azure | Carbon Negative | 2030 |
| AWS | 100% Renewable Energy | 2025 |

> **Source:** [Google 24/7 Carbon-Free Energy](https://sustainability.google/reports/247-carbon-free-energy/); [Tech Magazine: Top 10 Green Cloud](https://technologymagazine.com/top10/top-10-green-cloud-computing-companies)

---

## Carbon Offsets for GPU Clusters

### Renewable Energy Certificates (RECs)

> **Source:** [Reuters: Sustainable Data Centers](https://www.reuters.com/legal/legalindustry/sustainable-data-centers-renewable-energy-tool-reduce-carbon-footprint--pracin-2026-04-09/)
>
> "RECs, unlike carbon offsets (another sustainability tool that data center owners/operators may use), are tied directly to electricity"

**Key Difference:**
- **RECs**: Tied directly to electricity, address Scope 2 emissions
- **Carbon Offsets**: Represent 1 ton CO2e avoided/reduced, can apply to Scopes 1-3

### Carbon Offsetting for Data Centers

> **Source:** [EPA: RECs and Carbon Offsets](https://www.gba.org/resources/green-building-methods/energy-solutions/recs-and-carbon-offsets/)
>
> "A carbon offset is a tradable credit representing a metric ton of CO2e emissions avoided or reduced."

### Germany's Renewable Energy Requirements

> **Source:** [Ecohz: Renewable Energy in Data Centers](https://www.ecohz.com/sustainability-solutions/data-centers)
>
> "Germany adopted at the end of 2023 a new Energy Efficiency Act, mandating all German data centre operators to cover 50% of their electricity consumption with unsubsidised renewable electricity as of January 1, 2024, and 100% as of January 1, 2027."

### Microsoft Carbon Removal Agreements

Microsoft has signed agreements for:
- 1 million tons carbon credits over 10 years with Hafslund Celsio
- 4.8 million nature-based carbon removal credits with Anew Climate

---

## Key Papers and Research

### 1. Patterson et al. - Carbon Footprint of ML Training

> **Source:** [Stanford PDF: Carbon Footprint of Machine Learning](https://ees2.slac.stanford.edu/sites/default/files/2023-12/10%20-%20Patterson.pdf)
>
> "Four best practices can reduce ML training energy by up to 100x and CO2 emissions up to 1000x."

### 2. Lottick et al. - Energy and Carbon Footprints of ML

> **Source:** [ML CO2 Impact Paper | arXiv:1910.09700](https://arxiv.org/abs/1910.09700)
>
> "Published at Climate Change AI Workshop, NeurIPS 2019"

### 3. LLM Training Carbon Studies

| Study | Key Finding |
|-------|-------------|
| GPT-3 (Brown et al.) | 1,287 MWh, 502 tonnes CO2 |
| BERT (Devlin et al.) | ~1,400 lbs CO2 |
| Meituan (2025) | 40x GPT-3 energy for larger models |

### 4. Google Research: Carbon Efficiency

> **Source:** [Google: Good News About Machine Learning Training Carbon Footprint](https://research.google/blog/good-news-about-the-carbon-footprint-of-machine-learning-training/)
>
> "In 'The Carbon Footprint of Machine Learning Training Will Plateau, Then Shrink', accepted for publication in IEEE Computer, we focus on operational carbon emissions..."

### 5. Carbon-Aware Scheduling Research

> **Source:** [A Survey on Task Scheduling in Carbon-Aware Container Orchestration | arXiv:2508.05949](https://arxiv.org/html/2508.05949v1)
>
> "This paper proposes a cloud-based scheduling framework that integrates real-time carbon intensity data to optimize the execution of energy intensive tasks in cloud data centers."

---

## Summary: Getting Started

### Quick Wins

1. **Track emissions**: Add `codecarbon` to training scripts
2. **Choose regions**: US-West (CAISO) has ~50% lower carbon intensity than US-East (PJM)
3. **Schedule wisely**: Use Compute Gardener or carbon-aware Kubernetes scheduling
4. **Report**: Include emissions in papers (template provided in ML CO2 Impact Calculator)

### Tools Comparison

| Tool | Purpose | Language | License |
|------|---------|----------|---------|
| CodeCarbon | Emission tracking | Python | MIT |
| Compute Gardener | K8s scheduling | Go | Open Source |
| kube-green | K8s resource cleanup | Go | MIT |
| Electricity Maps API | Carbon intensity data | REST API | Commercial |
| WattTime API | Marginal emissions | REST API | Freemium |

### Further Reading

- [Green Software Foundation](https://greensoftware.foundation/)
- [Climate Change AI](https://www.climatechange.ai/)
- [CodeCarbon Documentation](https://docs.codecarbon.io/)
- [Compute Gardener](https://www.compute-gardener.com/)

---

*Last updated: May 2026*