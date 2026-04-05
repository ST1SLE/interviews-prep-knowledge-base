# OpenSpec — Environment Setup & Initialization

Installation, `openspec init`, project introspection, and the AGENTS.md context checklist.

**Prerequisite:** Read [01_fundamentals.md](01_fundamentals.md) first.

---

## 1. Prerequisites

| Requirement | Details |
|-------------|---------|
| Node.js | v20.19.0 or later |
| npm | Comes with Node.js |
| Git | Any recent version — OpenSpec is Git-native |
| AI coding assistant | Claude Code, Cursor, Windsurf, Copilot, or OpenCode |
| A test repository | You'll create one in this module |

---

## 2. Installation

```bash
npm install -g @fission-ai/openspec@latest
```

Verify:

```bash
openspec --version
```

This installs the OpenSpec CLI globally. It's a TypeScript-based tool — lightweight, no proprietary databases or abstraction layers. Everything it generates is standard, Git-trackable Markdown.

---

## 3. Initialize a Test Project

Create a playground project to learn with:

```bash
mkdir sdd-playground && cd sdd-playground
git init
```

Create a minimal project to have something for OpenSpec to introspect. For example, a simple FastAPI app:

```bash
# Create a basic project structure
mkdir -p src tests
```

```python
# src/main.py
from fastapi import FastAPI

app = FastAPI(title="SDD Playground")

@app.get("/")
def root():
    return {"message": "Hello, SDD"}

@app.get("/items/{item_id}")
def get_item(item_id: int):
    return {"item_id": item_id, "name": f"Item {item_id}"}
```

```
# requirements.txt
fastapi>=0.100.0
uvicorn>=0.23.0
```

Commit this baseline:

```bash
git add -A && git commit -m "Initial project structure"
```

Now initialize OpenSpec:

```bash
openspec init
```

---

## 4. What `openspec init` Creates

After initialization, your project gains this structure:

```
sdd-playground/
├── src/
├── tests/
├── requirements.txt
└── openspec/
    ├── specs/           ← Source of truth: permanent, evolving specifications
    ├── changes/         ← Active proposals: transient, per-change artifacts
    │   └── archive/     ← Completed proposals: immutable audit trail
    ├── config.yaml      ← OpenSpec configuration
    └── AGENTS.md        ← Context checklist for AI agents
```

| Directory | Purpose | Lifecycle |
|-----------|---------|-----------|
| `specs/` | The unified specification library. Organized by capability. Grows over time as changes are archived. | Permanent, ever-evolving |
| `changes/` | Working directory for active proposals. Each change gets its own subdirectory with 4 delta artifacts. | Transient — cleared after archiving |
| `changes/archive/` | Historical audit trail. Completed proposals move here (timestamped: `YYYY-MM-DD-<name>/`). | Immutable — never modified |

---

## 5. Project Context via AGENTS.md

OpenSpec captures project context primarily through `AGENTS.md` — the file that grounds the AI's understanding of your project. After `openspec init`, you should trigger **AI introspection** to enrich it with project-specific details.

**How to populate it:**

In your AI coding assistant (e.g., Claude Code), prompt:

```
Introspect my current project and update openspec/AGENTS.md with project context
```

The AI will analyze your codebase and capture:

- **Tech stack** — languages, frameworks, dependencies
- **Coding conventions** — naming, file organization, import patterns
- **Architectural constraints** — how components interact, data flow
- **Security boundary rules** — authentication patterns, input validation
- **Testing approach** — framework, coverage expectations

**Why it matters:** Every future AI interaction with this project is bounded by `AGENTS.md`. It prevents the AI from suggesting technologies you don't use, patterns that violate your conventions, or architectures that contradict your constraints.

**Review it carefully.** If the AI hallucinated something (e.g., claimed you use a database you don't have), correct it. This file is a living document — update it as your project evolves.

> **Note:** Some SDD workflows recommend a separate `project.md` fingerprint file. OpenSpec consolidates this into `AGENTS.md` and `config.yaml`. If your team prefers a separate project fingerprint, you can create one manually — just ensure `AGENTS.md` references it.

---

## 6. AGENTS.md — The Context Checklist

`AGENTS.md` is a persistent system prompt — a "Context Checklist" that the AI agent reads at the start of every session.

Its critical function: **force the AI to read existing specifications before writing any code.**

Typical AGENTS.md content instructs the agent to:

1. Read project context (stack, conventions, constraints) from `AGENTS.md` itself
2. Check `openspec/specs/` for existing capabilities relevant to the current task
3. Check `openspec/changes/` for any active proposals
4. Only then proceed with the requested work

**Why this is paramount:**

Without this checklist, every new AI session starts with zero memory. The agent doesn't know what was built before, what architectural decisions were made, or what capabilities already exist. It will hallucinate overlapping features, contradict prior designs, or reinvent existing functionality.

The AGENTS.md checklist creates a **closed-loop knowledge retention system** — the AI reads the accumulated specifications, maintains context continuity, and builds on top of validated history rather than starting fresh.

---

## Hands-On Exercise

1. **Install OpenSpec** following section 2
2. **Create the test project** from section 3 (or use any small project you have)
3. **Run `openspec init`** and verify the directory structure matches section 4
4. **Trigger introspection** — prompt your AI assistant to enrich `AGENTS.md` with project context
5. **Review `AGENTS.md`** — is it accurate? Does it correctly describe your tech stack and conventions? Fix any hallucinations.

**Success criteria:**
- `openspec/` directory exists with `specs/`, `changes/` (including `changes/archive/`) subdirectories
- `AGENTS.md` contains a context checklist and accurate project description (no hallucinations)
- You understand what `AGENTS.md` does and what problem it solves

---

**Next:** [03_core_lifecycle.md](03_core_lifecycle.md) — The Propose → Apply → Archive workflow
