# Reasoning-Effort Params

## 📌 Overview

Reasoning-effort parameters control how much internal "thinking" compute a model uses before producing output. These are also called **thinking tokens** or **reasoning tokens** — internal tokens that models use to "think" before generating visible output. They are hidden (not visible via API), billed as output tokens, and count against context window limits. Providers use different parameter names: OpenAI uses `reasoning.effort` (enum), Anthropic uses `thinking.budget_tokens` (integer), DeepSeek uses model selection alone.

---

## Definitions & Standards

> "Reasoning models generate reasoning tokens in addition to input and output tokens. The model uses these tokens to 'think,' breaking down the prompt and considering multiple approaches. After generating reasoning tokens, the model produces visible completion tokens and **discards the reasoning tokens from its context**."

— [OpenAI API Documentation](https://developers.openai.com/api/docs/guides/reasoning)

### Provider-Specific Parameter Naming

| Provider | Parameter | Type | Example |
|----------|-----------|------|---------|
| **OpenAI** | `reasoning.effort` | Enum | `none`, `minimal`, `low`, `medium`, `high`, `xhigh` |
| **Anthropic** | `thinking.budget_tokens` | Integer | `1024`, `32000`, etc. |
| **DeepSeek** | `deepseek-reasoner` model | Model selection | Single mode (no budget tuning) |
| **OpenRouter** | `reasoning.max_tokens` or `reasoning.effort` | Mixed | Provider-dependent |

> "For models that only support `reasoning.effort`, the `max_tokens` value will be used to determine the effort level."

— [OpenRouter Docs](https://openrouter.ai/docs/guides/best-practices/reasoning-tokens)

### Effort Level Guide (OpenAI)

| Effort | When to Use |
|--------|-------------|
| `none` | Extraction, routing, simple transforms |
| `low` | Small improvement without latency cost |
| `medium`/`high` | Planning, coding, synthesis |
| `xhigh` | Only when eval shows clear benefit |

> "If generated tokens hit the context window limit or your `max_output_tokens` value, you will receive a response with status: incomplete... This can happen **before any visible output tokens are produced**, meaning you may incur costs for input and reasoning tokens without receiving a visible response."

— [OpenAI API Documentation](https://developers.openai.com/api/docs/guides/reasoning)

> "When using extended thinking with tool use, thinking blocks are cached and count as input tokens when read from cache."

— [AWS Bedrock — Anthropic Extended Thinking](https://docs.aws.amazon.com/bedrock/latest/userguide/claude-messages-extended-thinking.html)

---

## Key Sources

### Academic Papers

**"Increasing the Thinking Budget is Not All You Need"** (arXiv:2512.19585)
*Ignacio Iacobacci et al. (Elm Company), December 2025*
**URL:** https://arxiv.org/html/2512.19585v1

> "Simply increasing the thinking budget is not the most effective use of compute. More accurate responses can instead be achieved through alternative configurations, such as self-consistency and self-reflection."

**Finding:** Summary strategy (multiple generations consolidated via LLM) at 6K tokens × 4 calls = **80.00%** on AIME24, vs vanilla at 20K tokens = 73.33% (same total budget)

---

**"Token-Budget-Aware LLM Reasoning (TALE)"** — ACL 2025 Findings
*Tingxu Han, Zhenting Wang, et al.*
**URL:** https://aclanthology.org/2025.findings-acl.1274.pdf
**Code:** https://github.com/GeniusHTX/TALE

> "Including a reasonable token budget (e.g., 50 tokens in this case) in the instructions reduces the token cost in the chain-of-thought (CoT) process from 258 output tokens to 86 output tokens, while still enabling the LLM to arrive at the correct answer."

**Token Reduction:** ~67% vs vanilla CoT with <3% accuracy loss

---

**"BudgetThinker: Empowering Budget-aware LLM Reasoning with Control Tokens"** (arXiv:2508.17196)

> "We argue that merely stating a budget constraint in the initial prompt is insufficient, and the model must be continuously reminded of its remaining token budget as it generates its response."

**Method:** Periodic control token injection + SFT + curriculum GRPO | **Result:** +4.9% accuracy improvement across budgets

---

**"Towards Thinking-Optimal Scaling of Test-Time Compute"** — NeurIPS 2025
**URL:** https://openreview.net/forum?id=6ICFqmixlS

> "Would excessively scaling the CoT length actually bring adverse effects to a model's reasoning performance?"

**Finding:** Excessively scaling CoT can impair performance in certain domains

---

**"Think Deep, Not Just Long: Measuring LLM Reasoning Effort via Deep-Thinking Tokens"** (arXiv:2602.13517)
*cs.CL (Work in progress)*

Introduces **Deep-Thinking Tokens** and **Deep-Thinking Ratio (DTR)** as measurement metrics.

---

**"Adaptive Test-Time Compute Allocation for Reasoning LLMs"** (arXiv:2604.14853)

**Formalization:** `b⋆(x;λ) = arg max_b [Acc(x,b) − λ·C(b)]`

---

## Repositories & Tools

### OptiLLM
**URL:** https://github.com/algorithmicsuperintelligence/optillm
- OpenAI API-compatible proxy implementing 20+ optimization techniques
- Stars: 3.4k | License: Apache-2.0
- Implements `reasoning_effort` param via "Thinkdeeper" technique
- Results: +30 pp on AIME 2025 (MARS), +18.6 pp on Math-L5 (CePO)

### OpenR Framework
**URL:** https://github.com/openreasoner/openr
**arXiv:** https://arxiv.org/abs/2410.09671
- Open-source framework for advanced reasoning with LLMs
- Includes process-supervision data generation and step-aware verification

### o1-reasoning-tokens
**URL:** https://github.com/da03/o1-reasoning-tokens
- Analyzes reasoning token counts from OpenAI's o1-mini
- Tests effect of `max_completion_tokens` on reasoning token usage

### Inspect AI (Evaluation Framework)
**URL:** https://inspect.aisi.org.uk/reasoning.html
- Used by Anthropic, DeepMind, Grok for LLM evals
- CLI flags: `--reasoning-effort`, `--reasoning-tokens`
- Supports OpenAI o-series, Claude Sonnet 3.7, Gemini 2.5/3, Grok 3, DeepSeek R1

### TALE (Token-Budget-Aware LLM Reasoning)
**URL:** https://github.com/GeniusHTX/TALE

Two implementations:
- **TALE-EP:** Training-free (budget estimation → prompt injection)
- **TALE-PT:** Post-training via SFT/DPO

---

## Debates & Controversies

### 4.1 Model-Dependent Default Values
**Evidence (Kevin Whinnery on X):**
> "Note to @OpenAIDevs API users - the default value of reasoning effort is now different depending on which model you use! gpt-5.1 -> 'none'"

— [X/Twitter](https://x.com/kevinwhinnery/status/1989338879065751863)

**Community clarification:**
> "If you do not specify Reasoning Effort, the system will automatically set it to a medium (or auto) level by default."

— [OpenAI Community](https://community.openai.com/t/clarification-on-reasoning-effort-default-in-the-o1-api/1066413)

**Problem:** Developers expecting consistent behavior may get unexpected costs/quality.

### 4.2 More Thinking ≠ Better Results
**Finding from arXiv:2512.19585:**
- Vanilla thinking plateaus at larger budgets
- Weaker models (Qwen3-4B, DeepSeek-8B) don't benefit from increased budgets
- Summary strategy at 24K total tokens outperforms vanilla at 20K tokens (80% vs 73.33%)

### 4.3 Token Elasticity Phenomenon
**From TALE paper:**
> "When the budget is reduced beyond a certain range, the token cost increases... the model, unable to meet the tight constraint, ignores it and reverts to longer reasoning."

**Example:** Budget of 10 tokens → output of 157 tokens (vs 86 tokens with 50-token budget)

### 4.4 Hidden/Incomplete Response Costs
Incomplete responses due to `max_output_tokens` limit can still incur full input + reasoning token costs.

### 4.5 Overthinking Problem
NeurIPS 2025 paper finds excessively scaling CoT length can harm performance in certain domains.

---

## Summary Table

### Performance Data (AIME 2024 — "Increasing the Thinking Budget")

| Configuration | Budget | Total Tokens | Qwen3-8B |
|--------------|--------|-------------|----------|
| Vanilla | 12,000 | 12,000 | 66.67% |
| Vanilla | 20,000 | 20,000 | 73.33% |
| Self-Consistency 3x | 8,000 | 24,000 | 63.33% |
| **Summary 3x** | **6,000** | **24,000** | **80.00%** |

### TALE Token Reduction Results

| Metric | Direct | Vanilla CoT | TALE-EP |
|--------|--------|-------------|---------|
| Accuracy | 52.31% | 83.75% | **81.03%** |
| Output Tokens | 14.57 | 461.25 | **148.72** |
| Cost (10⁻⁵$/sample) | 25.37 | 289.78 | **118.46** |

### DeepSeek Reasoning Model Specs

| Aspect | Value |
|--------|-------|
| Model | `deepseek-reasoner` |
| Max Output | Default 32K, Max 64K |
| Context | 128K |
| Pricing | Input: $0.28/M (miss), $0.028/M (hit); Output: $0.42/M |

### OpenAI Model Defaults

| Model | Default Effort |
|-------|----------------|
| GPT-5.4 | `none` |
| Older GPT-5 | `medium` |

---

## Related Topics

- [iCoT Tracking](./iCoT-Tracking.md) — Implicit chain-of-thought reasoning (hidden state vs. token-based reasoning)
- [FastAPI 2026](../Async-Frameworks/FastAPI-2026.md) — API framework that may surface reasoning metadata
- [MCP-Protocol](../MCP-Protocol/README.md) — Tool schemas and observability patterns
