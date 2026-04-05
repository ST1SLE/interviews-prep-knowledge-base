# OpenSpec & SDD — Cheatsheet

Quick reference for daily use. For explanations, see the learning modules.

---

## Commands

| Command | Phase | What it does |
|---------|-------|-------------|
| `npm install -g @fission-ai/openspec@latest` | Setup | Install OpenSpec CLI globally |
| `openspec init` | Setup | Scaffold `openspec/` directory structure |
| `/openspec:proposal` or `/opsx:propose` | Propose | Generate delta spec artifacts for a change |
| `/openspec:apply` or `/opsx:apply` | Apply | Execute implementation strictly bound to tasks.md |
| `/openspec:archive` or `/opsx:archive` | Archive | Merge deltas into specs/, move proposal to archive/ |
| `/opsx:explore` | Brownfield | Investigate legacy code — analysis only, no code changes |
| `/opsx:continue` | Brownfield | Advance explore results through propose/design/tasks flow |
| `openspec schema init` | Customization | Scaffold custom schema configuration |
| `openspec validate --all --strict` | CI/CD | Validate all spec files (headless, for automation) |
| `openspec validate --all --strict --no-interactive --json` | CI/CD | Headless validation with JSON output for GitHub Actions |

---

## Directory Structure

```
openspec/
├── specs/           ← Source of truth (permanent, evolving)
│   └── <capability>.md
├── changes/         ← Active proposals (transient, cleared after archive)
│   └── <change-name>/
│       ├── proposal.md
│       ├── specs.md
│       ├── design.md
│       └── tasks.md
├── archive/         ← Completed proposals (immutable audit trail)
│   └── <change-name>/
│       ├── proposal.md
│       ├── specs.md
│       ├── design.md
│       └── tasks.md
├── schemas/         ← Custom workflow schemas (optional)
│   └── schema.yaml
├── project.md       ← Project fingerprint (~250 lines)
└── AGENTS.md        ← Context checklist for AI agents
```

---

## Delta Artifacts

Generated per proposal inside `openspec/changes/<change-name>/`:

| File | Answers | Content |
|------|---------|---------|
| `proposal.md` | **Why?** | Business motivation, rationale |
| `specs.md` | **What?** | Functional requirements, acceptance criteria |
| `design.md` | **How?** | Technical architecture, algorithm choices |
| `tasks.md` | **Steps?** | Deterministic execution checklist with checkboxes |

---

## Delta Markup

Used in `specs.md` to denote changes to existing capabilities:

| Markup | Meaning |
|--------|---------|
| `ADDED` | Entirely new capability |
| `MODIFIED` | Change to an existing capability |
| `REMOVED` | Capability being removed |

---

## Workflow Quick Reference

```
1. /opsx:propose "description of change"
   → Review: proposal.md, specs.md, design.md, tasks.md
   → Edit artifacts if needed

2. /opsx:apply
   → Agent follows tasks.md strictly
   → Deviation? Halt → update spec → resume

3. /opsx:archive
   → changes/ moves to archive/
   → Delta specs merge into specs/
   → Done. Repeat for next change.
```

---

## Key Files

| File | Purpose | When to update |
|------|---------|---------------|
| `project.md` | AI's understanding of your project (stack, conventions, constraints) | After major architectural changes |
| `AGENTS.md` | Forces AI to read specs before writing code | Rarely — usually set once at init |
| `config.yaml` | Root-level OpenSpec configuration | When adding custom schemas |
| `schema.yaml` | Per-project schema definition (artifact types, templates, dependencies) | When customizing the pipeline |

---

## Alignment Levels

| Level | Name | Spec lifecycle | Maturity |
|-------|------|---------------|----------|
| 1 | Spec-First | Written before code, abandoned after | Entry |
| 2 | Spec-Anchored | Retained, used for maintenance and regression | Intermediate |
| 3 | Spec-as-Source | Spec IS the product, code is generated from it | Advanced |

---

## Schema Types

For customizing the artifact pipeline (advanced — see `openspec schema init`):

| Schema | Generated Artifacts | Validation | Best for |
|--------|-------------------|------------|----------|
| **Minimalist** | specs.md, tasks.md | Given/When/Then acceptance testing | Low-risk changes, UI tweaks, prototyping |
| **Default** | proposal.md, specs.md, design.md, tasks.md | Standard verification checklist + unit tests | Standard feature development |
| **Event-Driven** | Event Storming, Event Modeling (Mermaid.js), asyncapi.yaml, tasks.md | Rigid AsyncAPI YAML contract validation | Microservices, Kafka/RabbitMQ, distributed systems |

Schema resolution order: project-level `openspec/schemas/` overrides global `~/.openspec/schemas/`.

---

## Core Principles

1. **Specs constrain AI, not prompts** — the specification is the deterministic control mechanism
2. **Unarchived changes = technical debt** — always archive after successful Apply
3. **Unified > fragmented** — one consolidated spec library beats scattered per-feature docs
4. **Agent follows tasks.md strictly** — deviation requires halting and updating the spec
5. **Human reviews before Apply** — the Propose phase is your quality gate
6. **Separate "what" from "how"** — specs define the contract, code implements it

---

## Brownfield Quick Reference

For legacy/undocumented codebases — see [brownfield.md](brownfield.md) for full guide:

```
1. /opsx:explore              → AI investigates legacy code, maps dependencies, NO changes
   (use with Repomix MCP for token-optimized codebase packaging)

2. /opsx:continue             → Manually advance explore results through propose flow
   (one artifact at a time: proposal → specs → design → tasks)

3. Repeat for adjacent features → "greenfield zones" emerge incrementally
```

---

## CI/CD Quick Reference

For automated quality gates — see [orchestration.md](orchestration.md) for full guide:

```yaml
# GitHub Actions example
- name: Validate OpenSpec
  run: openspec validate --all --strict --no-interactive --json
```

Rejects PRs with missing, malformed, or incomplete specification artifacts.

---

## Learning Modules

| # | Module | Focus |
|---|--------|-------|
| 1 | [fundamentals.md](fundamentals.md) | Why SDD, paradigm shift, alignment levels |
| 2 | [setup.md](setup.md) | Installation, init, project introspection |
| 3 | [core_lifecycle.md](core_lifecycle.md) | Propose → Apply → Archive (core workflow) |
| 4 | [brownfield.md](brownfield.md) | Legacy integration, Explore → Continue |
| 5 | [custom_schemas.md](custom_schemas.md) | Schema engineering, minimalist & EDA |
| 6 | [orchestration.md](orchestration.md) | WorkTrees, CI/CD, MCP, contract testing |
