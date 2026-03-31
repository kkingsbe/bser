# BSER Examples

Real-world examples demonstrating the BSER framework in action. Use these as reference when learning BSER or when starting a new type of task you haven't encountered before.

---

## Directory Structure

```
setup/examples/
├── workflows/          # End-to-end BSER loop walkthroughs
├── plans/              # Example plan documents
├── epics/              # Multi-phase epic examples
└── scenarios/          # Complex real-world situations
```

---

## Workflows

Complete walkthroughs of the BSER loop for different task types.

| Example | What It Demonstrates |
|---------|---------------------|
| [Feature Workflow](./workflows/feature-workflow.md) | Standard feature from `/brief` through `/sync` with all phases |
| [Bugfix Workflow](./workflows/bugfix-workflow.md) | Quick bugfix using `/hotfix` and the fast-path review |
| [Epic Workflow](./workflows/epic-workflow.md) | Multi-phase epic execution with phase-by-phase breakdown |

---

## Plans

Annotated example plan documents showing structure and content.

| Example | What It Demonstrates |
|---------|---------------------|
| [Feature Plan](./plans/feature-plan-example.md) | Standard feature plan with scope, test cases, implementation notes |
| [Bugfix Plan](./plans/bugfix-plan-example.md) | Bugfix plan with root cause analysis, reproduction steps, verification |

---

## Epics

Multi-phase epic examples with decomposition and phase management.

| Example | What It Demonstrates |
|---------|---------------------|
| [Multi-Phase Epic](./epics/multi-phase-epic-example.md) | 4-phase GraphQL migration epic with architecture impact, risks, exit criteria |

---

## Scenarios

Complex real-world situations demonstrating BSER in context.

| Example | What It Demonstrates |
|---------|---------------------|
| [Payment Refactor](./scenarios/payment-refactor.md) | High-risk refactor with 6 phases, migration pattern, rollback strategy |
| [User Authentication](./scenarios/user-authentication.md) | New feature epic with provider abstraction, security considerations |

---

## How to Use These Examples

### Learning BSER

Start with [Feature Workflow](./workflows/feature-workflow.md) — it walks through every phase of the BSER loop with actual commands and expected outputs. This gives you the full picture before diving into specifics.

### Starting a New Task Type

- **New feature** → Reference [Feature Plan](./plans/feature-plan-example.md) for plan structure
- **Bugfix** → Reference [Bugfix Workflow](./workflows/bugfix-workflow.md) for the fast path
- **Multi-phase effort** → Reference [Multi-Phase Epic](./epics/multi-phase-epic-example.md) for epic structure
- **Complex refactor** → Reference [Payment Refactor](./scenarios/payment-refactor.md) for migration patterns

### Templates

These examples complement the templates in `setup/templates/`:

| Template | Purpose |
|----------|---------|
| `setup/templates/plan-template.md` | Blank plan document structure |
| `setup/templates/epic-template.md` | Blank epic document structure |
| `setup/templates/backlog-template.md` | Backlog file structure |
| `setup/templates/commands-registry-template.md` | Commands registry |

The examples show these templates in context — what填入 (what goes in each field), not just the structure.

---

## Related Documentation

- [Workflow Reference](../workflows.md) — Daily BSER loop reference
- [Setup Guide](../setup.md) — Framework setup and configuration
- [Commands](../setup/commands/) — Individual command documentation
