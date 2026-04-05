# OpenSpec — Custom Schemas & Schema Engineering

How to tailor the SDD pipeline by defining which artifacts get generated, in what order, and with what templates.

**Prerequisites:** Complete modules 01-03. You should have run at least one full Propose→Apply→Archive cycle with the default schema.

---

## Why Custom Schemas?

The default 4-artifact workflow (proposal.md → specs.md → design.md → tasks.md) is designed for general feature development. But not every change needs the same rigor:

- A **one-line CSS fix** doesn't need a formal architectural design document
- A **Kafka microservice** needs event modeling and async API contracts before any code is written
- A **rapid prototype** needs just specs and tasks, skipping design entirely

Custom schemas let you define **exactly which artifacts the pipeline generates**, what templates they use, and what dependencies gate progression between phases.

---

## Schema Basics

### What a Schema Controls

| Aspect | What you configure |
|--------|--------------------|
| **Artifacts** | Which .md files get generated during Propose (skip design.md? add event_storming.md?) |
| **Templates** | Markdown templates that artifacts must follow (e.g., Given/When/Then format for specs) |
| **Sequence dependencies** | Which artifacts must complete before others can start (e.g., asyncapi.yaml must validate before tasks.md is generated) |
| **Validation rules** | What checks run before the pipeline advances to the next phase |

### Scaffolding a Schema

```bash
openspec schema init
```

This creates a `schema.yaml` file (in `openspec/schemas/` or project root depending on scope).

### Resolution Order

Schemas follow a cascading resolution — more specific overrides more general:

```
1. Project-level:  openspec/schemas/schema.yaml    ← highest priority
2. Global:         ~/.openspec/schemas/schema.yaml  ← default fallback
3. Built-in:       OpenSpec default (4 artifacts)   ← if no custom schema exists
```

This lets you define organization-wide defaults in `~/.openspec/` while allowing individual projects to override with project-specific workflows.

### Configuration File

The root `config.yaml` activates a schema:

```yaml
# openspec/config.yaml
schema: minimalist   # or: default, event-driven, <custom-name>
```

---

## Built-In Schema: Minimalist

Engineered for low-risk changes, minor UI updates, or rapid prototyping.

### What it generates

Only **2 artifacts** (instead of the default 4):

```
openspec/changes/<name>/
├── specs.md      ← Functional requirements
└── tasks.md      ← Execution checklist
```

No `proposal.md` (skip business justification). No `design.md` (skip architectural planning).

### Spec format

Specs are written strictly as **user stories with Given/When/Then acceptance criteria**:

```markdown
## Feature: Health Check Endpoint

### Story
As a DevOps engineer
I want a /health endpoint
So that I can monitor service availability

### Acceptance Criteria

**Given** the server is running
**When** I send a GET request to /health
**Then** I receive a 200 response with JSON body containing "status" and "timestamp"

**Given** the server just started
**When** I send a GET request to /health
**Then** the "uptime" field is less than 5 seconds
```

Implementation tasks are derived directly from these acceptance criteria — each criterion maps to a verifiable test.

### When to use it

- Bug fixes with clear reproduction steps
- Frontend tweaks (text changes, styling)
- Adding a simple endpoint with well-defined behavior
- Prototyping where you'll iterate fast and throw away early versions

### When NOT to use it

- Features involving multiple interacting components
- Changes to shared infrastructure (auth, database schema)
- Anything where "how" matters as much as "what"

---

## Built-In Schema: Event-Driven Architecture (EDA)

Purpose-built for complex, asynchronous distributed systems — message brokers (Kafka, RabbitMQ), microservices, event sourcing.

### What it generates

A specialized artifact chain with strict ordering:

```
openspec/changes/<name>/
├── event_storming.md    ← Domain actors, commands, events, bounded contexts
├── event_modeling.md    ← Mermaid.js sequence diagrams of async flows
├── asyncapi.yaml        ← Formally validated AsyncAPI contract
└── tasks.md             ← Implementation steps (only generated AFTER contract validates)
```

### Phase dependencies (critical)

```
event_storming.md  →  event_modeling.md  →  asyncapi.yaml  →  tasks.md
                                                ↑
                                          Must validate before
                                          tasks.md is generated
```

The schema **blocks code generation until the async API contract is resolved and validated.** This prevents the classic failure mode of distributed systems: building services before agreeing on the contract.

### Event Storming artifact

