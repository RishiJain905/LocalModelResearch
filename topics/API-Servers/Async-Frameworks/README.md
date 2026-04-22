# Async-Frameworks

## 📌 Overview

Async-Frameworks covers the three pillars of modern Python async web development: **FastAPI** (the leading async API framework), **Pydantic V3** (the data validation backbone), and **ASGI-Native** (the underlying async server specification). Together these form the stack powering high-performance Python web services in 2026.

---

## Subtopics

| Subtopic | Description | Key Metric |
|----------|-------------|-----------|
| [FastAPI 2026](./FastAPI-2026.md) | High-performance async API framework, built on Starlette + Pydantic | **97.2k stars**, 38% Python dev adoption |
| [Pydantic V3](./Pydantic-V3.md) | Data validation library — Rust-core, V3 upcoming with BaseStruct | **2.13.3** current; V3 planned |
| [ASGI-Native](./ASGI-Native.md) | Async Server Gateway Interface spec (v3.0, 2019) — the async foundation | **4,253 RPS** (Uvicorn benchmark) |

---

## Summary Matrix

| | FastAPI 2026 | Pydantic V3 | ASGI-Native |
|---|---|---|---|
| **Role** | API framework | Data validation | Server interface spec |
| **Built on** | Starlette + Pydantic | — | ASGI v3.0 spec |
| **Key Feature** | Auto-docs, async, type hints | Rust-core validation | Event-driven, WebSocket |
| **Stars/Adoption** | 97.2k GitHub stars | ~most-used Python lib | Standard (2019) |
| **Latest Version** | 0.136.0 (Apr 2026) | 2.13.3 (Apr 2026) | v3.0 (2019, stable) |
| **Primary Use** | REST/Web APIs | Schema/validation | HTTP/WS servers |

---

## Key Interconnections

```
ASGI-Native (Server Interface Spec)
    ↓ powers
Starlette (ASGI toolkit)
    ↑ uses
FastAPI (API Framework)
    ↑ uses
Pydantic (Data Validation)
    ↓ V3 adds
BaseStruct (experimental, 5-10x faster)
```

- **FastAPI** depends on **Starlette** (ASGI) and **Pydantic** (validation)
- **ASGI-Native** defines the `scope`/`receive`/`send` interface that both **FastAPI** and **Starlette** implement
- **Pydantic V3** (planned) will further optimize FastAPI's validation path via BaseStruct

---

## Related Topics

- [MCP-Protocol](../MCP-Protocol/README.md) — Tool schemas and async request/response patterns parallel to ASGI
- [API-Servers parent](../README.md) — Back to API-Servers overview
