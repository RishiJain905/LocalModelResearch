# NPU-Native Runtimes (OpenVINO & Qualcomm AI Stack)

> **OpenVINO GitHub:** [openvinotoolkit/openvino](https://github.com/openvinotoolkit/openvino) · [openvinotoolkit/openvino.genai](https://github.com/openvinotoolkit/openvino.genai)  
> **Qualcomm:** [docs.qualcomm.com/bundle/publicresource/topics/80-62010-1/ai-overview.html](https://docs.qualcomm.com/bundle/publicresource/topics/80-62010-1/ai-overview.html)  
> **Papers:** [NITRO arXiv:2412.11053v1](https://arxiv.org/html/2412.11053v1) · [Benchmarking Edge AI arXiv:2409.14803v1](https://arxiv.org/html/2409.14803v1)

---

## Overview

NPU-native runtimes target **edge inference** on dedicated Neural Processing Units — Intel NPUs (via OpenVINO), Qualcomm Hexagon NPUs (via QNN), and other edge accelerators. They prioritize **energy efficiency** and **TTFT** over raw throughput.

---

## OpenVINO (Intel)

> *"OpenVINO toolkit optimizes neural network inference on Intel hardware — CPUs, GPUs, and NPUs."* — [Intel](https://github.com/openvinotoolkit/openvino)

### Key Repositories

| Repo | Description |
|------|-------------|
| [openvinotoolkit/openvino](https://github.com/openvinotoolkit/openvino) | Core OpenVINO runtime |
| [openvinotoolkit/openvino.genai](https://github.com/openvinotoolkit/openvino.genai) | GenAI-optimized runtime with LLM support |
| [openvinotoolkit/model_server](https://github.com/openvinotoolkit/model_server) | Production inference server |
| [vllm-project/vllm-openvino](https://github.com/vllm-project/vllm-openvino) | vLLM OpenVINO backend |

### NPU Workflow

```python
# Python GenAI API
from openvino_genai import LLMPipeline
pipe = LLMPipeline(model_path, "NPU")
pipe.generate("prompt", max_new_tokens=256)
```

```bash
# Via Optimum Intel
optimum-cli export openvino -m meta-llama/Llama-3.1-8B --weight-format int4 --awq
```

### Quantization Support

| Format | Description |
|--------|-------------|
| **INT4** | NNCF data-aware weight compression (2025.3+) |
| **INT8** | Full INT8 via NNCF |
| **FP16** | Default precision |

### Supported Models (OpenVINO)

> *"LLMs: LLaMA (2, 3, 3.1, 3.2), Qwen (1.5, 2, 2.5, 3), ChatGLM (2, 3, 4), Mistral, Gemma, Phi, DeepSeek, BLOOM, Snowflake Arctic, Command R, DBRX, Qwen2MoE"* — [OpenVINO Docs](https://openvinotoolkit.github.io/openvino.genai/docs/supported-models/)

HuggingFace collections: [OpenVINO/llms-optimized-for-npu](https://huggingface.co/collections/OpenVINO/llms-optimized-for-npu)

### Known Issues

> *"LNL (Lunar Lake) NPU can be slower than CPU/GPU for LLM inference."* — [OpenVINO GitHub issues #1563, #1882](https://github.com/openvinotoolkit/openvino)

- NPU LLM performance is **variable** — sometimes slower than CPU
- Prefill phase is compute-bound (NPU limited)
- Decode phase is DRAM bandwidth-bound

---

## Qualcomm AI Stack

> *"Qualcomm AI Stack provides a unified SDK for deploying AI models on Snapdragon and Qualcomm platforms."* — [Qualcomm Docs](https://docs.qualcomm.com/bundle/publicresource/topics/80-62010-1/ai-overview.html)

### Key Components

| Component | Description |
|-----------|-------------|
| **QNN (Qualcomm Neural Network)** | Low-level SDK for Hexagon NPU |
| **QAIRT** | Simplified AI runtime for deployment |
| **LiteRT QNN** | Google TensorFlow Lite integration |
| **Model HQ** | LLM model optimization for Snapdragon X |

### Hexagon NPU Workflow

- QNN SDK provides low-level access to Hexagon Tensor Processor (HTP)
- GGUF models supported via QAIRT SDK on HTP backend
- LiteRT QNN Delegate for TensorFlow Lite integration

### Quantization Support (Qualcomm)

| Precision | Description |
|-----------|-------------|
| **INT4** | BitNet-style 4-bit on Hexagon |
| **INT8** | Standard 8-bit |
| **FP16** | Full precision |

---

## Cross-Platform NPU Performance

> *"NVIDIA RTX 4050 delivers ~19× higher throughput than dedicated edge NPUs for LLM inference."* — [arXiv:2409.14803v1](https://arxiv.org/html/2409.14803v1)

### Edge NPU Benchmarks

| Platform | Throughput | Power | mJ/token |
|----------|-----------|-------|----------|
| **NVIDIA RTX 4050 (Laptop)** | 131.70 tok/s | 34.1W | 297.3 |
| **Hailo-10H NPU (RPi 5)** | 6.91 tok/s | 1.87W | 270.5 |
| **iPhone 16 Pro (Hot)** | 22.56 tok/s | ~10W est | N/A |
| **Samsung S24 Ultra** | 9.93 tok/s | ~12W est | N/A |

### NPU vs GPU: Thermal Constraints

> *"Mobile SoC GPUs suffer from thermal throttling that degrades performance significantly under sustained load."* — [arXiv:2409.14803v1](https://arxiv.org/html/2409.14803v1)

| Platform | Sustained Degradation |
|----------|---------------------|
| Apple Silicon (Metal) | ~44% throughput loss after 2 iterations |
| iPhone/Android GPU | Significant thermal throttling after 5-10 queries/hour |

---

## When to Use Edge NPUs

| Use Case | Recommendation |
|----------|---------------|
| **Interactive/real-time (TTFT critical)** | ✅ Edge NPUs — dramatically faster TTFT |
| **Battery-powered/energy-constrained** | ✅ Edge NPUs — superior energy proportionality |
| **Sustained agent workloads (>20 inf/hour)** | ✅ Edge NPUs — stable thermal profile |
| **Privacy-sensitive/offline** | ✅ Edge NPUs — no data leaves device |
| **Large models beyond edge capacity** | ❌ Use cloud |
| **Infrequent complex queries** | ❌ Use cloud |

---

## Latency vs Cloud

> *"Edge dramatically outperforms cloud for Time to First Token — huge margin."* — [Edge AI benchmarks](https://arxiv.org/html/2409.14803v1)

| Scenario | Edge Advantage |
|----------|--------------|
| **TTFT** | Edge wins by large margin |
| **Cost (Jetson Nano)** | $0.0017/response vs cloud ~970× more expensive |
| **95th-percentile latency** | Edge-cloud hybrid reduces by 53-61% |

---

## GitHub Repos

| Repo | Description |
|------|-------------|
| [openvinotoolkit/openvino](https://github.com/openvinotoolkit/openvino) | Core OpenVINO runtime |
| [openvinotoolkit/openvino.genai](https://github.com/openvinotoolkit/openvino.genai) | GenAI LLM-optimized runtime |
| [openvinotoolkit/model_server](https://github.com/openvinotoolkit/model_server) | Production inference server |
| [vllm-project/vllm-openvino](https://github.com/vllm-project/vllm-openvino) | vLLM Intel NPU backend |

---

## Papers & Technical Reports

| Paper | URL | Key Contribution |
|-------|-----|-----------------|
| **NITRO: LLM Inference on Intel Laptop NPUs** | [arXiv:2412.11053v1](https://arxiv.org/html/2412.11053v1) | Intel NPU LLM optimization |
| **Benchmarking Edge AI Platforms** | [arXiv:2409.14803v1](https://arxiv.org/html/2409.14803v1) | Cross-platform edge comparison |
| **Scaling LLM Test-Time Compute with Mobile NPU** | [arXiv:2509.23324v1](https://arxiv.org/html/2509.23324v1) | Mobile NPU utilization |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Best for** | Edge deployment, energy-constrained, privacy-sensitive, real-time TTFT |
| **OpenVINO** | Intel NPU/GPU/CPU, GenAI API, INT4/INT8 quantization |
| **Qualcomm** | Snapdragon/X Hexagon NPU, QNN SDK, QAIRT runtime |
| **Key advantage** | Energy efficiency, TTFT, offline operation |
| **Key disadvantage** | Lower absolute throughput vs GPU; NPU software stack maturity issues |

---

*Last updated: 2026-04-21*
