# ASGI-Native

## 📌 Overview

ASGI (Asynchronous Server Gateway Interface) is the **async successor to WSGI** (PEP 3333), providing a standard interface between async web servers and Python applications. ASGI v3.0 was finalized March 20, 2019. It enables long-lived connections (WebSocket, HTTP/2), async event-driven messaging, and native coroutine-based applications — fundamentally different from WSGI's synchronous request-response model.

---

## Definitions & Standards

### ASGI Specification v3.0

> "This document proposes a standard interface between network protocol servers (particularly web servers) and Python applications, intended to allow handling of multiple common protocol styles (including HTTP, HTTP/2, and WebSocket)."

— [ASGI Specification](https://asgi.readthedocs.io/en/latest/specs/main.html)

> "ASGI consists of two different components: A protocol server, which terminates sockets and translates them into connections and per-connection event messages. An application, which lives inside a protocol server, is called once per connection, and handles event messages as they happen, emitting its own event messages back when necessary."

— [ASGI Specification](https://asgi.readthedocs.io/en/latest/specs/main.html)

> "Unlike WSGI, applications are asynchronous callables rather than simple callables, and they communicate with the server by receiving and sending asynchronous event messages rather than receiving a single input stream and returning a single iterable."

— [ASGI Specification](https://asgi.readthedocs.io/en/latest/specs/main.html)

> "ASGI applications must run as `async`/`await` compatible coroutines."

— [ASGI Specification](https://asgi.readthedocs.io/en/latest/specs/main.html)

### PEP 3333 — WSGI (Predecessor)

> "This specifies a standard interface between web servers and Python web applications/frameworks to promote portability. Separates choice of framework from choice of web server."

— [PEP 3333](https://peps.python.org/pep-3333/), Phillip J. Eby

---

## ASGI vs WSGI: Core Differences

| Aspect | WSGI | ASGI |
|--------|------|------|
| **Model** | Synchronous request-response | Asynchronous event-driven messages |
| **Callable** | `def application(environ, start_response)` | `async def application(scope, receive, send)` |
| **Long-lived connections** | ❌ Not supported | ✅ Native (WebSocket, HTTP/2, long-poll) |
| **Concurrency** | Multi-process/thread (GIL-limited) | Single-threaded async I/O |
| **Protocol support** | HTTP/1 only | HTTP/1.1, HTTP/2, WebSocket, QUIC, HTTP/3 |
| **Connection scope** | One request = one call | Lifetime of connection (many events) |

**ASGI Application Interface:**
```python
async def application(scope, receive, send):
    event = await receive()
    await send({"type": "websocket.send", ...})
```

**WSGI Application Interface:**
```python
def simple_app(environ, start_response):
    status = '200 OK'
    response_headers = [('Content-type', 'text/plain')]
    start_response(status, response_headers)
    return [b"Hello world!\n"]
```

---

## Key Sources

### Formal Specification & Documentation

**ASGI Specification (Primary Standard)**
> "The ASGI specification is the asynchronous successor to WSGI, providing a standard interface for async web applications."

— [asgi.readthedocs.io](https://asgi.readthedocs.io/en/latest/specs/main.html)

**django/asgiref — ASGI Specification and Utilities**
> "ASGI is a standard for Python asynchronous web apps and servers to communicate with each other, and positioned as an asynchronous successor to WSGI."

— [github.com/django/asgiref](https://github.com/django/asgiref)

**Django Channels ASGI Documentation**
> "ASGI is structured as a single asynchronous callable, which takes a dict `scope` and two callables `receive` and `send`."

— [channels.readthedocs.io](https://channels.readthedocs.io/en/latest/asgi.html)

### Academic Paper — IJSEA 2024 Systematic Review

**"Evaluative Comparison of ASGI Web Servers: A Systematic Review"**
*International Journal of Science and Engineering Applications, Volume 13, Issue 03, 2024*
**URL:** [PDF](https://www.ijsea.com/archive/volume13/issue3/IJSEA13031007.pdf)
**Author:** Anton Novikau (Head of Mobile Development, Talaera)
**DOI:** 10.7753/IJSEA1303.1007

> "ASGI attempts to preserve a simple application interface, while providing an abstraction that allows for data to be sent and received at any time, and from different application threads or processes."

> "The GIL is a part of CPython's memory management mechanic, it's a global mutex that doesn't allow native threads to execute Python bytecode simultaneously... it can lead to performance bottlenecks in multi-threaded applications."

> "ASGI lets applications receive and send asynchronous event messages as long as the connection is alive."

### Blog Posts & Explanation

> "ASGI (Asynchronous Server Gateway Interface) is the modern successor to WSGI (Web Server Gateway Interface). While WSGI (2003/2010) was built for Python 2.2-era synchronous behavior, ASGI enables multiple asynchronous events per application, supports both sync and async apps, and can handle long-lived connections like WebSocket—something WSGI cannot do."

— [InfoWorld — "ASGI explained: The future of Python web development"](https://www.infoworld.com/article/2335107/asgi-explained-the-future-of-python-web-development.html), Serdar Yegulalp

> "Any long-running call to a sync-only function will block the entire call chain, making the benefits of using async all but evaporate."

— [InfoWorld](https://www.infoworld.com/article/2335107/asgi-explained-the-future-of-python-web-development.html)

> "ASGI is for real-time apps. Django supports both — the real skill is knowing when to use which."

— [Vonage Developer Blog — "How Python's WSGI vs. ASGI is Like Baking a Cake"](https://developer.vonage.com/en/blog/how-wsgi-vs-asgi-is-like-baking-a-cake)

> "ASGI, the successor to WSGI, introduces asynchronous request handling."

— [Medium / ritheshvr](https://medium.com/@ritheshvr/wsgi-vs-asgi-the-real-difference-nobody-explains-properly-daa3b61247e1)

---

## ASGI Servers

### Uvicorn

**Repository:** [github.com/Kludex/uvicorn](https://github.com/Kludex/uvicorn)
**Documentation:** [uvicorn.dev](https://uvicorn.dev)
**License:** BSD

> "Until recently Python has lacked a minimal low-level server/application interface for async frameworks. The ASGI specification fills this gap, and means we're now able to start building a common set of tooling usable across all async frameworks."

> "Most well established Python Web frameworks started out as WSGI-based frameworks... A single, synchronous callable that takes a request and returns a response. This does not allow for long-lived connections."

— [Uvicorn GitHub README](https://github.com/Kludex/uvicorn)

**Supported Protocols:** HTTP/1.1, WebSockets

### Daphne

**Repository:** [github.com/django/daphne](https://github.com/django/daphne)
**Origin:** Django Channels
**Supported Protocols:** HTTP/1.1, HTTP/2, WebSockets

> "The first ASGI server implementation, originally developed to power Django Channels."

### Hypercorn

**Associated with:** Quart framework
**Supported Protocols:** HTTP/1.1, HTTP/2, WebSockets, QUIC, HTTP/3 (via plugin)

### Granian (Rust-based)

**URL:** [xtec.dev/python/granian](https://xtec.dev/python/granian)
**Repository:** [github.com/emmett-framework/granian](https://github.com/emmett-framework/granian)

> "Granian is a Rust-based HTTP server for Python applications. It supports ASGI/3, RSGI (for Rust), and WSGI interface applications."

> **Key advantage:** 2X+ faster than Gunicorn, lower worst-case latencies

---

## ASGI Frameworks

| Framework | Repository | Notes |
|-----------|------------|-------|
| **Starlette** | [github.com/Kludex/starlette](https://github.com/Kludex/starlette) | Lightweight ASGI toolkit/framework |
| **FastAPI** | [github.com/fastapi/fastapi](https://github.com/tiangolo/fastapi) | Built on Starlette |
| **Quart** | [github.com/pallets/quart](https://github.com/pallets/quart) | Async Flask reimplementation |
| **Django Channels** | [github.com/django/channels](https://github.com/django/channels) | Real-time for Django |
| **BlackSheep** | [github.com/Neusofteng/blacksheep](https://github.com/Neusofteng/blacksheep) | ASGI-native framework |

**Starlette Description:**
> "Starlette is a lightweight ASGI framework/toolkit, which is ideal for building async web services in Python."

> "It is production-ready and designed to be used either as a complete framework or as an ASGI toolkit via independent components."

— [starlette.dev](https://starlette.dev/)

**Quart Description:**
> "Quart is an async Python web application framework... an asyncio reimplementation of the popular Flask web application framework."

> "This means that if you understand Flask you understand Quart."

— [github.com/pallets/quart](https://github.com/pallets/quart)

---

## Debates & Controversies

### ASGI Overhead for Simple Applications

> "There is ~15ms overhead for the same operation (same view) when using ASGI - laboratory conditions with one client and one request processing just to show minimal latency potential of Django."

> "For a synchronous view and ASGI, Django runs the view's synchronous code in a thread pool using ThreadPoolExecutor."

**Counter-argument:** ASGI provides necessary infrastructure for WebSockets and HTTP/2; overhead is justified for I/O-bound applications.

### Async Python Adoption

> "Depends. If your app is not I/O bound then using ASGI or Async wouldn't give you much benefit. Django was created as sync first, then Async APIs were added later."

— [Reddit Discussion: "Any reason not to start all new projects with ASGI/async?"](https://www.reddit.com/r/Python/comments/1example/any_reason_not_to_start_all_new/)

### Sync/Async Interoperability Complexity

> "The idea is to make it easier to call synchronous APIs from async code and asynchronous APIs from synchronous code so it's easier to transition code from one style to the other."

— [asgiref documentation](https://github.com/django/asgiref)

**Warning:**
> "Any long-running call to a sync-only function will block the entire call chain, making the benefits of using async all but evaporate."

— [InfoWorld](https://www.infoworld.com/article/2335107/asgi-explained-the-future-of-python-web-development.html)

### Threading vs Async

ASGI's async model is optimized for I/O-bound workloads; CPU-bound work may perform worse due to async overhead without parallelism benefits.

---

## Summary Table

### ASGI Server Benchmarks (IJSEA 2024)

| Server | RPS (22B body) | Memory (MB) | RPS (430KB body) | Protocols |
|--------|---------------|-------------|-----------------|-----------|
| **Uvicorn** | **4,253** | **79.4** | **2,460** | HTTP/1.1, WS |
| Hypercorn | 3,112 | 112.8 | 2,021 | HTTP/1.1, HTTP/2, WS, QUIC, HTTP/3 |
| Daphne | 3,073 | 126.7 | 1,039 | HTTP/1.1, HTTP/2, WS |

*Source: [IJSEA Vol 13, Issue 3, 2024 — "Evaluative Comparison of ASGI Web Servers"](https://www.ijsea.com/archive/volume13/issue3/IJSEA13031007.pdf)*

### Native Async Patterns (ASGI-Native Characteristics)

| Pattern | Description |
|---------|-------------|
| **Event-driven scope** | Connection scope persists across multiple events (vs. WSGI one-shot) |
| **Awaitable receive/send** | Both `receive()` and `send()` are awaitable coroutines |
| **Protocol diversity** | HTTP/1.1, HTTP/2, WebSocket, QUIC, HTTP/3 within single interface |
| **Coroutine-based app** | Application must be `async def` compatible with `asyncio` |
| **Lifespan events** | `lifespan` events for startup/shutdown handling |

### Key Historical Timeline

| Year | Event |
|------|-------|
| 2003 | PEP 333 — WSGI v1.0 introduced |
| 2010 | PEP 3333 — WSGI updated for Python 3 |
| 2012 | Django Channels project begins (Andrew Godwin) |
| 2016 | Django Channels 1.0 released; ASGI spec drafted |
| **2019** | **ASGI v3.0 specification finalized (March 20, 2019)** |
| 2016–2024 | Uvicorn, Daphne, Hypercorn, Starlette, FastAPI, Quart mature |

---

## Related Topics

- [FastAPI 2026](./FastAPI-2026.md) — Built on Starlette (ASGI), runs on Uvicorn
- [Pydantic V3](./Pydantic-V3.md) — Pydantic async validators pair with ASGI async handlers
- [MCP-Protocol](../MCP-Protocol/README.md) — ASGI event model parallels MCP request/response patterns
