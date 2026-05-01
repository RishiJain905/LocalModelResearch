# Dynamic Frequency Scaling (DFS) and Undervolting for Energy-Efficient ML Training

## Table of Contents
1. [Overview](#overview)
2. [GPU DVFS Fundamentals](#gpu-dvfs-fundamentals)
3. [NVIDIA GPU Boost & ClockBoost](#nvidia-gpu-boost--clockboost)
4. [AMD P-State Driver](#amd-p-state-driver)
5. [Energy-Efficient Training Research](#energy-efficient-training-research)
6. [PyTorch Energy Profiling Tools](#pytorch-energy-profiling-tools)
7. [CUDA Power Management](#cuda-power-management)
8. [NVIDIA Data Center GPU Manager (DCGM)](#nvidia-data-center-gpu-manager-dcgm)
9. [Cool/Binned vs Performance Chips](#coolbinned-vs-performance-chips)
10. [Undervolting Stability Testing](#undervolting-stability-testing)
11. [Key References](#key-references)

---

## Overview

Dynamic Frequency Scaling (DFS) and undervolting are critical techniques for reducing energy consumption in ML training workloads. By adjusting voltage and frequency based on workload demands, practitioners can achieve significant energy savings with minimal performance degradation.

**Key Statistics:**
- GPU DVFS can achieve **8.7%–23.1%** energy savings for DNN training depending on GPU architecture
- Kernel-level DVFS for LLM training achieves up to **14.6% energy savings** with only **0.6% performance loss**
- NVIDIA B200 power profiles deliver up to **15% energy savings** with at most **3% performance loss**

---

## GPU DVFS Fundamentals

### What is DVFS?

Dynamic Voltage and Frequency Scaling (DVFS) adjusts both core clock frequency and voltage based on workload demands. The key relationship:

> **Power ∝ C × V² × f**

Where C is capacitance, V is voltage, and f is frequency. Since power scales with voltage squared, reducing voltage provides disproportionate energy savings.

### GPU-Specific Considerations

GPUs have **two frequency domains**:
- **Core frequency (f_core)**: Controls ALU cores and on-chip components
- **Memory frequency (f_mem)**: Controls SDRAM modules

> "Different GPU applications have varying utilization of cores and SDRAM. Raising frequency of low-utilization components may not improve performance but increases power consumption." — Tang et al., arXiv:1905.11012

### Energy Efficiency Sweet Spots

Research shows default frequency settings are typically NOT optimal for energy efficiency:

| GPU | Architecture | Training Energy Savings | Inference Energy Savings |
|-----|-------------|------------------------|------------------------|
| Tesla P100 | Pascal | 23.1% | 26.4% |
| Tesla V100 | Volta | 14.5% | 22.3% |
| GTX 2080Ti | Turing | 8.7% | 19.6% |

**Source:** [The Impact of GPU DVFS on Energy and Performance of Deep Learning (arXiv:1905.11012)](https://arxiv.org/pdf/1905.11012)

---

## NVIDIA GPU Boost & ClockBoost

### GPU Boost Technology

NVIDIA GPU Boost™ automatically increases GPU clock rates when sufficient power and thermal headroom are available.

### GPU Boost 3.0 (Pascal and later)

GPU Boost 3.0 exposes **individual points of the GPU voltage-frequency curve** to end users, enabling fine-grained control:

> "The main feature of GPU Boost 3.0 is that NVIDIA exposes the individual points of the GPU voltage-frequency curve to the end-user. That means enthusiasts can finetune every single point on the voltage frequency curve. That allows for greater finetuning and avoids the lost opportunity overclocking range with a fixed offset across the entire voltage-frequency curve." — [SkatterBencher](https://skatterbencher.com/nvidia-gpu-boost-3-0/)

**Key Features:**
1. Per Voltage Point Frequency Offset
2. Voltage Point Lock via NVAPI
3. Overvolting expressed as a percentage

> "With GPU Boost 3.0, there are no less than **eighty (80!)** individually adjustable points on Pascal GPUs."

### Application Clocks vs Default Boost

For Tesla GPUs, GPU Boost allows users to leverage available power headroom to select higher graphics clocks:

```bash
# Enable persistence mode (required for clock management)
nvidia-smi -pm 1

# Query available application clock rates
nvidia-smi -q -d CLOCK

# Set specific application clocks
nvidia-smi -ac 877,1531

# Disable auto-boost to maintain fixed clocks
nvidia-smi -ac 877,1531 -pl 250
```

**Source:** [CUDA Pro Tip: Increase Application Performance with NVIDIA GPU Boost](https://developer.nvidia.com/blog/cuda-pro-tip-increase-application-performance-nvidia-gpu-boost/)

### nvidia-smi Clock Management Commands

| Command | Description |
|---------|-------------|
| `nvidia-smi -pm 1` | Enable persistence mode |
| `nvidia-smi -q -d CLOCK` | Query available clock rates |
| `nvidia-smi -ac 877,1531` | Set application clocks (memory, graphics) |
| `nvidia-smi -auto-boost-default=0` | Disable auto-boost |
| `nvidia-smi -rgc` | Reset GPU to default clocks |

---

## AMD P-State Driver

### Overview

`amd-pstate` is AMD's CPU performance scaling driver based on **Collaborative Processor Performance Control (CPPC)**, providing finer grain frequency management than legacy ACPI hardware P-States.

> "ACPI P-states switch only between 3 states; CPPC allows flexible, low-latency interface for direct performance hints to hardware."

**Source:** [Linux Kernel Documentation - amd-pstate](https://docs.kernel.org/admin-guide/pm/amd-pstate.html)

### Driver Operation Modes

| Mode | Description |
|------|-------------|
| **Active** | Low-level firmware control via `amd_pstate_epp` driver; provides EPP hints (0x0 to 0xff) |
| **Passive** | Kernel software specifies desired QoS target; performance as percentage of nominal |
| **Guided** | Driver requests min/max performance levels; platform autonomously selects |

### CPPC Performance Registers

| Register | Purpose |
|----------|---------|
| Highest Performance | Absolute maximum (may not be sustainable) |
| Nominal Performance | Maximum sustained performance |
| Lowest Non-linear Performance | Lowest level where nonlinear power savings begin |
| Energy Performance Preference (EPP) | Hint to bias toward performance (0x0) or efficiency (0xff) |

### Preferred Core Feature

> "Not all cores can reach maximum frequency due to semiconductor process variation. AMD redefines 'maximum frequency' where only a fraction of cores can reach it."

This enables the scheduler to prefer cores achieving higher frequency with lower voltage, with rankings dynamically changing based on workload, thermals, and platform conditions.

### Configuration Example

```bash
# Check CPPC support
cat /sys/devices/system/cpu/amd_pstate/status

# Set operation mode
echo "active" > /sys/devices/system/cpu/amd_pstate/status

# Set energy efficiency preference (0x0 = performance, 0xff = efficiency)
echo "ff" > /sys/devices/system/cpu/cpufreq/policy0/energy_performance_preference
```

---

## Energy-Efficient Training Research

### Empirical Study: GPU DVFS Impact on Deep Learning

**Paper:** "The Impact of GPU DVFS on Energy and Performance of Deep Learning" (arXiv:1905.11012)

**Key Findings:**
- Optimal core frequency can improve DNN performance by **up to 33%** and conserve up to **23.1%** training energy
- Default frequencies are usually NOT at the energy efficiency optimum
- Energy scaling curves show a **valley trend** with increasing core frequency

> "For V100, AlexNet performance improves ~83% from f_core=510 MHz to f_core=945 MHz, while power only increases 66%. This creates energy valley at f_core=945 MHz. However, when f_core ≥1000 MHz, power has a big jump requiring higher voltage, and energy rises again."

**Neural Network Results:**
| Network | Training Energy Savings | Inference Energy Savings |
|---------|----------------------|-------------------------|
| AlexNet | 7.7%–25.7% | 17.9%–28.7% |
| VggNet-16 | 9.2%–19.1% | 18.9%–26.9% |
| GoogleNet | 2.3%–24.3% | 10.9%–28.2% |
| ResNet-50 | 3.2%–25.3% | 19.5%–24.7% |

**Source:** [arXiv:1905.11012](https://arxiv.org/pdf/1905.11012)

### Kernel-Level DVFS for LLM Training

**Paper:** "Reducing Compute Waste in LLMs through Kernel-Level DVFS" (arXiv:2601.08539v1)

This paper presents the **first kernel-level application of DVFS** for GPU-based LLMs:

> "Pass-level DVFS reduces energy consumption by about **2% without performance loss**, while our **kernel-level approach achieves up to 14.6% energy savings with only a 0.6% slowdown**."

**Key Insight:**
> "DVFS can lower the GPU energy consumption of LLM training significantly with, when optimizing for waste-reduction, almost no performance loss."

**Source:** [arXiv:2601.08539v1](https://arxiv.org/html/2601.08539v1)

### DVFS-GPT: DVFS for Large Language Models

**Paper:** "DVFS-GPT: Dynamic Voltage and Frequency Scaling for Energy-Efficient Large Language Models" (2025)

Key contributions:
1. Novel relaxation enabling gradient-based control over discrete VF pairs
2. Theoretical regret bounds matching information-theoretic limits
3. Practical implementation demonstrating **32% energy reduction** on real GPUs
4. Extensions to multi-GPU deployments and robust control under clock variability

**Source:** [IRJMETS Paper](https://www.irjmets.com/upload_newfiles/irjmets70700042545/paper_file/irjmets70700042545.pdf)

### SLO-Aware GPU DVFS for LLM Inference

Research on SLO-aware GPU DVFS for energy-efficient LLM inference serving adjusts GPU frequency configurations while meeting Service Level Objectives.

**Source:** [ResearchGate - SLO-Aware GPU DVFS](https://www.researchgate.net/publication/380946703_SLO-aware_GPU_DVFS_for_Energy-efficient_LLM_Inference_Serving)

---

## PyTorch Energy Profiling Tools

### Zeus - Deep Learning Energy Measurement and Optimization

**Zeus** is an open-source toolbox for measuring and optimizing deep learning energy consumption, now part of the PyTorch ecosystem.

> "Training a single 200B LLM on AWS p4d instances consumes ~11.9 GWh — enough to power 1,000+ average US households for a year."

**GitHub:** [ml-energy/zeus](https://github.com/ml-energy/zeus)

**Key Features:**
- **ZeusMonitor**: Measure time and energy within arbitrary execution windows
- **GlobalPowerLimitOptimizer**: Optimize GPU power limit for energy savings
- **PipelineFrequencyOptimizer (Perseus)**: Optimize pipeline parallelism for large model training

### ZeusMonitor Usage

```python
from zeus.monitor import ZeusMonitor

monitor = ZeusMonitor(gpu_indices=[0,1,2,3])

monitor.begin_window("training")
for e in range(100):
    monitor.begin_window("epoch")
    # training code...
    measurement = monitor.end_window("epoch")
monitor.end_window("training")
```

### Power Limit Optimization Strategies

```python
from zeus.optimizer.power_limit import (
    GlobalPowerLimitOptimizer,
    Time,         # Minimize time
    Energy,       # Minimize energy
    MaxSlowdownConstraint,  # Minimize energy with max slowdown
)

# Minimize energy while tolerating at most 10% slowdown
plo = GlobalPowerLimitOptimizer(monitor, MaxSlowdownConstraint(factor=1.1))
```

### Other Energy Profiling Tools

| Tool | Description | Source |
|------|-------------|--------|
| **CodeCarbon** | tracks carbon emissions from ML computations | [codecarbon.io](https://codecarbon.io/) |
| **Carbontracker** | monitors energy consumption of DL training | [GitHub](https://github.com/lfwa/carbontracker) |
| **PowerDL** | unified in-memory GPU energy profiling for PyTorch/TensorFlow | [ScienceDirect](https://www.sciencedirect.com/science/article/pii/S2352711026001202) |

---

## CUDA Power Management

### Persistence Mode

Persistence mode keeps the NVIDIA driver loaded even when no active processes are using the GPU, reducing initialization latency:

```bash
# Enable persistence mode
nvidia-smi -pm 1

# Disable persistence mode
nvidia-smi -pm 0
```

> "Persistence mode uses a few more watts per idle GPU, but prevents the fairly long delays that occur each time a GPU application is started." — [Reddit](https://www.reddit.com/r/nvidia/comments/lyh69k/linux_nvidiasmi_just_wondering_what_is_the_off/)

### Power Limit Control

```bash
# Query current power limit
nvidia-smi -q -d POWER

# Set power limit (in watts)
nvidia-smi -pl 250

# Query power usage
nvidia-smi --query-gpu=timestamp,power.draw --format=csv -l 1
```

### Compute Mode

| Mode | Description |
|------|-------------|
| Default (0) | Multiple compute applications allowed |
| Exclusive Thread (1) | Only one compute application allowed |
| Prohibited (2) | No compute applications allowed |
| Exclusive Process (3) | Multiple compute processes per GPU |

```bash
# Set compute mode
nvidia-smi -c 3  # Exclusive process mode
```

---

## NVIDIA Data Center GPU Manager (DCGM)

### Overview

DCGM provides GPU monitoring, diagnostics, and management for data center Tesla GPUs. It includes power management features and profiling capabilities.

**Source:** [NVIDIA DCGM Documentation](https://docs.nvidia.com/datacenter/dcgm/latest/user-guide/feature-overview.html)

### Key Power Management Features

| Feature | Description |
|---------|-------------|
| `DCGM_FI_DEV_POWER_COPY` | Power used by NVLink or PCIe |
| `DCGM_FI_DEV_POWER_USE` | GPU power usage |
| `DCGM_FI_DEV_POWER_STATE | Power budget state |
| `DCGM_FI_DEV_POWER_CAP` | Current power limit |
| `DCGM_FI_DEV_PERF_STATE` | Performance state (P0-P12) |

### DCGM Monitoring

```bash
# Monitor GPUs continuously
dcgmi dmon -s 60

# Run diagnostics
dcgmi diag -g 0 -r 1

# Set power limit via DCGM
dcgmi dmon -p 250
```

### GPU Clock Management via DCGM

```bash
# Query clock information
dcgmi dmon -c

# Set max clock limit
nvidia-smi -lgc 300,1530
```

---

## Cool/Binned vs Performance Chips

### GPU Binning Fundamentals

GPU chips are tested during manufacturing and binned into categories based on:
- Maximum achievable clock frequency
- Voltage characteristics
- Power efficiency
- Thermal characteristics

### Classification

| Type | Characteristics | Use Case |
|------|------------------|----------|
| **Performance/Binned Chips** | Higher clock speeds, higher voltage, higher power consumption | Maximum throughput workloads |
| **Cool/Energy-Efficient Chips** | Lower voltage requirements, better energy efficiency | Extended training runs, power-constrained facilities |

### Impact on Energy Efficiency

Research on Blackwell B200 power profiles shows:

> "Power savings of up to **15%** are achieved with, at most, **3% performance loss**."

For power-constrained data centers, using more efficient chips or running at optimized power profiles can significantly improve throughput per watt.

### NVIDIA Data Center GPU Manager - Power Profiles

NVIDIA provides energy-optimized power profiles for data center GPUs:

| Profile | Use Case | Power Savings | Performance Impact |
|---------|----------|--------------|-------------------|
| MAX-Q | Power-constrained | Up to 15% | At most 3% loss |
| MAX-P | Performance-optimized | 0% | 2-3% improvement |
| Training | Training workloads | 5% | 1% loss |
| Inference | Inference workloads | 8% | 3% loss |

**Source:** [NVIDIA Developer Blog - Data Center Power Profiles](https://developer.nvidia.com/blog/optimize-data-center-efficiency-for-ai-and-hpc-workloads-with-power-profiles/)

---

## Undervolting Stability Testing

### Why Stability Testing Matters

> "Undervolting lowers the operating temperature of your CPU or GPU to unlock additional thermal headroom, which results in sustained boost clocks and consistent performance over long sessions rather than the short, erratic bursts of performance and the thermal penalty that comes with overclocking." — [XDA Developers](https://www.xda-developers.com/undervolting-is-great-but-you-need-to-properly-test-stability/)

### Testing Methodology

**Key Principle:**
> "There is absolutely such a thing as undervolting **too far.** Push the curve too aggressively, and you'll achieve the exact opposite of what an undervolt seeks to do."

### Recommended Stress Testing Tools

| Tool | Description | Best For |
|------|-------------|----------|
| **OCCT** | Comprehensive stability testing | GPU undervolt verification |
| **FurMark** | GPU stress testing | Thermal stress testing |
| **3D Mark Time Spy** | Graphics benchmark | Long-duration stability |
| **SLI Stress Test** | Extended testing | Production validation |

> "OCCT can be your one-stop benchmarking tool... Use 3D Mark firestrike. It reacts extremely sensible to unstable core clock / Voltages. If you can get 2-3 runs without problems through, you are quite well [stable]." — [Linus Tech Tips](https://linustechtips.com/topic/842049-what-should-i-use-to-stability-test-a-gpu-undervolt/)

### Stability Testing Process

1. **Baseline**: Run at default voltage, record performance
2. **Gradual Reduction**: Decrease voltage by 15-25mV increments
3. **Stress Test**: Run OCCT or FurMark for 30-60 minutes
4. **Verification**: Run Time Spy Graphics Test 2 on loop
5. **Production Test**: Run actual training workload for extended period

### Key Metrics to Monitor

| Metric | Acceptable Range | Warning Sign |
|--------|------------------|--------------|
| GPU Temperature | < 80°C under load | > 85°C sustained |
| Clock Stability | Consistent boost clocks | Frequent downclocking |
| Power Draw | Normal range | Spikes or drops |
| Error Count | Zero errors | Any ECC or 计算错误 |

> "Undervolting can never damage your components so there is no urgent need to find the absolute stability." — [Reddit r/linux_gaming](https://www.reddit.com/r/linux_gaming/comments/1mx9u9n/when_undervolting_your_gpu_how_important_is_it_to/)

### Automated Undervolting Utilities

- **NVIDIA Inspector** (Windows)
- **MSI Afterburner** (cross-platform)
- **nvidia-settings** (Linux)
- **GreenICT** (specialized for data center)

---

## Key References

### Official Documentation
| Document | URL |
|----------|-----|
| NVIDIA DCGM Documentation | https://docs.nvidia.com/datacenter/dcgm/latest/user-guide/ |
| NVIDIA GPU Boost Documentation | https://developer.nvidia.com/blog/cuda-pro-tip-increase-application-performance-nvidia-gpu-boost/ |
| AMD P-State Kernel Documentation | https://docs.kernel.org/admin-guide/pm/amd-pstate.html |
| PyTorch CUDA Memory | https://docs.pytorch.org/docs/stable/torch_cuda_memory.html |

### Research Papers
| Paper | URL |
|-------|-----|
| Impact of GPU DVFS on Deep Learning (2019) | https://arxiv.org/pdf/1905.11012 |
| Kernel-Level DVFS for LLMs (2026) | https://arxiv.org/html/2601.08539v1 |
| DVFS-GPT for LLMs (2025) | https://www.irjmets.com/upload_newfiles/irjmets70700042545/paper_file/irjmets70700042545.pdf |
| Data Center Power Profiles (2025) | https://arxiv.org/pdf/2510.03872 |

### Tools & Software
| Tool | URL |
|------|-----|
| Zeus (PyTorch Energy Optimization) | https://github.com/ml-energy/zeus |
| NVIDIA PyProf | https://github.com/NVIDIA/PyProf |
| CodeCarbon | https://codecarbon.io/ |
| DCGM Exporter | https://github.com/NVIDIA/dcgm-exporter |

### Community Resources
| Resource | URL |
|----------|-----|
| NVIDIA GPU Boost 3.0 (SkatterBencher) | https://skatterbencher.com/nvidia-gpu-boost-3-0/ |
| Undervolting Stability Guide (XDA) | https://www.xda-developers.com/undervolting-is-great-but-you-need-to-properly-test-stability/ |
| GPU Monitoring Guide (Spheron) | https://www.spheron.network/blog/gpu-monitoring-for-ml/ |

---

## Related Topics

- [Energy-Efficient Training Techniques](../green-ai-thermals/README.md)
- [GPU Provisioning and Management](../training-infrastructure/dynamic-provisioning.md)
- [Observability for ML Infrastructure](../training-infrastructure/observability-resilience/README.md)

---

*Last updated: 2026-05-01*