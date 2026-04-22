# API-Servers

> Protocols, frameworks, and infrastructure for serving, connecting, and exposing LLM capabilities via standardized API interfaces.

## Overview

This folder covers protocols and frameworks for LLM API servers, including MCP (Model Context Protocol), inference engines, and related connection standards that enable LLMs to interact with external tools, data, and services.

---

## Subtopics

| Folder | Description |
|--------|-------------|
| [MCP-Protocol/](./MCP-Protocol/) | Model Context Protocol — open standard for connecting LLMs to tools, data, and services |
| [RealTime-MultiModal/](./RealTime-MultiModal/) | Real-time multimodal AI infrastructure — WebRTC, WebSockets, and stateful session management |

---

## MCP-Protocol Subtopics

| Document | Description |
|----------|-------------|
| [FastMCP](./MCP-Protocol/FastMCP.md) | Python framework for building MCP servers (1M+ PyPI downloads/day) |
| [Standard-Connectors](./MCP-Protocol/Standard-Connectors.md) | Official transports (stdio, Streamable HTTP), SDKs, company servers |
| [Tool-Discovery](./MCP-Protocol/Tool-Discovery.md) | How tools are discovered in MCP — protocol, registry, web discovery |

---

## RealTime-MultiModal Subtopics

| Document | Description |
|----------|-------------|
| [WebRTC](./RealTime-MultiModal/WebRTC.md) | UDP-based real-time audio/video transport — sub-500ms, browser-native |
| [WebSockets](./RealTime-MultiModal/WebSockets.md) | Persistent bidirectional connections for AI streaming and agent communication |
| [Stateful-Sessions](./RealTime-MultiModal/Stateful-Sessions.md) | Session management, memory architectures, context overflow solutions |
