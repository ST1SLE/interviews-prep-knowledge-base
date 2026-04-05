# OpenSpec — Brownfield Integration & Legacy Modernization

How to apply SDD to existing, undocumented codebases without triggering context overload or hallucination.

**Prerequisites:** Complete [01_fundamentals.md](01_fundamentals.md), [02_setup.md](02_setup.md), and [03_core_lifecycle.md](03_core_lifecycle.md). You should be comfortable with the Propose→Apply→Archive cycle.

---

## The Problem

The core Propose→Apply→Archive cycle from [03_core_lifecycle.md](03_core_lifecycle.md) assumes you're working on a greenfield project or one that already has specs. But most real-world codebases are **brownfield** — legacy code with no documentation, no specs, and implicit architectural decisions buried across thousands of files.

Naively asking an LLM to "reverse-engineer this entire codebase into specifications" will:
- **Blow the context window** — large codebases exceed token limits
- **Trigger hallucination** — the AI will fabricate structure it can't actually see
- **Produce unusable output** — a 5000-line spec dump nobody will read or maintain

OpenSpec solves this with an **incremental, bounded approach**: specify only what you're about to touch, not the entire system.

---

## The Explore → Continue Workflow

Two specialized commands for brownfield scenarios:

```
/opsx:explore     →  Investigate (read-only, no code changes)
/opsx:continue    →  Advance explore results into the propose flow
```

### Phase 0: Explore

```
/opsx:explore
```

The AI acts **solely as a thinking partner** during this phase:

- Investigates the legacy code architecture
- Clarifies undocumented requirements
- Maps dependencies between components
- Identifies patterns, conventions, and anti-patterns
- **Writes zero code** — this is analysis only

The output is a contextual understanding that grounds subsequent specifications in reality, preventing hallucinated specs.

**Key rule:** never skip Explore and jump straight to Propose on unfamiliar legacy code. The AI needs to build an accurate mental model first.

### Phase 1: Continue

```
/opsx:continue
```

After Explore builds understanding, Continue manually advances through the standard flow **one artifact at a time**:

```
Explore results → proposal.md → specs.md → design.md → tasks.md
```

Each artifact gets generated and reviewed individually, giving you granular control over exactly what's being specified about the legacy system. This prevents the AI from running ahead and producing specs based on assumptions you haven't validated.

After all 4 artifacts are approved, you proceed with the normal Apply → Archive cycle.

---

## Repomix: Token-Optimized Codebase Packaging (Complementary Tool)

Large legacy codebases won't fit in an LLM's context window raw. **[Repomix](https://github.com/yamadashy/repomix)** is a separate, standalone tool that packages codebases into a token-optimized format suitable for AI ingestion. It is **not part of OpenSpec** but pairs well with the brownfield workflow.

### What it does

- Compresses the codebase representation without losing structural information
- Strips irrelevant content (build artifacts, node_modules, etc.)
- Organizes code into a format the AI can efficiently parse
- Available as a CLI tool and as an MCP server for AI assistants

### How to use it with OpenSpec

1. Install Repomix: `npm install -g repomix` (or set up its MCP server in your AI assistant)
2. Before `/opsx:explore`, use Repomix to package the legacy codebase into a compressed snapshot
3. Feed the compressed representation to the AI alongside the explore command

### When you need it

- Codebases with 50+ files or 10k+ lines
- Projects with deep directory nesting
- Monorepos where you only need to understand one service

---

## The "Greenfield Zone" Strategy

You don't modernize an entire legacy codebase at once. Instead, you create **greenfield zones** — bounded areas where specs exist and SDD discipline applies — surrounded by legacy code that remains unspecified.

### How it works

1. **Pick the hotspot** — the component you're actually about to modify
2. **Explore** that specific component and its immediate dependencies
3. **Continue** to create specs for that bounded context only
4. **Apply** the change, **Archive** the result
5. **Repeat** for adjacent, highly-coupled features

### What happens over time

```
Sprint 1:  [legacy] [legacy] [SPECIFIED] [legacy] [legacy]
Sprint 3:  [legacy] [SPECIFIED] [SPECIFIED] [SPECIFIED] [legacy]
Sprint 6:  [SPECIFIED] [SPECIFIED] [SPECIFIED] [SPECIFIED] [legacy]
```

Specification density naturally accretes around the most frequently modified, high-risk areas. The most active code paths transform into fully specified, validated greenfield zones — exactly where coverage matters most.

### Key benefit

You don't waste effort specifying code that nobody touches. The effort is concentrated where the risk of regression and unintended side effects is highest.

---

## Brownfield Workflow Summary

```
1. Identify the component to modify

2. /opsx:explore
   → AI investigates legacy code (read-only)
   → Uses Repomix MCP if codebase is large
   → Maps dependencies, conventions, implicit contracts

3. /opsx:continue
   → Generate proposal.md → review
   → Generate specs.md → review (specs reflect ACTUAL legacy behavior)
   → Generate design.md → review
   → Generate tasks.md → review

4. /opsx:apply
   → Normal execution bound to tasks.md

5. /opsx:archive
   → Specs merge into openspec/specs/
   → A "greenfield zone" is born

6. Repeat for adjacent features
```

---

## Exercise

Use one of your existing projects (e.g., random-coffee-bot or text2brainrot):

1. **Initialize OpenSpec** in the project: `openspec init`
2. **Run `/opsx:explore`** targeting a specific component (e.g., "Explore the matching algorithm in random-coffee-bot" or "Explore the TTS pipeline in text2brainrot")
3. **Evaluate the AI's analysis** — did it accurately understand the code? Did it hallucinate any dependencies?
4. **Run `/opsx:continue`** to create specs for that component, reviewing each artifact individually
5. **Archive** the result

**Success criteria:**
- The AI explored a bounded legacy component without attempting to reverse-engineer the entire system
- Generated specs accurately reflect the actual behavior (not hallucinated)
- `openspec/specs/` now contains a localized specification for that component
- You understand why bounded exploration beats full-system specification

---

## Common Mistakes

| Mistake | Why it's bad | Fix |
|---------|-------------|-----|
| Trying to spec the entire legacy system at once | Context overflow, hallucination, useless output | Bound your scope to the component you're modifying |
| Skipping Explore and going straight to Propose | Specs will be based on assumptions, not reality | Always Explore unfamiliar code first |
| Not using Repomix for large codebases | Context window limits cause truncation and hallucination | Use Repomix to package large codebases before exploring |
| Specifying code nobody touches | Wasted effort, stale specs | Focus on hotspots — most modified, highest risk |

---

**Next:** [05_custom_schemas.md](05_custom_schemas.md) — Tailoring the SDD pipeline with custom schemas
