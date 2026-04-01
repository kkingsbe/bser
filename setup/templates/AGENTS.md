# AGENTS.md
<!-- BSER: See .bser-version for framework version -->

## Project Overview

<1-2 sentence description of the project, its tech stack, and primary purpose>

## Commands

These are the exact commands for building, testing, and running this project. Use these — do not guess or infer alternatives.

```bash
# Install dependencies
<e.g., npm install, pnpm install, cargo build, pip install -e .>

# Run full test suite
<e.g., npm test, cargo test, pytest>

# Run a single test file
<e.g., npx jest path/to/file.spec.ts, cargo test --test test_name, pytest path/to/test.py>

# Lint / type check
<e.g., npm run lint, npm run typecheck, cargo clippy>

# Start dev server
<e.g., npm run start, ng serve, cargo run>

# Build for production
<e.g., npm run build, cargo build --release>
```

**Dev server URL:** `<e.g., http://localhost:4200, http://localhost:3000>`
**Primary branch:** `<e.g., main, develop>`

## BSER Workflow

This project uses the BSER (Brief → Scope → Execute → Review+Sync) framework for human-in-the-loop agentic development.

### Key Files

- `ARCHITECTURE.md` — Living architecture doc, reconciled to the codebase by the `@syncer` agent. Read this before making structural decisions.
- `CONVENTIONS.md` — Coding patterns and style decisions. Follow these when writing code.
- `.plans/` — Implementation plan documents. Every feature/fix gets a plan doc before execution.
- `.plans/backlog.md` — Captured future work items. Append-only during work sessions.

### Plan Categories

Every plan document should be tagged with one of these categories in its frontmatter. This enables `/estimate` to analyze patterns by category.

- **feature** — New functionality or capability
- **bugfix** — Fixing broken behavior
- **refactor** — Restructuring without changing behavior
- **chore** — Dependencies, config, CI, tooling, docs-only changes

### Workflow Rules

- Always read the relevant plan doc in `.plans/` before implementing. The plan is the contract.
- Follow TDD: write tests first, then implement to make them pass.
- Commit after each logical unit of progress with descriptive commit messages.
- Never expand scope beyond the current plan. If you discover something new, add it to the "Future (Out of Scope)" section of the plan doc.
- Never refactor unrelated code during a feature implementation.
- If the plan is wrong or insufficient, stop and report the issue rather than improvising.

## Code Style

<Language-specific style rules, e.g.:>
- Use TypeScript strict mode
- Prefer named exports over default exports
- Use descriptive variable names — no single-letter variables outside loop indices

## Architecture Rules

<Guard rails for structural decisions, e.g.:>
- All new modules must be registered in ARCHITECTURE.md before implementation
- Shared code goes in the designated shared module — do not duplicate across services
- Database schema changes require a migration file

## Testing

<Testing strategy, e.g.:>
- Unit tests for all business logic — colocate test files with source (`*.spec.ts`)
- Integration tests for API endpoints and cross-module interactions

### Test Depth Requirements

For test case category definitions, see `{{partials:test-case-categories}}`

A test that only calls a function with perfect input and asserts `!= null` is not a useful test. Test *behavior*, not just execution.

## Git Conventions

- Primary branch: `<main or develop>`
- Branch naming: `feat/<name>`, `fix/<name>`, `spike/<name>`
- Commit messages: imperative mood, lowercase, no period (`add user auth endpoint`)
- One logical change per commit — do not bundle unrelated changes
- Never commit directly to the primary branch during feature work

## Dependencies

- Check existing dependencies before adding new ones
- Prefer standard library solutions over third-party packages when reasonable
- Document why a new dependency was added in the plan doc's Implementation Notes
