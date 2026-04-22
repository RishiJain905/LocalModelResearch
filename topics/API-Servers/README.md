# API-Servers

> Protocols, frameworks, and infrastructure for serving, connecting, and exposing LLM capabilities via standardized API interfaces.

## Overview

This folder covers protocols and frameworks for LLM API servers, including MCP (Model Context Protocol), inference engines, and related connection standards that enable LLMs to interact with external tools, data, and services.

---

## Subtopics

| Folder | Description |
|--------|-------------|
| [MCP-Protocol/](./MCP-Protocol/) | Model Context Protocol — open standard for connecting LLMs to tools, data, and services |

---

## MCP-Protocol Subtopics

| Document | Description |
|----------|-------------|
| [FastMCP](./MCP-Protocol/FastMCP.md) | Python framework for building MCP servers (1M+ PyPI downloads/day) |
| [Standard-Connectors](./MCP-Protocol/Standard-Connectors.md) | Official transports (stdio, Streamable HTTP), SDKs, company servers |
| [Tool-Discovery](./MCP-Protocol/Tool-Discovery.md) | How tools are discovered in MCP — protocol, registry, web discovery |
