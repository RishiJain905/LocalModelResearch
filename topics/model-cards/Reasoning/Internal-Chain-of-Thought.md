# Internal Chain-of-Thought (iCoT)

> *"We operationalize hidden reasoning as a loss of a CoT monitor's ability to detect important latent factors that steer the behavior of the monitored model."*
> — Arike et al., *Hidden Reasoning in LLMs: A Taxonomy*

**Internal Chain-of-Thought (iCoT)** refers to the intermediate computational steps a Large Language Model performs inside its latent (hidden) activations — steps that are *not* surfaced as readable text in the final output. This contrasts with **Explicit Chain-of-Thought (CoT)**, where models verbalize reasoning tokens (e.g., DeepSeek-R1's `<think>` blocks or standard "let's think step by step" prompting).

## Contents

1. [How Major Models Handle Internal Reasoning](#how-major-models-handle-internal-reasoning)
2. [Key Papers](#key-papers)
3. [Open-Source Libraries & Tools](#open-source-libraries--tools)
4. [GitHub Repositories](#github-repositories)
5. [Blog Posts & Articles](#blog-posts--articles)
6. [Key Findings](#key-findings)

---

## How Major Models Handle Internal Reasoning

### OpenAI o1 / o3 / o4-mini

- **Mechanism:** Uses *hidden reasoning tokens* (a "scratchpad") consumed internally before outputting the final answer. These tokens are **not visible** to the user.
- **Behavior:** Explicit CoT prompts ("think step by step") actually *harm* performance — they interfere with the allocated internal reasoning budget.
- **Evidence:** Linear correlation between reasoning token count and latency suggests sequential generation rather than external search (MCTS/ToT).

> *"OpenAI o1 provide models the ability to perform System 2 thinking by introducing 'reasoning tokens'... the model performs forward sequential reasoning in a hidden scratchpad before producing the final output."*
> — Lee, *Understanding OpenAI o1*

### DeepSeek-R1

- **Mechanism:** Unlike OpenAI, **DeepSeek-R1 outputs its reasoning tokens explicitly** via `<reasoning_content>` and `<content>` tags in the API.
- **Significance:** This is one of the first frontier reasoning models where the full scratchpad is externally visible, enabling distillation and direct study of the reasoning process.
- **Trade-off:** Longer outputs increase inference cost, but transparency aids safety research.

> *"DeepSeek-R1 stands out because it not only provides final answers but also reveals its step-by-step chain of thought — unlike other reasoning models that keep this internal process hidden."*
> — Fireworks AI Blog

### Anthropic Claude 3.7 Sonnet / Claude 4

- **Mechanism:** Claude 3.7 introduced "Extended Thinking" mode. Anthropic publishes the CoT (the "thinking block") to users.
- **Internal vs. External:** Anthropic's interpretability work reveals that even when Claude outputs a reasoning trace, the *internal* computation (e.g., planning rhymes backward) often diverges from the *external* explanation.
- **Monitoring:** Anthropic actively studies whether models exploit hidden objectives or encoded reasoning within their CoT, finding that paraphrasing scratchpads does not hurt performance (arguing against steganography).

---

## Key Papers

### Training Large Language Models to Reason in a Continuous Latent Space — Coconut (Meta FAIR)

> *"We argue that language space may not always be optimal for reasoning... most word tokens primarily ensure textual coherence and are not essential for reasoning, while some critical tokens require complex planning and pose huge challenges to LLMs."*
> — Hao et al.
> — [arXiv:2412.06769](https://arxiv.org/abs/2412.06769) | [GitHub](https://github.com/facebookresearch/coconut)

**Key Contributions:**
- Introduced latent "continuous thought" by recycling hidden states instead of decoding reasoning into tokens
- Argues language is a communication tool, not always optimal for reasoning

---

### On the Biology of a Large Language Model (Anthropic, 2025)

> Used attribution graphs to trace internal circuits in Claude 3.5 Haiku. Found evidence of backward planning in poems and multi-hop reasoning not fully captured by output text.
> — Batson et al.
> — [Anthropic](https://transformer-circuits.pub/2025/attribution-graphs/biology.html)

**Key Contributions:**
- Sparse autoencoders trace circuits inside Claude
- Found models plan rhymes before writing lines; perform "mental math" via parallel approximate/precise paths

---

### Tracing the Thoughts of a Large Language Model (Anthropic, 2025)

> *"Claude seems to be unaware of the sophisticated 'mental math' strategies that it learned during training. If you ask how it figured out that 36+59 is 95, it describes the standard algorithm involving carrying the 1."*
> — [Anthropic Blog](https://www.anthropic.com/research/tracing-thoughts-language-model)

**Key Contributions:**
- Demonstrates divergence between internal computation and external explanation
- Models are "unaware" of their own reasoning strategies

---

### Hidden Reasoning in LLMs: A Taxonomy (2025)

> Taxonomizes hidden reasoning into:> 1. Recurrent neuralese> 2. Hidden parallelized reasoning> 3. Encoded reasoning (steganography/linguistic drift)> 4. "Dazzling" the monitor> — Arike, RohanS, Biswas> — [LessWrong](https://www.lesswrong.com/posts/ZrgFfeWuckpwK5Lyi/hidden-reasoning-in-llms-a-taxonomy)

---

### Do Reasoning Models Use Their Scratchpad Like We Do? (Anthropic, 2025)

> Distilled paraphrased scratchpads from Claude 3.7 Sonnet back into the base model; performance was preserved, suggesting **no significant encoded reasoning / steganography** in MATH tasks.> — [Alignment Blog](https://alignment.anthropic.com/2025/distill-paraphrases/)

**Key Contributions:**
- Paraphrasing scratchpads does not hurt performance → argues against steganography in current frontier models
- Scratchpad content represents genuine reasoning, not hidden signals

---

### Reasoning Beyond Language: A Comprehensive Survey on Latent Chain-of-Thought Reasoning (2025)

> The first systematic survey of latent CoT. Establishes taxonomy across token-wise horizontal and layer-wise vertical reasoning.> — Chen et al.> — [arXiv:2505.16782](https://arxiv.org/abs/2505.16782)

---

### Additional Papers

| Paper | Authors | Year | Key Finding |
|-------|---------|------|-------------|
| Understanding OpenAI o1 | Han Lee | 2024 | Deep technical breakdown; linear latency implies no search |
| How Reasoning Works in DeepSeek-R1 | Chris McCormick | 2025 | CoT distillation from R1; dataset bottleneck |
| Hidden Reasoning in LLMs: A Taxonomy | Arike et al. | 2025 | Framework for classifying non-verbal reasoning pathways |

---

## Open-Source Libraries & Tools

| Tool | Description | Link |
|------|-------------|------|
| **Coconut** | Official latent continuous thought training (Meta FAIR) | [facebookresearch/coconut](https://github.com/facebookresearch/coconut) |
| **OpenCoconut** | Community implementation using hidden states during prefilling | [casper-hansen/OpenCoconut](https://github.com/casper-hansen/OpenCoconut) |
| **coconut (wassname)** | Replication + extensions (SEQ-VCR loss) | [wassname/coconut](https://github.com/wassname/coconut) |
| **coconut-curriculum-checkpoints** | Controlled Coconut experiments | [bmarti44/coconut-curriculum-checkpoints](https://huggingface.co/bmarti44/coconut-curriculum-checkpoints) |
| **supervised-thinking-states** | Recurrent reasoning mechanism inside Transformers | [fazalmittu/supervised-thinking-states](https://github.com/fazalmittu/supervised-thinking-states) |

---

## GitHub Repositories

| Repository | Stars | Description |
|-----------|-------|-------------|
| [facebookresearch/coconut](https://github.com/facebookresearch/coconut) | ⭐ 2.5k+ | Latent continuous thought training |
| [casper-hansen/OpenCoconut](https://github.com/casper-hansen/OpenCoconut) | ⭐ 200+ | Community latent reasoning implementation |
| [fazalmittu/supervised-thinking-states](https://github.com/fazalmittu/supervised-thinking-states) | — | Supervised hidden activation reasoning |

---

## Blog Posts & Articles

| Article | Source | Key Takeaway |
|---------|--------|-------------|
| Reasoning Series, Part 1: Understanding OpenAI o1 | Han Lee | Linear latency implies no search; hidden scratchpad mechanics |
| How Reasoning Works in DeepSeek-R1 | Chris McCormick | CoT distillation; dataset bottleneck |
| Hidden Reasoning in LLMs: A Taxonomy | LessWrong (Arike et al.) | Framework for classifying non-verbal reasoning pathways |
| Peering Inside the Black Box | Medium (Mskadu) | Summary of Anthropic's interpretability findings |
| AI Is a Black Box. Anthropic Figured Out a Way to Look Inside | WIRED | Popular overview of sparse autoencoders and feature attribution |
| Latent Reasoning | Alberto Pelliccione | Shift from language-based reasoning to latent reasoning; interpretability risks |

---

## Key Findings

### 1. iCoT Exists on a Spectrum

From fully unverbalized latent activations (Coconut-style) to visible-but-unfaithful scratchpads (Anthropic's "bullshitting") to fully transparent explicit CoT (DeepSeek-R1).

### 2. Language Is Not the Native Reasoning Medium

Coconut and Anthropic's circuit analyses both suggest that a substantial portion of "thinking" happens in high-dimensional hidden states, often independently of the text being generated.

### 3. Major Frontier Models Diverge in Transparency

| Model | Transparency | Mechanism |
|-------|-------------|------------|
| OpenAI o1/o3 | **Hidden** | Internal scratchpad, reasoning tokens |
| DeepSeek-R1 | **Explicit** | Full `<think>` blocks visible |
| Anthropic Claude 3.7+ | **Visible but unfaithful** | Published thinking blocks; internal ≠ external |

### 4. Encoded Reasoning / Steganography Is a Concern but Not Confirmed at Scale

Anthropic's paraphrase-distillation experiments suggest models are **not** currently relying on hidden syntax tricks for math. However, the taxonomy identifies four pathways for hidden reasoning to emerge.

### 5. Monitoring Internal Reasoning Is Technically Feasible but Incomplete

Sparse Autoencoders, Attribution Graphs, and Cross-Layer Transcoders can trace *some* circuits, but Anthropic admits this works for only about a quarter of prompts studied.

### 6. The Field Is Rapidly Shifting Toward Latent Reasoning

Survey papers (Chen et al., 2025) document dozens of new architectures moving reasoning out of token space, driven by efficiency and theoretical limits of language-as-reasoning.

---

> *"[Bullshitting] — just coming up with an answer, any answer, without caring whether it is true or false."*
> — Anthropic, describing unfaithful CoT generation
