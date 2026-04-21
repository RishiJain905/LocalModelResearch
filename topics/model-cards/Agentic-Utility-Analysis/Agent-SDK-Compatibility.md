# Agent SDK Compatibility

> "Large language model (LLM)-powered autonomous agents demand robust, standardized protocols to integrate tools, share contextual data, and coordinate tasks across heterogeneous systems." — Abul Ehtesham et al., *A Survey of Agent Interoperability Protocols* (arXiv:2505.02279)

## Contents

1. [Key Papers](#key-papers)
2. [Open-Source Libraries & Tools](#open-source-libraries--tools)
3. [GitHub Repositories](#github-repositories)
4. [Blog Posts & Articles](#blog-posts--articles)
5. [Key Findings](#key-findings)

---

## Key Papers

### A Survey of Agent Interoperability Protocols: MCP, ACP, A2A, and ANP (2025)

> *"Large language model (LLM)-powered autonomous agents demand robust, standardized protocols to integrate tools, share contextual data, and coordinate tasks across heterogeneous systems."* — Abul Ehtesham, Aditi Singh, Gaurav Kumar Gupta, Saket Kumar
> — arXiv
> — [arXiv:2505.02279](https://arxiv.org/html/2505.02279v1)

**Key Contributions:**
- Comparative survey of four interoperability protocols addressing distinct tiers: MCP (tool access), ACP (multimodal messaging), A2A (collaborative task execution), and ANP (decentralized agent marketplaces)
- Proposes a phased adoption roadmap: **MCP → ACP → A2A → ANP**
- Security threat taxonomy across creation, operation, and update lifecycle phases

> "No single protocol suffices for all contexts; instead, the authors propose a phased, complementary adoption strategy."

---

### Unified Tool Integration for LLMs: A Protocol-Agnostic Approach to Function Calling (2025)

> *"The tool-augmented LLM ecosystem is fragmented across four dimensions: protocol fragmentation, manual implementation overhead, complex execution workflow, and OpenAI dominance."* — Authors
> — arXiv
> — [arXiv:2508.02979](https://arxiv.org/html/2508.02979v1) | [arXiv:2507.10593](https://arxiv.org/html/2507.10593v2)

**Key Contributions:**
- Introduces **ToolRegistry**, a protocol-agnostic tool management library
- Demonstrates **60–80% code reduction** across integration scenarios
- Achieves **up to 3.1× performance improvement** via optimized concurrent execution
- Supports Python functions, MCP servers, OpenAPI services, and LangChain tools through a single interface

> "ToolRegistry natively supports Python functions, MCP servers, OpenAPI services, and LangChain tools through a single interface."

---

### A Survey of LLM-Driven AI Agent Communication: Protocols, Security Risks, and Defense Countermeasures (2025)

> *"This survey systematically deconstructs LLM agent systems through a methodology-centered taxonomy, linking architectural foundations, collaboration mechanisms."* — Authors
> — arXiv
> — [arXiv:2506.19676](https://arxiv.org/html/2506.19676v2)

**Key Contributions:**
- Categorizes agent communication into three classes: User-Agent, Agent-Agent, and Agent-Environment
- Proposes a three-layered communication architecture
- Deep analysis of security risks in memory and tool invocation modules

---

### Survey of LLM Agent Communication with MCP: A Software Design Pattern Centric Review (2025)

> *"This review consolidates recent advancements in large language model (LLM)-driven agentic AI, classical software design methodologies, and the emerging Model Context Protocol (MCP), with the goal of guiding the design of resilient and scalable inter-agent communication frameworks."* — Anjana Sarkar, Soumyendu Sarkar
> — arXiv
> — [arXiv:2506.05364](https://arxiv.org/pdf/2506.05364)

**Key Contributions:**
- Bridges classical software design patterns with MCP-based agent communication
- Examines how proven design patterns enhance reliability and scalability of LLM-driven agentic systems

---

### The Berkeley Function Calling Leaderboard (BFCL): From Tool Use to Agentic Evaluation of LLMs (2025)

> *"Function calling, also called tool use, refers to an LLM's ability to invoke external functions, APIs, or user-defined tools in response to user queries—an essential capability for agentic LLM applications."* — Shishir G. Patil et al.
> — ICML 2025 / Proceedings of Machine Learning Research 267
> — [OpenReview](https://openreview.net/forum?id=2GmDdhBdDk) | [Gorilla Leaderboard](https://gorilla.cs.berkeley.edu/leaderboard.html) | [GitHub](https://github.com/ShishirPatil/gorilla/tree/main/berkeley-function-call-leaderboard)

**Key Contributions:**
- Comprehensive benchmark evaluating function calling across simple, multiple, parallel, and multi-turn scenarios
- Evaluates Python, Java, JavaScript, SQL, and REST API calling
- Tracks live API evaluation for real-world accuracy

---

## Open-Source Libraries & Tools

### ToolRegistry

> [arXiv:2508.02979](https://arxiv.org/html/2508.02979v1) | `pip install toolregistry` (per paper)

Protocol-agnostic tool management library unifying registration, representation, execution, and lifecycle management. Supports Python functions, MCP servers, OpenAPI services, and LangChain tools under a single interface. Achieves 60–80% code reduction and up to 3.1× performance improvement via dual-mode concurrent execution engine.

---

### MCP Python SDK

> [modelcontextprotocol.io](https://modelcontextprotocol.io/docs/sdk) | `pip install mcp`

Official Python SDK for building Model Context Protocol servers and clients. Provides client-server interfaces for secure tool invocation and typed data exchange using JSON-RPC 2.0.

---

### MCP TypeScript SDK

> [github.com/modelcontextprotocol/typescript-sdk](https://github.com/modelcontextprotocol/typescript-sdk)

Tier-1 official SDK for building MCP servers and clients in TypeScript/JavaScript environments.

---

### Langfuse

> [langfuse.com](https://langfuse.com) | Open-source LLM engineering platform

Provides tracing and observability across multiple agent frameworks including LangGraph, OpenAI Agents SDK, CrewAI, AutoGen, Semantic Kernel, Mastra, and more.

---

### MCP-AgentBench Ecosystem

> [Hugging Face Datasets](https://huggingface.co/datasets/tuandunghcmut/BFCL_v4_information)

Evaluates agents in a standardized tool universe, enabling cross-framework agent compatibility testing.

---

## GitHub Repositories

| Repository | Stars | Description |
|-----------|-------|-------------|
| [langchain-ai/langchain](https://github.com/langchain-ai/langchain) | ⭐ 134k | The agent engineering platform — most widely adopted LLM application framework |
| [microsoft/autogen](https://github.com/microsoft/autogen) | ⭐ ~54k | Framework for creating multi-agent AI applications; merging into Microsoft Agent Framework |
| [openai/openai-agents-python](https://github.com/openai/openai-agents-python) | ⭐ 20.3k | Lightweight, powerful framework for multi-agent workflows by OpenAI |
| [microsoft/semantic-kernel](https://github.com/microsoft/semantic-kernel) | ⭐ ~27k | Integrate cutting-edge LLM technology into apps; being unified with AutoGen |
| [modelcontextprotocol/python-sdk](https://github.com/modelcontextprotocol/python-sdk) | — | Official Python SDK for Model Context Protocol |
| [modelcontextprotocol/typescript-sdk](https://github.com/modelcontextprotocol/typescript-sdk) | — | Official TypeScript SDK for MCP |
| [a2aproject/a2a-samples](https://github.com/a2aproject/a2a-samples) | ⭐ 1.4k | Samples using the Agent-to-Agent (A2A) Protocol |
| [awslabs/amazon-bedrock-agent-samples](https://github.com/awslabs/amazon-bedrock-agent-samples) | ⭐ 789 | Example notebooks and scripts for Amazon Bedrock Agents |
| [aws/bedrock-agentcore-sdk-python](https://github.com/aws/bedrock-agentcore-sdk-python) | — | Amazon Bedrock AgentCore SDK for deploying agents with any framework/model |
| [crewAIInc/crewAI](https://github.com/crewAIInc/crewAI) | — | Framework for orchestrating role-playing, autonomous AI agents |
| [agntcy/acp-spec](https://github.com/agntcy/acp-spec) | — | Agent Connect Protocol (ACP) specification |
| [ShishirPatil/gorilla](https://github.com/ShishirPatil/gorilla) | — | Gorilla LLM + Berkeley Function Calling Leaderboard |
| [Applied-Machine-Learning-Lab/Awesome-Function-Callings](https://github.com/Applied-Machine-Learning-Lab/Awesome-Function-Callings) | — | Awesome list for function calling in LLMs |
| [e2b-dev/awesome-ai-agents](https://github.com/e2b-dev/awesome-ai-agents) | — | Curated list of AI autonomous agents |
| [langchain-ai/langgraph](https://github.com/langchain-ai/langgraph) | — | Build resilient language agents as graphs |
| [langchain-ai/langchainjs](https://github.com/langchain-ai/langchainjs) | — | The agent engineering platform for JavaScript/TypeScript |

---

## Blog Posts & Articles

1. **Gravitee** — *"Google's Agent-to-Agent (A2A) and Anthropic's Model Context Protocol (MCP)"* — [link](https://www.gravitee.io/blog/googles-agent-to-agent-a2a-and-anthropics-model-context-protocol-mcp) — A2A focuses on agent-to-agent interaction; MCP standardizes how AI agents connect to external tools. Together they form the backbone of scalable, decentralized agentic AI.

2. **Auth0** — *"MCP vs A2A: A Guide to AI Agent Communication Protocols"* — [link](https://auth0.com/blog/mcp-vs-a2a/) — Each agent in an A2A system might use MCP to call their own tools; whether calling via MCP or delegating via A2A, identity and authorization are critical.

3. **Boomi** — *"What Is MCP, ACP, and A2A? AI Agent Protocols Explained"* — [link](https://boomi.com/blog/what-is-mcp-acp-a2a/) — MCP uses JSON-RPC 2.0 for communication between AI agents, LLMs, and internal business APIs. With native MCP support in 2025, organizations can deploy agents with standard connections.

4. **FastForward / boldstart.vc** — *"With the addition of Agentgateway, the Linux Foundation is becoming a hub for agentic AI infrastructure"* — [link](https://fastforward.boldstart.vc/with-the-addition-of-agentgateway-the-linux-foundation-is-becoming-a-hub-for-agentic-ai-infrastructure/) — Agentgateway provides drop-in security, observability and governance for agent-to-agent and agent-to-tool communication. A2A, AGNTCY, and Agentgateway are all now under the Linux Foundation.

5. **The New Stack** — *"Cisco Donates the AGNTCY Project to the Linux Foundation"* — [link](https://thenewstack.io/cisco-donates-the-agntcy-project-to-the-linux-foundation/) — AGNTCY moved to Linux Foundation to create open infrastructure for AI agent communication and prevent ecosystem fragmentation.

6. **p0stman** — *"AI Agent Frameworks 2026 \| OpenAI, Claude, Google, LangChain"* — [link](https://p0stman.com/ai-agent-frameworks) — MCP provides interoperability: an agent built with one framework can use MCP servers from another. A2A Protocol is the emerging agent-to-agent communication standard.

7. **Ry Walker Research** — *"Agent Frameworks Compared"* — [link](https://rywalker.com/research/agent-frameworks) — Comparative analysis of AutoGen, CrewAI, LangChain/LangGraph, LlamaIndex, Mastra, and Vercel AI SDK. Microsoft is consolidating AutoGen into the Microsoft Agent Framework.

8. **Langfuse** — *"Comparing Open-Source AI Agent Frameworks"* — [link](https://langfuse.com/blog/2025-03-19-ai-agent-comparison) — Comprehensive comparison covering LangGraph, OpenAI Agents SDK, Google ADK, Smolagents, CrewAI, AutoGen, Semantic Kernel, Agno, Mastra, and Microsoft Agent Framework.

9. **Turing** — *"A Detailed Comparison of Top 6 AI Agent Frameworks in 2026"* — [link](https://www.turing.com/resources/ai-agent-frameworks) — Deep dives into LangGraph, LlamaIndex, CrewAI, Microsoft Semantic Kernel, Microsoft AutoGen, and OpenAI Swarm with comparative analysis and pricing.

10. **futuresearch.ai** — *"The Hidden Incompatibilities Between LLM Providers"* — [link](https://futuresearch.ai/blog/llm-provider-quirks/) — Documents real incompatibilities in structured outputs and tool calling across OpenAI, Anthropic Claude, and Gemini APIs despite apparent convergence.

---

## Key Findings

> "No single protocol suffices for all contexts; instead, the authors propose a phased, complementary adoption strategy." — *A Survey of Agent Interoperability Protocols* (arXiv:2505.02279)

> "The tool-augmented LLM ecosystem is fragmented across four dimensions: protocol fragmentation, manual implementation overhead, complex execution workflow, and OpenAI dominance." — *Unified Tool Integration for LLMs* (arXiv:2508.02979)

> "MCP is about helping an AI agent figure out what tools it can use, what they do, and how to call them safely." — **Auth0 Blog**

> "MCP provides interoperability — an agent built with one framework can use MCP servers from another." — **p0stman**

1. **Four complementary protocols address distinct interoperability tiers**: MCP (tool access), ACP (multimodal messaging), A2A (collaborative task execution), and ANP (decentralized agent marketplaces). They are designed to work together, not replace each other.

2. **Microsoft is consolidating its agent frameworks**: AutoGen is merging into the unified **Microsoft Agent Framework**, combining ideas from both AutoGen and Semantic Kernel.

3. **Function calling formats remain incompatible across providers**: OpenAI, Anthropic Claude, and Google Gemini each use different JSON schema conventions, message formats, and tool invocation patterns despite apparent API convergence.

4. **MCP has emerged as the de facto USB-C for AI tools**: Built on JSON-RPC 2.0, it standardizes how applications deliver tools, datasets, and sampling instructions to LLMs. Major providers including OpenAI have adopted it.

5. **Linux Foundation is becoming the neutral governance hub**: A2A (donated by Google), AGNTCY (donated by Cisco), and Agentgateway (donated by Solo.io) are all now under the Linux Foundation umbrella to prevent ecosystem fragmentation.

6. **ToolRegistry demonstrates cross-SDK portability is achievable**: A protocol-agnostic approach can reduce integration code by 60–80% and improve performance by 3.1× by unifying Python functions, MCP servers, OpenAPI services, and LangChain tools.

7. **BFCL provides the most widely used benchmark for tool/function calling evaluation**: Covers simple, multiple, parallel, multi-turn, and live API scenarios across Python, Java, JavaScript, SQL, and REST APIs.

8. **SDK ecosystem is rapidly maturing but remains fragmented**: LangChain dominates with 134k stars, AutoGen has 54k, OpenAI Agents SDK has 20.3k, and Semantic Kernel has ~27k — but each uses different abstractions for agents, tools, and orchestration.

9. **Agent tracing and observability are becoming cross-framework necessities**: Tools like Langfuse now support 10+ frameworks, indicating the industry is standardizing on observability even while agent implementation remains heterogeneous.

10. **The industry is converging on a layered interoperability model**: MCP for agent-tool communication, A2A for agent-agent communication, and gateway layers (Agentgateway) for security, observability, and policy enforcement.
