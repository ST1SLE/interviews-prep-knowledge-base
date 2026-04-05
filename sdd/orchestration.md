# OpenSpec — Parallel Execution, CI/CD, and Enterprise Integration

Git WorkTrees for parallel development, CI/CD quality gates, MCP-based backlog synchronization, and contract testing.

**Prerequisites:** Complete modules 01-05. This is the most advanced module — it assumes comfort with the full OpenSpec lifecycle and custom schemas.

---

## 1. The Parallelism Problem

The unified source-of-truth architecture (from [fundamentals.md](fundamentals.md)) creates a concurrency challenge: if multiple developers or AI agents propose changes simultaneously, the single specification library can suffer merge conflicts and architectural divergence.

OpenSpec solves this through **Git WorkTrees** — an advanced Git feature that checks out multiple branches simultaneously across separate physical directories.

---

## 2. Git WorkTrees for Parallel Development

### What WorkTrees Are

A WorkTree is a separate working directory linked to the same Git repository. Each WorkTree can have its own branch checked out independently. No switching branches, no stashing — true parallel work.

```bash
# Create a worktree for feature-auth on a new branch
git worktree add ../sdd-playground-auth -b feature/auth

# Create another for feature-logging
git worktree add ../sdd-playground-logging -b feature/logging
```

Now you have three directories, all sharing the same Git history:
```
sdd-playground/              ← main branch
sdd-playground-auth/         ← feature/auth branch
sdd-playground-logging/      ← feature/logging branch
```

### The OpenSpec Parallel Protocol

**Critical rule:** all initial change proposals must originate on the main branch.

Why? Because the AI must analyze the true, holistic, current state of the specifications before proposing modifications. If proposals start on stale feature branches, they'll miss recent spec changes.

**Workflow:**

```
1. On main branch:
   /opsx:propose "Add user authentication"
   /opsx:propose "Add structured logging"
   → Both proposals are reviewed against the SAME source of truth

2. Create WorkTrees for each approved proposal:
   git worktree add ../project-auth -b feature/auth
   git worktree add ../project-logging -b feature/logging

3. In each WorkTree, independently:
   /opsx:apply
   → Agent executes in isolation
   → Runs tests in isolation
   → No interference between parallel features

4. Merge back sequentially (NOT simultaneously):
   cd main-project
   git merge feature/auth
   /opsx:archive          ← updates specs/ with auth capability
   git merge feature/logging
   /opsx:archive          ← updates specs/ with logging capability

5. Clean up:
   git worktree remove ../project-auth
   git worktree remove ../project-logging
```

### Why Sequential Merge Matters

Archiving must happen one feature at a time. If you merge both branches and then archive, the specification library can't cleanly attribute which delta belongs to which feature. The sequential protocol ensures:

- Each archive operation merges one set of deltas into specs/
- The unified source of truth remains internally consistent
- No merge conflicts in specification files
- Full audit trail per feature

---

## 3. SubAgent Orchestration (OpenCode)

For teams using OpenCode (or similar agentic tooling), the parallel WorkTree pattern can be automated with **SubAgents** — asynchronous AI processes that each operate in their own WorkTree.

### How it works

```
Main Agent (on main branch):
├── Reviews and approves proposals
├── Spawns SubAgent A → WorkTree for feature/auth
├── Spawns SubAgent B → WorkTree for feature/logging
│
├── SubAgent A: /opsx:apply in isolation
│   → Writes code, runs tests, completes tasks.md
│
├── SubAgent B: /opsx:apply in isolation
│   → Writes code, runs tests, completes tasks.md
│
├── SubAgent A completes → merge feature/auth → /opsx:archive
└── SubAgent B completes → merge feature/logging → /opsx:archive
```

Each SubAgent:
- Has its own isolated filesystem (WorkTree)
- Runs its own test suite
- Cannot interfere with other SubAgents
- Must pass all validation before its branch can merge

### When to use SubAgents

- Multiple independent features can be developed simultaneously
- Features don't share modified files (reducing merge conflict risk)
- You have CI infrastructure to validate each branch independently

---

## 4. CI/CD Quality Gates

For SDD to work at scale, the specification discipline must be **enforced automatically** — not just by developer goodwill.

### The Validation Command

```bash
openspec validate --all --strict --no-interactive --json
```

| Flag | Purpose |
|------|---------|
| `--all` | Validate all specification files in the project |
| `--strict` | Fail on warnings, not just errors |
| `--no-interactive` | No prompts — suitable for CI environments |
| `--json` | Machine-readable output for pipeline parsing |

### What it checks

- Specification documents conform to required syntactic norms
- Mandatory testing scenarios are present
- Proper architectural rationales exist
- No orphaned changes (proposals that were never archived)

### GitHub Actions Integration

```yaml
# .github/workflows/spec-validation.yml
name: OpenSpec Validation

on:
  pull_request:
    branches: [main]

jobs:
  validate-specs:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install OpenSpec
        run: npm install -g @fission-ai/openspec@latest

      - name: Validate specifications
        run: openspec validate --all --strict --no-interactive --json
```

### What this enforces

