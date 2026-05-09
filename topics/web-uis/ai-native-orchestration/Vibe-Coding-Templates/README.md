# Vibe-Coding Templates

## 📌 Overview

> **Vibe-Coding Templates** are starter kits, Cursor Rules, MCP server configurations, and workflow scaffolds that encode best practices for AI-assisted coding — making vibe-coding reproducible, team-shareable, and aligned with project conventions.

---

## 📚 Key Concepts

### What is Vibe Coding?
**Source:** [roadmap.sh/vibe-coding](https://roadmap.sh/vibe-coding/best-practices)

> *"Vibe coding is a human-AI collaborative approach to building software where the human provides high-level direction and intent while AI handles the implementation details. The human reviews and guides, but doesn't write code line-by-line."*

### The Four Pillars of Vibe Coding
**Source:** [aiagentskit/vibe-coding-starter-kit](https://github.com/aiagentskit/vibe-coding-starter-kit)

| Pillar | Description |
|--------|-------------|
| **Context** | Comprehensive project background — tech stack, architecture, conventions |
| **Atomic Tasks** | Break complex features into tiny, reviewable, reversible steps |
| **Memory** | Persistent project state across AI sessions (summaries, decisions, TODOs) |
| **Patterns** | Reusable solution templates for common engineering problems |

### Best Practices Checklist
**Source:** [roadmap.sh/vibe-coding/best-practices](https://roadmap.sh/vibe-coding/best-practices)

1. Use plan mode before writing any code
2. Enforce atomic commits — one feature/fix/refactor per commit
3. Review every AI suggestion before accepting
4. Use git branches for every new feature
5. Maintain a project context file updated after each session
6. Define clear acceptance criteria before prompting
7. Use type systems and linters as safety nets
8. Test incrementally — don't wait until the end
9. Keep prompts focused and specific
10. Document your AI interactions and decisions

---

## 🛠️ Key Templates & Starters

### AI Agents Kit — Vibe Coding Starter Kit
**URL:** [github.com/aiagentskit/vibe-coding-starter-kit](https://github.com/aiagentskit/vibe-coding-starter-kit)

> *"A documentation system with AI agents that learns your codebase, follows your standards, and accelerates your workflow."*

**Four Pillars Implementation:**
- Context: `CONTEXT.md` — comprehensive project overview
- Atomic Tasks: `TASKS/` — micro-tasks with clear acceptance criteria
- Memory: `MEMORY.md` — persistent session summaries
- Patterns: `PATTERNS/` — reusable solution templates

---

### HumanStack — NextJS + FastAPI + Supabase Template
**URL:** [github.com/humanstack/vibe-coding-template](https://github.com/humanstack/vibe-coding-template)

> *"A curated vibe coding template for NextJS + FastAPI + Supabase with Cursor Rules, pre-configured for seamless AI-assisted development."*

**Includes:**
- Next.js 14 App Router setup
- FastAPI backend scaffold
- Supabase integration
- Pre-configured Cursor Rules
- Authentication flow
- API route examples

---

### Cursor Pre-configured MCP Servers
**URL:** [github.com/jpke/cursor-vibe-coding-template](https://github.com/jpke/cursor-vibe-coding-template)

> *"Pre-configured MCP servers and Cursor Rules for vibe coding workflows."*

**Supported MCP Servers:**
- File system operations
- Git operations
- Search and replace
- Browser automation
- Custom project-specific servers

---

### KhazP — PRD → Tech Design → Build Workflow
**URL:** [github.com/KhazP/vibe-coding-prompt-template](https://github.com/KhazP/vibe-coding-prompt-template)

> *"5-step workflow: PRD → Clarifying Questions → Technical Design → Implementation → Testing."*

**Workflow Steps:**
1. **PRD Input** — Define what to build
2. **Clarifying Questions** — AI asks about edge cases and requirements
3. **Technical Design** — AI produces `SPEC.md`
4. **Implementation** — Build in atomic commits
5. **Testing** — Verify against acceptance criteria

---

### Hscspring — AI-Friendly Project Scaffolding
**URL:** [github.com/hscspring/create-vibe-app](https://github.com/hscspring/create-vibe-app)

> *"AI-friendly project scaffolding — create projects structured for AI comprehension and modification."*

**Principles:**
- Consistent file naming conventions
- Clear module boundaries
- Explicit dependency graphs
- Self-documenting code structure

---

### Backblaze B2 — Vibe Coding Starter Kit
**URL:** [github.com/backblaze-b2-samples/vibe-coding-starter-kit](https://github.com/backblaze-b2-samples/vibe-coding-starter-kit)

> *"Full-stack TypeScript + Python with B2 storage — vibe coding template for rapid prototyping."*

---

## 📋 Cursor Rules

### What Are Cursor Rules?
**Source:** [cursor.sh/rules](https://cursor.sh/rules)

Cursor Rules encode project conventions, architecture decisions, and coding standards as **machine-readable instructions** that guide Cursor's AI behavior per-project.

### Example Cursor Rule Structure
```markdown
# Project Context
- Tech stack: Next.js 14, TypeScript, Prisma, PostgreSQL
- Code style: Functional components, TypeScript strict mode
- Testing: Vitest + React Testing Library
- Linting: ESLint + Prettier

# Architecture
- `/app` — Next.js pages and API routes
- `/components` — Reusable UI components
- `/lib` — Utility functions and Prisma client
- `/prisma` — Database schema and migrations

# Conventions
- All components must have prop types
- API routes return standardized JSON shape
- Database mutations go through Prisma transactions
```

---

## 🔧 Tool Comparison

**Source:** [Till Freitag's comparison](https://roadmap.sh/vibe-coding/best-practices)

| Tool | Best For | Limitations |
|------|----------|-------------|
| **Cursor** | Developers who want AI pair programmer | Desktop app, local setup |
| **Lovable** | Non-technical founders rapid MVP | Server-side code limited |
| **Bolt** | Fast prototyping, browser-only | WASM limitations for some Node modules |
| **Replit** | Beginners, education | Vendor lock-in |
| **Emergent** | Full autonomous pipelines | Complex projects need oversight |

---

## ⚠️ Key Challenges & Risks

| Risk | Description | Mitigation |
|------|-------------|-----------|
| **Context loss** | AI loses project context in long sessions | Keep `CONTEXT.md` updated, atomic commits |
| **Hallucination** | AI invents non-existent APIs | Use type systems, linters, explicit specs |
| **Code quality drift** | Speed over best practices | Enforce linting, code review per AI suggestion |
| **Security** | AI may generate insecure code | Security audits, dependency scanning |
| **Lock-in** | Generated code tied to specific platform | Use portable templates, export to GitHub |

---

## 🔗 Related Documents

← Back to [ai-native-orchestration](./README.md)
