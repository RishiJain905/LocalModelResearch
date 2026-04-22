# Hardware-Accelerations / Misc

> *Additional hardware acceleration topics — covering vendor software stacks, workload-specific optimization, consumer vs datacenter GPU tradeoffs, and GEMM/matmul kernel optimization.*

---

## Subtopics

| Subtopic | Description |
|----------|-------------|
| **[Vendor-Software-Stacks.md](./Vendor-Software-Stacks.md)** | Full AI/ML software ecosystems from NVIDIA (CUDA/TensorRT/Triton), AMD (ROCm), Intel (oneAPI/OpenVINO), Qualcomm (AI Hub/QNN), Apple (CoreML/MLX/MPS), and Google (JAX/XLA/TPU) |
| **[Workload-Specific-Acceleration.md](./Workload-Specific-Acceleration.md)** | Domain-specific acceleration for vision (DALI, OpenCV DNN), speech (Whisper.cpp, Riva), recommendations (Merlin, HugeCTR), LLMs (vLLM, llama.cpp, TensorRT-LLM), RAG (FAISS, Milvus), scientific computing, and medical imaging (MONAI, Clara) |
| **[Consumer-vs-Datacenter-GPU.md](./Consumer-vs-Datacenter-GPU.md)** | Architectural and software differences between consumer (RTX 4090) and datacenter (H100, A100, MI300X) GPUs — memory bandwidth, VRAM, FP8, NVLink, cloud pricing, driver differences |
| **[GEMM-MatMul-Kernel-Optimization.md](./GEMM-MatMul-Kernel-Optimization.md)** | Deep dive into GEMM as the core AI operation — CUDA memory hierarchy, CUTLASS, hipBLASLt, Intel AMX, Apple MLX, FlashAttention 1/2/3, kernel fusion, and benchmark data across all major platforms |

---

## Key Findings

### Vendor Software Stacks
- **NVIDIA** maintains the deepest ecosystem with CUDA, TensorRT, Triton — the dominant choice for production AI
- **AMD ROCm** has matured significantly in 2026, now treated as first-class in PyTorch 2.6
- **Apple MLX** delivers 3× faster LLM inference vs PyTorch MPS on Apple Silicon with unified memory
- **Google TPU/JAX** offers 4× better price-performance for inference workloads vs H100

### Workload-Specific Acceleration
- **LLM inference**: vLLM (PagedAttention), llama.cpp (INT4 quantization), TensorRT-LLM (FP8)
- **Vision**: NVIDIA DALI removes CPU bottlenecks in training pipelines
- **RAG**: FAISS (GPU-accelerated), Milvus, Qdrant all enable billion-scale vector search
- **Medical**: MONAI + NVIDIA Clara for federated learning across hospitals

### Consumer vs Datacenter GPU
- **Memory bandwidth is the key differentiator**: HBM3 (3.35 TB/s) vs GDDR6X (1 TB/s) fundamentally limits AI performance
- **VRAM capacity**: RTX 4090's 24GB limits unquantized models to ~7B; H100/MI300X run 70B+ unquantized
- **FP8 is datacenter-only**: H100 native FP8 = 6× performance gain for transformers
- **NVLink dropped on consumer GPUs**: Multi-GPU scaling requires datacenter hardware
- **Cloud pricing 5× variance**: Same H100 is $2.49/hr on RunPod vs $12.30/hr on AWS

### GEMM Kernel Optimization
- **FlashAttention-3 achieves 75% of H100 theoretical max** (~740 TFLOPS FP16, 1.2 PFLOPS FP8)
- **CUTLASS Ping-Pong GEMM** is the fastest matmul kernel architecture on Hopper
- **TMA (Tensor Memory Accelerator)** on Hopper enables async memory copies that overlap with compute
- **Apple Neural Engine** + MLX delivers 4× speedup vs M4 baseline for LLM time-to-first-token
- **Intel AMX** on Xeon Sapphire Rapids provides BF16 and INT8 matrix multiply acceleration
- **Kernel fusion** (GEMM + activation) eliminates memory traffic for intermediate writes
