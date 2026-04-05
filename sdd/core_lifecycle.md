# OpenSpec — The Core Lifecycle: Propose → Apply → Archive

The operational heart of Spec-Driven Development. This module walks through the three-phase delta workflow with hands-on exercises.

**Prerequisites:** Complete [fundamentals.md](fundamentals.md) and [setup.md](setup.md) first. You need a project with OpenSpec initialized.

---

## Overview

OpenSpec revolves around a continuous three-phase cycle:

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│   PROPOSE          APPLY           ARCHIVE          │
│   (draft intent) → (execute) →     (consolidate)    │
│                                                     │
│   "What & why"     "Build it"      "Lock it in"     │
│                                                     │
│   ↑                                          │      │
│   └──────── loop back if needed ─────────────┘      │
│                                                     │
└─────────────────────────────────────────────────────┘
```

Each phase has a distinct purpose, a specific command, and clear outputs. The phases are not rigidly linear — if the Apply phase reveals that the architectural plan is flawed, you loop back to Propose and update the specs.

**Key insight:** the Propose phase decouples architectural negotiation from code implementation. You finalize the "what" before touching the "how."

---

## Phase 1: Propose (Drafting Intent)

### Command

```
/openspec:proposal    (full form)
/opsx:propose         (shorthand)
```

### What You Do

Provide a natural language description of the change:

```
/opsx:propose Add a health check endpoint at /health that returns server status and timestamp
```

### What the AI Does

Instead of immediately writing code, the AI enters **architectural planning mode**:

1. Reads `AGENTS.md` → reads `project.md` → reads existing `specs/`
2. Creates an isolated directory: `openspec/changes/<change-name>/`
3. Generates **4 delta artifacts** inside that directory

### The 4 Delta Artifacts

| Artifact | Question it answers | Content |
|----------|-------------------|---------|
| `proposal.md` | **Why** is this change needed? | Business motivation, context, rationale |
| `specs.md` | **What** exactly should change? | Functional parameters, boundary constraints, testable acceptance scenarios |
| `design.md` | **How** will it be built? | Technical approach, architecture, algorithm choices |
| `tasks.md` | **What are the steps?** | Granular, deterministic, checkboxed execution plan |

### Delta Markup

Within `specs.md`, changes to existing capabilities are marked with explicit sections:

- **`ADDED`** — entirely new capability
- **`MODIFIED`** — change to an existing capability
- **`REMOVED`** — capability being removed

This markup lets both human reviewers and AI agents understand exactly what's changing without diffing entire documents.

### Token Efficiency

Where competing frameworks might generate 800+ lines of verbose planning documents, OpenSpec typically restricts to ~250 highly focused lines. This reduces review overhead and minimizes LLM context window consumption.

### Human Checkpoint

**This is your control point.** After the AI generates the 4 artifacts:

1. Read `proposal.md` — does the motivation make sense?
2. Read `specs.md` — are the requirements complete and correct?
3. Read `design.md` — is the technical approach sound?
4. Read `tasks.md` — are the steps logical and complete?

Do NOT proceed to Apply until you're satisfied. Edit the artifacts directly if something is wrong. This is where you catch hallucinations, missing edge cases, and flawed architectural assumptions — **before** any code exists.

---

## Phase 2: Apply (Rigorous Execution)

### Command

```
/openspec:apply    (full form)
/opsx:apply        (shorthand)
```

### What Happens

The AI agent transitions from planning to implementation. It is **strictly bound to `tasks.md`** — no improvisation, no "free rein," no scope creep.

### Agent Behavior During Apply

The agent:

1. **Follows tasks sequentially** from `tasks.md`
2. **Runs verification checks** continuously (tests, linting, compilation)
3. **Updates checkbox status** in `tasks.md` as each step completes
4. **Monitors system health** — ensures the app still loads, existing tests pass

### The Deviation Protocol

If implementation reality diverges from the plan (e.g., a dependency doesn't support the planned approach, or a test reveals an edge case not covered in specs):

1. **Halt** — stop implementing
2. **Document** — record the deviation and why the plan doesn't work
3. **Suggest spec update** — propose amendments to the specification
4. **Resume only after approval** — the human confirms the spec change

This is the critical difference from vibe coding: the spec constrains the code, not the other way around. If reality doesn't match the spec, you update the spec first — you don't silently work around it.

### What NOT to Do During Apply

- Don't let the AI add features not in the spec
- Don't let it refactor code outside the scope of the current change
- Don't skip the testing steps in tasks.md
- Don't approve deviations without updating the spec

---

## Phase 3: Archive (Knowledge Consolidation)

### Command

```
/openspec:archive    (full form)
/opsx:archive        (shorthand)
```

### What Happens

Two critical actions:

1. **Move to history:** `openspec/changes/<name>/` → `openspec/archive/<name>/`
   - Creates an immutable audit trail
   - Preserves the exact context of how and why decisions were made
   - Never modified after archiving

2. **Merge into source of truth:** delta specs merge into `openspec/specs/`
   - The unified specification library now reflects the new/changed capability
   - Any future AI agent reading the project sees the full, updated system state

### Why Archiving is Non-Negotiable

**"Unarchived changes are technical debt."**

If you skip archiving:
- The next AI session won't know about the change (specs/ is stale)
- Future proposals might contradict or duplicate the unarchived work
- Context continuity breaks — the closed-loop knowledge retention system has a gap
- You're back to vibe coding territory

Always archive immediately after a successful Apply phase.

### After Archiving

Verify:
- `openspec/changes/` should be empty (or have only other active proposals)
- `openspec/specs/` should include the new capability
- `openspec/archive/` should contain the completed proposal with all 4 artifacts

---

## Full Walkthrough: Exercise 1 — New Feature

Using the project you set up in [setup.md](setup.md):

### Step 1: Propose

```
/opsx:propose Add a health check endpoint at /health that returns JSON with status, timestamp, and uptime
```

Wait for the AI to generate the 4 artifacts in `openspec/changes/`.

### Step 2: Review

Read each artifact:

```bash
ls openspec/changes/         # see the change directory
cat openspec/changes/*/proposal.md
cat openspec/changes/*/specs.md
cat openspec/changes/*/design.md
cat openspec/changes/*/tasks.md
```

Ask yourself:
- Does `specs.md` cover edge cases? (What if the server just started? What format is the timestamp?)
- Does `tasks.md` include a testing step?
- Does `design.md` make reasonable technical choices for your stack?

### Step 3: Apply

```
/opsx:apply
```

Watch the agent:
- Follow tasks from `tasks.md`
- Write the actual code
- Update checkboxes as it goes
- Run any specified tests

### Step 4: Verify

- Hit the `/health` endpoint — does it return the expected JSON?
- Do existing endpoints still work?
- Are all checkboxes in `tasks.md` marked complete?

### Step 5: Archive

```
/opsx:archive
```

### Step 6: Confirm

```bash
ls openspec/changes/     # should be empty
ls openspec/specs/       # should contain health check capability
ls openspec/archive/     # should contain the completed proposal
```

---

## Full Walkthrough: Exercise 2 — Modifying an Existing Feature

Now propose a **modification** to the feature you just created:

### Step 1: Propose

```
/opsx:propose Add a "version" field to the health check response, reading from a VERSION file or defaulting to "dev"
```

### Step 2: Review

This time, pay special attention to `specs.md`:
- Look for the **`MODIFIED`** markup — it should indicate changes to the existing health check capability
- The spec should reference the existing capability (from `openspec/specs/`) and show what's being added

### Steps 3-6: Apply, Verify, Archive, Confirm

Same flow as Exercise 1.

**Key observation:** after archiving, `openspec/specs/` should now reflect the health check capability **with** the version field — a clean evolution of the specification, not a separate document.

---

## Common Mistakes

| Mistake | Why it's bad | Fix |
|---------|-------------|-----|
| Skipping the review step | Hallucinated specs lead to wrong code | Always read all 4 artifacts before Apply |
| Not archiving after Apply | Future sessions lose context | Archive immediately after successful Apply |
| Letting the AI deviate without updating spec | Spec/code divergence — you're back to vibe coding | Follow the deviation protocol |
| Editing code directly without a proposal | Bypasses the spec constraint entirely | Always go through Propose → Apply → Archive |
| Starting Apply before the spec is right | You'll build the wrong thing faster | Invest time in the Propose review — it pays off |

---

## Summary

| Phase | Command | Input | Output | Human action |
|-------|---------|-------|--------|-------------|
| **Propose** | `/opsx:propose` | Natural language intent | 4 delta artifacts in `changes/` | Review & approve |
| **Apply** | `/opsx:apply` | Approved proposal | Working code + completed tasks.md | Monitor & verify |
| **Archive** | `/opsx:archive` | Completed implementation | Updated `specs/` + history in `archive/` | Confirm state |

**Success criteria for this module:**
- Completed two full Propose→Apply→Archive cycles (new feature + modification)
- `openspec/specs/` reflects both the health check capability and the version field
- `openspec/archive/` contains both historical proposals
- Can explain what each phase does, why the order matters, and what happens if you skip a phase

---

**Quick reference:** [cheatsheet.md](cheatsheet.md)
