# P-EAGLE: Implementation

> **Source:** [vLLM PR #32887](https://github.com/vllm-project/vllm/pull/32887) · [AWS Blog](https://aws.amazon.com/blogs/machine-learning/p-eagle-faster-llm-inference-with-parallel-speculative-decoding-in-vllm/) · [TensorRT-LLM Docs](https://nvidia.github.io/TensorRT-LLM/1.2.0rc6/features/speculative-decoding.html) · [SGLang Issue #23171](https://github.com/sgl-project/sglang/issues/23171)  
> **Subtopic of:** [P-EAGLE](./README.md)

---

## vLLM Integration (v0.16.0+)

### Enable P-EAGLE with One Config Flag

```python
# In vllm/config/speculative.py
parallel_drafting: bool = True
```

### Serving Command

```bash
vllm serve openai/gpt-oss-20b \
   --speculative-config '{
     "method": "eagle3",
     "model": "amazon/gpt-oss-20b-p-eagle",
     "num_speculative_tokens": 5,
     "parallel_drafting": true
   }'
```

> *Source: [vLLM PR #32887](https://github.com/vllm-project/vllm/pull/32887)*

---

## System-Level Challenges & Solutions

### Challenge 1: Batch Shape Mismatch

> *"In standard EAGLE, drafting and verification share the same per-request token layout. Parallel drafting breaks this: MASK placeholders (`[token, MASK, MASK, …]`) create a draft batch shape that no longer matches the verification batch shape. This requires rebuilding batch metadata: expanding input token IDs, hidden states, positions, recomputing slot mapping and per-request start indices."*  
> — [AWS Blog](https://aws.amazon.com/blogs/machine-learning/p-eagle-faster-llm-inference-with-parallel-speculative-decoding-in-vllm/)

### Solution: Fused Triton Kernel

vLLM introduced a **single Triton kernel** that rebuilds batch metadata in one pass:

1. Copies previous token IDs and positions from target batch into new destination slots
2. Inserts the per-request bonus token sampled by the target model
3. Fills extra parallel-drafting slots with special MASK token ID
4. Generates lightweight metadata: rejected-token mask, masked-token mask, new-token indices, hidden-state mapping

> *"Fusing it into one kernel reduces launch overhead and extra memory traffic, keeping the drafting setup cheap."*  
> — [AWS Blog](https://aws.amazon.com/blogs/machine-learning/p-eagle-faster-llm-inference-with-parallel-speculative-decoding-in-vllm/)

### Hidden State Management (Code)

```python
# Copy target hidden states to their new positions
self.hidden_states[out_hidden_state_mapping] = target_hidden_states

# Fill masked positions with the learned Parallel Drafting hidden state
mask = self.is_masked_token_mask[:total_num_output_tokens]
torch.where(
    mask.unsqueeze(1),
    self.parallel_drafting_hidden_state_tensor,
    self.hidden_states[:total_num_output_tokens],
    out=self.hidden_states[:total_num_output_tokens],
)
```

### Challenge 2: KV Cache Handling

- Valid tokens get normal slot assignment
- Rejected tokens map to `PADDING_SLOT_ID` (-1) to suppress spurious cache writes
- CUDA graph capture range extended by `K × max_num_seqs` for larger draft batch

---

## Training Implementation

### The Memory Challenge

> *"Training K parallel prediction depths on sequences of length n creates n×K total positions with O((nK)²) attention complexity. PARD's per-example mask construction requires O((nK)²) operations per example—prohibitive at long contexts."*  
> — [vLLM Speculators RFC](https://github.com/vllm-project/speculators/issues/292)

### Solution 1: Amortized Mask Construction

**Key insight:** The causal structure across prediction depths is position-invariant.

**Method:** Construct attention mask once at training initialization for maximum sequence length. Per-example masks obtained via tensor slicing (constant-time view operation, no memory allocation).

### Training Efficiency Comparison

| Method | Load (128 examples) | Epoch Time |
|---|---|---|
| EAGLE-3 | 14.8s | 2.5h |
| PARD | 718.5s (48× slower) | 12+h (5× slower) |
| **P-EAGLE** | **17.5s** | **1.8h** |

> *P-EAGLE's amortized mask construction achieves near-EAGLE-3 training efficiency while enabling parallel drafting.*  
> — [arXiv:2602.01469](https://arxiv.org/abs/2602.01469)

### Solution 2: Sequence Partition Algorithm

- Divides the N×K position sequence into contiguous chunks
- Maintains correct attention dependencies across chunk boundaries
- Accumulates gradients across chunks of the same sequence

### Conditional-On-Distribution (COD) Sampling

Reduces positions via geometric decay:
- Depth 0: retains all n positions
- Depth 1: randomly retains n × r positions
- Depth 2: retains n × r², etc.

Where r ∈ (0,1) is the `down_sample_ratio`. Reduces total positions from n×K to n×(1 + r + r² + ⋯ + r^{K-1}).

---

## TensorRT-LLM Support

### Two Implementation Flavors

| | One Model | Two Model |
|---|---|---|
| **How it works** | Drafter inserted directly into model code as submodule | Draft tokens produced in `PyExecutor`, attached to requests before target model |
| **Performance** | Faster — entire drafting loop launched as single CUDA graph | Standard |
| **Flexibility** | Limited — no dynamic draft lengths | More flexible |
| **Overlap scheduler** | Supported | **Automatically disabled** |

### EAGLE Support Matrix (TensorRT-LLM)

| Algorithm | One Model | Two Model |
|---|---|---|
| EAGLE 3 | Llama 4 Maverick | Llama 4 Maverick, Llama 3.1 8B, Llama 3.3 70B |
| Draft/target | — | All models |
| NGram | — | All models |
| User-provided | — | All models |

### Configuration Example

```python
from tensorrt_llm.llmapi import EagleDecodingConfig

eagle3_one_model = False  # Set True for faster one-model implementation (Llama 4 only)
speculative_config = EagleDecodingConfig(
    max_draft_len=3, 
    speculative_model="/path/to/draft_model", 
    eagle3_one_model=eagle3_one_model
)
llm = LLM("/path/to/target_model", speculative_config=speculative_config, disable_overlap_scheduler=True)
```

> *Note: TensorRT-LLM supports a **modified version** of EAGLE-3 where **tree structures for draft sequences are not supported** — each request uses a single sequence of draft tokens with length `max_draft_len`.*  
> — [TensorRT-LLM Docs](https://nvidia.github.io/TensorRT-LLM/1.2.0rc6/features/speculative-decoding.html)

---

## SGLang Support (Feature Request)

From [GitHub Issue #23171](https://github.com/sgl-project/sglang/issues/23171):

**Key implementation pieces identified:**

> *"**Parallel input preparation.** The drafter batch contains all K positions in one shot: `h_context`, `mask`, `h_shared`."*
>
> *"**Fused input-prep kernel.** vLLM introduced a single Triton kernel that rebuilds batch metadata — copies target token IDs / positions, inserts bonus tokens, fills parallel-drafting slots with the MASK id, and emits rejected-token masks + slot mappings in one pass."*
>
> *"**KV cache handling.** Valid tokens get normal slot assignments; rejected tokens map to `PADDING_SLOT_ID (-1)` to suppress spurious cache writes. CUDA graph capture range must extend by `K × max_num_seqs`."*

SGLang is actively working on P-EAGLE support.

---

## Key URLs

| Resource | URL |
|---|---|
| vLLM PR #32887 | [github.com/vllm-project/vllm/pull/32887](https://github.com/vllm-project/vllm/pull/32887) |
| vLLM Speculators RFC | [github.com/vllm-project/speculators/issues/292](https://github.com/vllm-project/speculators/issues/292) |
| vLLM Speculators PR #343 | [github.com/vllm-project/speculators/pull/343](https://github.com/vllm-project/speculators/pull/343) |
| TensorRT-LLM EAGLE | [github.com/NVIDIA/TensorRT-LLM/blob/main/examples/eagle/README.md](https://github.com/NVIDIA/TensorRT-LLM/blob/main/examples/eagle/README.md) |
| SGLang Issue #23171 | [github.com/sgl-project/sglang/issues/23171](https://github.com/sgl-project/sglang/issues/23171) |