- **Every PR must have corresponding specs.** Push code without specs → CI fails → PR blocked.
- **Specs must be well-formed.** Missing acceptance criteria, empty sections, broken markup → CI fails.
- **No unarchived changes.** Lingering proposals in `changes/` that weren't properly archived → CI warns.

This prevents both human developers and AI agents from bypassing the SDD workflow.

---

## 5. MCP Integration: Backlog Synchronization

The Model Context Protocol (MCP) bridges the gap between project management tools (Linear, Jira) and the Git-based OpenSpec workflow.

### The "Git Barrier" Problem

In most teams:
- **Product owners** define requirements in Linear/Jira/Azure DevOps
- **Developers** implement in Git
- **Status updates** are manual — someone has to remember to move the ticket from "In Progress" to "Done"

This manual synchronization is error-prone and creates a disconnect between business state and implementation state.

### How OpenSpec + Linear MCP Works

When connected to Linear via MCP, OpenSpec commands act as **automated state machine triggers**:

| OpenSpec Action | Linear Ticket State |
|----------------|-------------------|
| `/opsx:propose` starts | Backlog → In Progress |
| `/opsx:apply` completes | (stays In Progress) |
| `/opsx:archive` completes | In Progress → Done |

The workflow strictly separates:
- **"The What"** — business use cases defined by Product Owners in Linear
- **"The How"** — technical constraints defined by AI agents in the Git repo

### Setup

1. Configure the Linear MCP server in your AI assistant
2. When proposing a change, reference the Linear ticket: `/opsx:propose [LINEAR-123] Add user authentication`
3. The MCP server intercepts OpenSpec commands and automatically transitions ticket states

### Benefit

Product management perfectly mirrors development state — no manual updates, no stale tickets, no "is this done yet?" questions.

---

## 6. Contract Testing

For systems with APIs (REST, async), OpenSpec integrates with contract testing tools to ensure AI-generated code never breaks API contracts.

### OpenAPI Contract Locking

Specification artifacts can be compiled into formal `openapi.yaml` contracts:

```yaml
# Generated from specs.md
openapi: 3.0.0
info:
  title: SDD Playground API
  version: 1.0.0
paths:
  /health:
    get:
      summary: Health check endpoint
      responses:
        '200':
          description: Server status
          content:
            application/json:
              schema:
                type: object
                required: [status, timestamp, version]
                properties:
                  status:
                    type: string
                  timestamp:
                    type: string
                    format: date-time
                  version:
                    type: string
```

### Spectral Linting

[Spectral](https://stoplight.io/spectral) validates OpenAPI/AsyncAPI specs against configurable rules:

```bash
# Install
npm install -g @stoplight/spectral-cli

# Lint the generated contract
spectral lint openapi.yaml
```

Embed this in the AI's validation checklist within `tasks.md`:
```markdown
- [ ] Generate openapi.yaml from specs
- [ ] Run `spectral lint openapi.yaml` — must pass with 0 errors
- [ ] Only then proceed to implementation
```

### Consumer-Provider Contract Testing

For services with downstream consumers:

| Tool | Purpose |
|------|---------|
| **Swagger Mock Validator** | Validates that implementation matches the OpenAPI contract |
| **Pact** | Generates consumer-driven contract tests — ensures provider changes don't break consumers |

The spec → contract → test chain ensures: spec is written first, contract is generated from spec, implementation is validated against contract. AI cannot silently drift from the agreed interface.

---

## Exercise

### Part 1: Parallel WorkTrees

1. On your test project's main branch, propose two independent features:
   ```
   /opsx:propose Add a /metrics endpoint returning request count
   /opsx:propose Add a /ready endpoint for Kubernetes readiness probes
   ```
2. Create two WorkTrees:
   ```bash
   git worktree add ../project-metrics -b feature/metrics
   git worktree add ../project-ready -b feature/ready
   ```
3. In each WorkTree, run `/opsx:apply` independently
4. Merge back to main **sequentially**, archiving after each merge
5. Verify: `openspec/specs/` contains both capabilities, no conflicts

### Part 2: CI/CD Gate

1. Create `.github/workflows/spec-validation.yml` using the template from section 4
2. Push a PR that modifies code but has no corresponding spec changes
3. Verify: CI fails and blocks the PR
4. Add proper specs, push again — CI should pass

**Success criteria:**
- Two features developed in parallel WorkTrees without conflicts
- Sequential merge-and-archive produces a consistent specification library
- CI pipeline rejects PRs without proper specifications

---

## Common Mistakes

| Mistake | Why it's bad | Fix |
|---------|-------------|-----|
| Proposing on feature branches instead of main | AI misses recent spec changes → conflicting proposals | Always propose on main, then branch for apply |
| Merging WorkTrees simultaneously | Spec library gets conflicting deltas | Merge and archive one at a time, sequentially |
| CI validates only code, not specs | Developers bypass SDD discipline | Add `openspec validate` as a blocking CI check |
| Not cleaning up WorkTrees | Disk clutter, confusion about which is current | `git worktree remove` after merge |
| Skipping contract testing for APIs | AI silently changes API shape → downstream breaks | Lock contracts with OpenAPI + Spectral before implementing |

---

**Reference:** [cheatsheet.md](cheatsheet.md)
