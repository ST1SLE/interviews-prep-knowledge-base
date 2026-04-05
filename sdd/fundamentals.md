# Spec-Driven Development — Fundamentals

Why SDD exists, the paradigm shift from vibe coding, alignment levels, and architectural topologies.

---

## 1. The Problem: Vibe Coding Doesn't Scale

"Vibe coding" — iteratively prompting an LLM to generate code with minimal structured foresight — works for prototyping but breaks down in production systems.

**Why it fails at scale:**

- **Context loss.** LLMs rapidly lose context across expansive codebases. Without rigid external constraints, they forget earlier architectural decisions, misinterpret vague prompts, and produce inconsistent implementations.
- **Architectural drift.** A pivotal MSR'26 study analyzed 807 GitHub repositories using AI coding assistants (Cursor AI, etc.). Finding: while unconstrained AI generation increased initial velocity, it was invariably accompanied by persistent, compounding code complexity and architectural drift.
- **Code becomes the spec.** Without an explicit specification, the codebase itself becomes the de-facto specification — a brittle, inflexible artifact that is exceptionally difficult to decipher, debug, and safely refactor.
- **Breaking contracts.** AI-generated implementations routinely break API integration contracts or introduce security anti-patterns that only surface during production runtime.

**Bottom line:** the prompt is not a reliable constraint mechanism. You need something more deterministic.

---

## 2. The SDD Solution

Spec-Driven Development (SDD) replaces the improvisational `Prompt → Code` pattern with a structured, deterministic pipeline:

```
Specification → Architecture Plan → Task Breakdown → Agent Execution → Code
```

**Core idea:** the software specification — not the code — is the primary, authoritative artifact. The spec is an executable validation gate and a strict constitutional constraint that directly guides and bounds AI output.

| Aspect | Vibe Coding | SDD |
|--------|-------------|-----|
| Primary artifact | Code | Specification |
| AI constraint | Natural language prompt | Formal spec + task checklist |
| Documentation | Post-hoc or absent | Pre-implementation, living |
| Architectural consistency | Degrades over time | Maintained by spec |
| Reproducibility | Low (prompt-dependent) | High (spec-deterministic) |

The overarching objective: separate the stable **"what"** (business requirement, interface contract) from the flexible **"how"** (specific algorithmic implementation). This enables rapid, AI-driven iteration without incurring catastrophic technical debt.

---

## 3. Three Alignment Levels

SDD implementation exists on a continuum — three distinct levels of how seriously the team treats the specification artifact:

| Level | Name | How specs are used | Spec lifecycle |
|-------|------|--------------------|----------------|
| 1 | **Spec-First** | Drafted before code to align on requirements, then abandoned post-implementation | Disposable scaffold |
| 2 | **Spec-Anchored** | Retained post-implementation as an authoritative reference for maintenance, regression testing, and bug fixes | Living reference |
| 3 | **Spec-as-Source** | The spec IS the product. Developers edit specs, AI autonomously generates and refactors code to match | The source of truth itself |

**Spec-First** prevents hallucination during initial development but leads to inevitable spec/code divergence once the feature ships.

**Spec-Anchored** adds continuous validation — the AI agent must anchor its understanding against retained specs before proposing any changes. This catches drift.

**Spec-as-Source** is the ultimate maturity level. Human developers interact primarily with specification documents. When business requirements change, they amend the spec. AI agents then autonomously interpret these changes, refactor code, generate tests, and validate architectural compliance.

---

## 4. Unified vs Fragmented Topologies

As SDD frameworks emerged (30+ tools: Spec Kit, OpenSpec, BMAD, Intent, PromptX, Tessl, etc.), two architectural topologies diverged:

### Fragmented Topology (e.g., GitHub Spec Kit)

Every feature request, bug fix, or PR generates its own isolated set of specification and planning documents. Highly prescriptive for junior developers — forces deep analysis of each task.

**Problem:** as the project scales, specifications scatter across dozens or hundreds of disconnected files. Cross-feature interaction detection becomes mathematically prohibitive. Validating disjointed specs against the live system is nearly impossible for both humans and AI agents.

### Unified Topology (e.g., OpenSpec)

All specifications are consolidated into a singular, "living" specification library organized by core system capability. Changes are proposed as localized "deltas," reviewed, implemented, and then merged back into the master specification.

**Advantage:** deterministic feature interaction detection. AI agents can ingest a single coherent document to map the full system boundary before executing any new task.

| Trait | Fragmented (Spec Kit) | Unified (OpenSpec) |
|-------|----------------------|-------------------|
| Specification storage | Per feature/issue/PR | Consolidated by capability |
| System state representation | Isolated snapshots of intent | Live, holistic current state |
| Feature interaction detection | Complex multi-file synthesis | Native via unified capability analysis |
| Primary project fit | Greenfield, rigid compliance | Brownfield, rapid iteration, legacy modernization |
| LLM context window load | High (redundant loading) | Optimized (focused on core capabilities) |
| Installation | Python-based | TypeScript-based, lightweight npm |

---

## 5. Core Philosophy

Three principles that define the SDD mindset:

1. **"Specs are the product, code is verification."** The specification is the authoritative artifact. Code is merely the executable proof that the specification is valid. If the code deviates from the spec, the code is wrong.

2. **"Unarchived changes are technical debt."** Every proposed change that isn't formally archived into the source-of-truth specification library is a liability — it will cause context loss, conflicting assumptions, and architectural drift in future sessions.

3. **"The spec constrains the AI — not the prompt."** Natural language prompts are ambiguous and ephemeral. Formal specifications with acceptance criteria, boundary constraints, and deterministic task breakdowns are what mathematically stabilize AI outputs across sessions.

---

## Exercise

1. Browse the [Fission-AI/OpenSpec](https://github.com/Fission-AI/OpenSpec) repository on GitHub.
2. Identify the three core directories: `openspec/specs/`, `openspec/changes/`, `openspec/archive/`.
3. Read `docs/concepts.md` — compare what you learn there with this module.
4. Look at any existing spec files in `openspec/specs/` — how are they structured?

**Success criteria:** You can articulate (a) why vibe coding fails at scale, (b) the difference between the three alignment levels, and (c) why unified topology scales better than fragmented.
