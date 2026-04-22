# Pydantic V3

## 📌 Overview

Pydantic is the most widely used Python data validation library, playing a core role in FastAPI's type-safety and data validation layer. As of April 2026, **V3 has not been released yet** — current stable is **2.13.3** (released April 20 2026). V3 is planned as a minor evolution rather than a V1→V2-scale breaking change, primarily removing V1 legacy helpers and potentially stabilizing the experimental **BaseStruct** type.

---

## Definitions & Standards

> "Pydantic is the most widely used data validation library for Python. Fast and extensible, Pydantic plays nicely with"

— [pydantic.dev](https://pydantic.dev)

> "There will not be another breaking change of this magnitude!" — Pydantic V1→V2 migration warning acknowledged by maintainers

— [Pydantic Version Policy](https://pydantic.dev/docs/validation/latest/get-started/version-policy/)

> "V3 will not involve any large changes to the API as V2 did! Instead V3 will: remove the V1 migration helpers including deprecated methods"

— [GitHub Issue #10033](https://github.com/pydantic/pydantic/issues/10033), Samuel Colvin (maintainer), Aug 2 2024

> "The high-level goal is to take some of the lessons we've learned from `BaseModel`, and present a new type which should achieve higher performance, more features and better default semantics... We anticipate that Pydantic V3 may be the milestone where `BaseStruct` is stabilised."

— [GitHub Issue #10032](https://github.com/pydantic/pydantic/issues/10032), Samuel Colvin, Aug 2 2024

---

## Key Sources

### Official Documentation

**Migration Guide (V1 → V2)**
> "Pydantic V2 is now the current production release of Pydantic... Pydantic V1 is still available when you need it, though we recommend migrating to Pydantic V2 for its improvements and new features."

— [pydantic.dev/docs/validation/latest/get-started/migration/](https://pydantic.dev/docs/validation/latest/get-started/migration/)

**Version Policy**
> "V1: Active development stopped. Critical bug fixes and security patches continue until the release of Pydantic V3. V2: Current stable version. No intentional breaking changes in minor releases. V3+: Expected roughly once a year. Breaking changes will be trivial compared to V1→V2."

— [pydantic.dev Version Policy](https://pydantic.dev/docs/validation/latest/get-started/version-policy/)

### V3 Roadmap

**Issue #10033 — V3 Scope (Samuel Colvin, Aug 2 2024)**
> "TL;DR: V3 will not involve any large changes to the API as V2 did! Instead V3 will: remove the V1 migration helpers including deprecated methods"

— [github.com/pydantic/pydantic/issues/10033](https://github.com/pydantic/pydantic/issues/10033)

**Issue #10032 — BaseStruct Evolution**
> "The high-level goal is to take some of the lessons we've learned from `BaseModel`, and present a new type which should achieve higher performance, more features and better default semantics... We anticipate that Pydantic V3 may be the milestone where `BaseStruct` is stabilised."

— [github.com/pydantic/pydantic/issues/10032](https://github.com/pydantic/pydantic/issues/10032)

### Performance

**Rust Core Performance**
> "In Pydantic v2, the core validation engine is implemented in **Rust**, making it one of the fastest data validation solutions in the Python ecosystem."

— [Mike Huls](https://mikehuls.com/pydantic-performance-4-tips-on-how-to-validate-large-amounts-of-data-efficiently/), 2024

**Benchmark Repository**
> "Benchmark performance and explore the new features of various releases of Pydantic."

— [prrao87/pydantic-benchmarks](https://github.com/prrao87/pydantic-benchmarks)

### Academic / Research Usage

> "By using Pydantic's clearly defined schema (KeywordResult), keyword outputs remain structured and consistent, making it simple for downstream agents to consume this data without ambiguity."

— [Analytics Vidhya — Multi-Agent Research Assistant](https://www.analyticsvidhya.com/blog/2025/03/multi-agent-research-assistant-system-using-pydantic/), March 2025

> "While Pydantic provides runtime validation, our framework aims to support greater flexibility."

— [arXiv:2410.18146](https://arxiv.org/pdf/2410.18146), October 2024

---

## Repositories & Tools

| Repository | URL | Description |
|------------|-----|-------------|
| [pydantic/pydantic](https://github.com/pydantic/pydantic) | GitHub | Main library |
| [pydantic/bump-pydantic](https://github.com/pydantic/bump-pydantic) | GitHub | Migration tool V1→V2 |
| [prrao87/pydantic-benchmarks](https://github.com/prrao87/pydantic-benchmarks) | GitHub | Performance benchmarking |
| [pydantic/pydantic-ai](https://github.com/pydantic/pydantic-ai) | GitHub | AI agent framework built on Pydantic |

---

## Debates & Controversies

### V1→V2 Migration Pain

> "V1 -> V2 migration was a disaster and broke my trust in the project. Does not play nicely with NumPy or common scientific tools."

— [Reddit r/Python](https://www.reddit.com/r/Python/comments/1kqw4t5/pydantic_v3_roadmap_and_the_future_of_python_data/)

**Counter from maintainers:**
> "There will not be another breaking change of this magnitude!"

### Deprecation Overload

> "Pydantic has too much deprecation, making it difficult to keep up with updates and maintaining the code. Lots of functionality has been renamed, and some are removed during v1→v2 transition. Even sample code from November 2023 is deprecated now!"

— [Reddit r/Python](https://www.reddit.com/r/Python/comments/1kqw4t5/pydantic_v3_roadmap_and_the_future_of_python_data/)

### Performance vs msgspec

> "msgspec decodes ~6.5x faster than Pydantic V2. msgspec decodes ~30x faster than Pydantic V1."

— Various benchmark comparisons

**Counter-view:**
> "msgspec is an extremely fast serialization and validation library that consistently outperforms Pydantic v2... Pydantic v2 is a feature-rich framework."

### Breaking Changes in Minor Releases

> "Well, it is a breaking change on a minor revision, but Python doesn't use semantic versioning so there's no reason that can't happen."

**Pydantic's response:**
> "We will not intentionally make breaking changes in minor releases of V2. Functionality marked as deprecated will not be removed until the next major V3 release."

— [Reddit discussion](https://www.reddit.com/r/Python/comments/1kqw4t5/pydantic_v3_roadmap_and_the_future_of_python_data/)

---

## Summary Table

### Version Status

| Version | Status | Release Date | Key Info |
|---------|--------|-------------|----------|
| V1 | End-of-life (security patches only) | 2018 | Original validation library |
| V2 | **Current stable (2.13.3)** | 2023 | Rust core, 5–50x faster than V1 |
| **V3** | **Planned, not yet released** | ~2026 (expected) | Remove V1 legacy; BaseStruct stabilization |

### Migration V1 → V2 Key Changes

| V1 | V2 |
|----|----|
| `parse_obj()` | `model_validate()` |
| `dict()` | `model_dump()` |
| `json()` | `model_dump_json()` |
| `construct()` | `model_construct()` |
| `from_orm` | `model_validate` with `from_attributes=True` |

### V3 Expected Changes

- Remove `pydantic.v1` namespace
- Remove deprecated V1 migration helpers
- Fix edge cases considered broken in V2
- Potential stabilization of **BaseStruct** (5–10x improvement goal over V2)
- Python V1 support ends with V3 release

### Performance Comparison

| Version | Core | Relative Speed |
|---------|------|---------------|
| V1 | Python | Baseline (1x) |
| V2 | Rust | 5–50x faster than V1 |
| V3 (target) | Rust + BaseStruct | 5–10x faster than V2 |

---

## Related Topics

- [FastAPI 2026](./FastAPI-2026.md) — FastAPI uses Pydantic as its data validation layer
- [ASGI-Native](./ASGI-Native.md) — ASGI async patterns that complement Pydantic's async validators
- [MCP-Protocol](../MCP-Protocol/README.md) — Tool/discovery schemas similar to Pydantic model usage