The AI maps:
- **Domain actors** — who triggers events
- **Commands** — what actions are taken
- **Domain events** — what state changes occur
- **Bounded contexts** — where responsibilities end

### Event Modeling artifact

Mermaid.js diagrams visualizing asynchronous interaction flows and timelines:

```markdown
```mermaid
sequenceDiagram
    participant OrderService
    participant Kafka
    participant InventoryService
    
    OrderService->>Kafka: OrderCreated event
    Kafka->>InventoryService: OrderCreated event
    InventoryService->>Kafka: InventoryReserved event
    Kafka->>OrderService: InventoryReserved event
```​
```

### AsyncAPI contract

A formally validated `asyncapi.yaml` — the async equivalent of OpenAPI/Swagger but for event-driven systems. Defines channels, message schemas, and protocol bindings.

### When to use it

- Adding features to Kafka/RabbitMQ-based architectures
- Designing new microservice interactions
- Any change where multiple services communicate asynchronously

---

## Creating Your Own Schema

Beyond the built-in schemas, you can design fully custom pipelines.

### Schema YAML structure

```yaml
# openspec/schemas/schema.yaml
name: my-custom-schema
description: Custom workflow for ML pipeline features

artifacts:
  - name: requirements
    template: templates/requirements.md
    description: Data requirements and model constraints

  - name: experiment_design
    template: templates/experiment.md
    description: Experiment design with hypothesis and metrics
    depends_on: [requirements]

  - name: tasks
    template: templates/tasks.md
    description: Implementation checklist
    depends_on: [experiment_design]

validation:
  - artifact: requirements
    checks:
      - has_section: "## Data Requirements"
      - has_section: "## Success Metrics"
```

### Key configuration options

| Field | Purpose |
|-------|---------|
| `artifacts[].name` | Artifact identifier |
| `artifacts[].template` | Markdown template file path |
| `artifacts[].depends_on` | Which artifacts must complete first |
| `validation[].checks` | Structural validation rules |

### Template files

Templates are Markdown files with placeholder sections that the AI must fill:

```markdown
# Requirements: {{change_name}}

## Problem Statement
<!-- Why this change is needed -->

## Data Requirements
<!-- What data is needed, sources, format -->

## Success Metrics
<!-- How we measure if this worked -->

## Constraints
<!-- Performance, cost, timeline bounds -->
```

---

## Schema Comparison

| Schema | Artifacts | Validation | Best for |
|--------|-----------|------------|----------|
| **Minimalist** | specs.md, tasks.md | Given/When/Then acceptance tests | Low-risk, prototyping, bug fixes |
| **Default** | proposal.md, specs.md, design.md, tasks.md | Standard checklist + unit tests | General feature development |
| **Event-Driven** | event_storming.md, event_modeling.md, asyncapi.yaml, tasks.md | Rigid AsyncAPI contract validation | Microservices, message brokers |
| **Custom** | You define | You define | Domain-specific workflows (ML, data pipelines, etc.) |

---

## Exercise

1. **Scaffold a custom schema:**
   ```bash
   openspec schema init
   ```

2. **Create a minimalist schema** that generates only `specs.md` and `tasks.md`:
   - Configure `config.yaml` to use it
   - Write a spec template that enforces Given/When/Then format

3. **Run a proposal with the minimalist schema:**
   ```
   /opsx:propose Add a /version endpoint that returns the app version
   ```
   - Verify: only `specs.md` and `tasks.md` are generated (no proposal.md, no design.md)
   - Verify: specs use Given/When/Then format

4. **Compare** with a default-schema proposal — what's different? When would you prefer each?

**Success criteria:**
- Custom schema activates via `config.yaml`
- Proposal phase produces only the configured artifacts
- You understand when to use minimalist vs default vs EDA schemas
- You could sketch a custom schema YAML for a domain you know (e.g., ML experiments)

---

## Common Mistakes

| Mistake | Why it's bad | Fix |
|---------|-------------|-----|
| Using minimalist for complex features | No design review → architectural mistakes | Match schema rigor to change risk |
| Forgetting `depends_on` in EDA schema | Tasks generated before contract validates → contract violations | Always enforce: contract validates before tasks |
| Creating schemas that are too rigid | Developers work around the schema instead of using it | Start minimal, add constraints only when they catch real problems |
| Not setting resolution order correctly | Global schema overrides project-specific one | Project-level `openspec/schemas/` takes priority over `~/.openspec/` |

---

**Next:** [orchestration.md](orchestration.md) — Parallel execution, CI/CD gates, and MCP integration
