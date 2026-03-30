# BSER Framework — Setup Guide

> One-time setup for the Brief → Scope → Execute → Review+Sync workflow.
> After completing this guide, see **BSER-workflow.md** for daily usage.

---

## Prerequisites

- Kilo Code CLI installed and configured
- Git initialized in your project
- A working Kilo config at `~/.config/kilo/` or `~/.kilocode/`
- agent-browser installed: `npx skills add https://github.com/vercel-labs/agent-browser --skill agent-browser`

---

## 1. Project File Structure

Create these directories and files in your project root:

```bash
mkdir -p .plans/epics
mkdir -p .kilocode/commands
mkdir -p .kilo/agents
mkdir -p .reports/screenshots

touch AGENTS.md
touch ARCHITECTURE.md
touch CONVENTIONS.md
touch .bser-version
touch .plans/backlog.md
touch .kilocode/commands/commands.md
touch BSER-QUICKREF.md
```

### .gitignore Additions

Add these to your project's `.gitignore`. Reports and screenshots are ephemeral outputs — they can always be regenerated. Fixlists are transient review artifacts. Test baselines *are* committed (they're part of the plan's branch).

```gitignore
# BSER ephemeral outputs
.reports/
.plans/*.fixlist.md
```

The following **should** be committed:
- `.bser-version` (framework version — see Versioning section)
- `.plans/*.md` (plan docs, backlog, templates)
- `.plans/epics/` (epic plan directories)
- `.plans/*.baseline-tests.txt` (test baselines — needed by `/review`)
- `AGENTS.md`, `ARCHITECTURE.md`, `CONVENTIONS.md`
- `BSER-QUICKREF.md` (quick reference for daily usage — keep open while working)
- `.kilocode/commands/commands.md` (commands registry)
- `.kilocode/commands/*.md` (slash commands — except commands.md which is the registry)
- `.kilo/agents/*.md` (subagent definitions)

### Versioning

The BSER framework uses a single `.bser-version` file at the project root to track which version of the framework is in use.

**Where the version is defined:** `.bser-version`

**Format:**
```text
version: 1.0
date-adopted: 2024-01-15
notes: Initial setup
```

**How to check the current version:**
```bash
cat .bser-version
```

**How to upgrade:**
1. Update the `version` field in `.bser-version`
2. Run through the [Migration Guide](#7-migrating-from-a-previous-bser-version) if there are breaking changes
3. Update `notes` to record when and why the upgrade happened

The version follows [Semantic Versioning](https://semver.org/): `major.minor.patch`
- `major`: Breaking changes that require migration
- `minor`: New commands, agents, or features (backward compatible)
- `patch`: Bug fixes, documentation updates

### AGENTS.md (Starter Template)

This is the project-wide agent configuration file. Every Kilo Code session — commands, subagents, and interactive — loads this automatically. It's the single place to define how agents should behave in this project.

```markdown
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

- `ARCHITECTURE.md` — Living architecture doc, updated by the `@syncer` agent after each merge. Read this before making structural decisions.
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
- Integration tests for API endpoints
- Minimum: every function in the plan's "Test Cases" section must have a corresponding test

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
```

> **Adapt this template.** The sections above are starting points. Strip out what doesn't apply, add what does. The `## Commands` section is the most critical to fill in accurately per-project — it's what every command reads to know how to build, test, and run your project. The `## BSER Workflow` section tells every agent session how to behave within the framework.

### ARCHITECTURE.md (Starter Template)

```markdown
# Architecture

> This document is maintained by the syncer agent after each feature merge.
> Last updated: <date>

## Overview

<1-2 sentence description of what this project is>

## Module Boundaries

| Module | Responsibility | Key Files |
|--------|---------------|-----------|
| | | |

## Data Flow

<How data moves through the system — even a sentence or two is fine to start>

## Key Decisions

<Record non-obvious architectural decisions here. Why MongoDB over Postgres? Why NestJS? etc.>
```

### CONVENTIONS.md (Starter Template)

```markdown
# Conventions

> This document is maintained by the syncer agent after each feature merge.
> Last updated: <date>

## Code Style

<Naming conventions, file organization patterns, import ordering, etc.>

## Patterns

<Recurring patterns in this codebase — how services are structured, how errors are handled, etc.>

## Testing

<Testing strategy — what gets unit tested, what gets integration tested, naming conventions for test files>
```

### .plans/backlog.md (Starter Template)

```markdown
# Backlog

> Append-only during work sessions. Groom between sessions if needed.

## Ideas / Future Work

-
```

### .kilocode/commands/commands.md (Commands Registry)

This is the master registry of all available slash commands. Maintain this manually — update it when you add, modify, or remove commands. The syncer agent updates this during `/sync`.

```markdown
# Commands Registry

## Available Commands

| Command | Description |
|---------|-------------|
| `/brief` | Generate a project briefing |
| `/scope` | Create a scoped implementation plan |
| `/epic` | Decompose a large task into phases |
| `/implement` | Continue implementing a plan |
| `/review` | Review the current branch diff |
| `/sync` | Update docs after merge |
| `/hotfix` | Quick fix escape hatch |
| `/recap` | End-of-session summary |
| `/impact` | Dependency impact analysis |
| `/estimate` | Planned vs actual calibration |

## Adding Commands

When you add a new command to `.kilocode/commands/<name>.md`:
1. Add an entry to the table above with the command name and description
2. Run `/sync` on any merged plan to ensure this file is updated
```

---

## 2. Plan Document Template

Save this as `.plans/_TEMPLATE.md` for reference (or let the `/scope` command generate plans in this format automatically):

```markdown
# Plan: <feature-name>

**Status:** PLANNING | IN_PROGRESS | IN_REVIEW | COMPLETE
**Category:** feature | bugfix | refactor | chore
**Branch:** feat/<short-name>
**Parent:** main | epic/<epic-name>
**Created:** <date>
**One-liner:** <single sentence description>

## Scope

<What this feature does, in 2-3 sentences max>

## Files to Change

- `src/module/file.ts` — <what changes>
- `src/module/file.spec.ts` — <test cases>

## Test Cases

- [ ] <test case 1>
- [ ] <test case 2>
- [ ] <test case 3>

## Implementation Notes

<Key decisions, gotchas, dependencies>

## Future (Out of Scope)

<Anything that came up during planning but is NOT part of this task.
Move these to .plans/backlog.md during sync.>

---

## Completion Log

**Completed:** <date>
**Actual changes:** <summary of what was built>
**Deviated from plan:** <yes/no, and how>
**Impact:** <what modules now depend on this>
```

### Epic Template

For large tasks that span multiple phases — refactors, multi-module features, migrations — use an epic. An epic is a parent plan that decomposes into ordered child plans, each of which follows the normal BSER loop independently.

Save this as `.plans/_EPIC_TEMPLATE.md`:

```markdown
# Epic: <epic-name>

**Status:** PLANNING | IN_PROGRESS | COMPLETE
**Branch:** epic/<epic-name>
**Created:** <date>
**One-liner:** <single sentence describing the full outcome>

## Objective

<What this epic achieves when fully complete, in 3-5 sentences. What does the codebase look like after all phases are done?>

## Phases

Each phase is a self-contained plan that branches from and merges back into `epic/<epic-name>`. Phases are ordered — later phases may depend on earlier ones being merged.

| # | Plan | Description | Dependencies | Status |
|---|------|-------------|-------------|--------|
| 1 | `<epic>-phase-1-<name>` | <what this phase does> | none | PLANNING |
| 2 | `<epic>-phase-2-<name>` | <what this phase does> | phase 1 | PLANNING |
| 3 | `<epic>-phase-3-<name>` | <what this phase does> | phase 2 | PLANNING |

## Architecture Impact

<How does this epic change the system architecture? What modules are affected?>

## Risks & Open Questions

- <risk or question 1>
- <risk or question 2>

## Exit Criteria

<How do you know the epic is done? What should be true when all phases are complete?>

---

## Progress Log

| Phase | Started | Completed | Notes |
|-------|---------|-----------|-------|
| 1 | | | |
| 2 | | | |
| 3 | | | |
```

---

## 3. Custom Slash Commands

Create each of these markdown files in `.kilocode/commands/` (project-level) or `~/.kilocode/commands/` (global).

**Design principle:** Every command that produces human-facing context (briefs, recaps, reviews, impact analyses, estimates) gathers its data and then invokes `@reporter` to render a visual HTML report. Action commands (scope, implement, sync, hotfix) that produce code or docs stay as terminal output. This means your primary interface for understanding the project state is always a polished, scannable report — not a wall of terminal text.

### `/brief` — Phase 1: Briefing

```markdown
# .kilocode/commands/brief.md
---
description: Generate a project briefing to rebuild mental model before starting work
mode: architect
---
Generate a development briefing for this project by gathering the following data:

1. RECENT CHANGES
   - Run `git log --oneline --since="48 hours ago"` (or since last merge to main if more recent).
   - For each commit, note: what changed, which modules/services were touched.
   - Check for any new dependencies or patterns introduced.

2. CURRENT STATE
   - What branch am I on? What is its status vs main?
   - Are there uncommitted changes?
   - Scan .plans/ for any documents with status IN_PROGRESS or IN_REVIEW.

3. EPIC STATUS
   - Check for any `epic/*` branches: `git branch --list 'epic/*'`
   - For each active epic, read its plan doc and summarize: how many phases total, how many complete, what's the current/next phase.
   - Flag if any epic branch has diverged significantly from main (more than 20 commits behind).

4. ARCHITECTURE SNAPSHOT
   - Read ARCHITECTURE.md. Flag anything in recent commits that contradicts, extends, or is missing from it.
   - List the top-level module boundaries and their responsibilities.

5. SUGGESTED NEXT ACTIONS
   - Based on open plans, epic progress, and recent momentum, identify 1-3 logical next tasks.
   - Flag anything that looks broken, half-finished, or blocking.

Once you have all the data, invoke @reporter to render it as a briefing report.

The report should include:
- A stat card row: commits in last 48h, open plans count, active epics count, current branch, uncommitted file count
- A "Recent Changes" section with a concise table of commits and affected modules
- A "Current State" section showing branch status and open plans with their progress
- An "Epic Progress" section (only if active epics exist) — for each epic show: phase completion as a progress bar or fraction, current phase name, and whether the epic branch needs rebasing against main
- An architecture snapshot section — only if there are flags to raise
- A "Suggested Next Actions" section with 1-3 prioritized tasks
- If there are 3+ recent commits touching different modules, include a mermaid graph showing which modules were affected

Save to: `.reports/brief-<YYYY-MM-DD>.html`

## Examples

**Invocation:**
```
/brief
```

**Expected Output:**
HTML report at `.reports/brief-<YYYY-MM-DD>.html` with:
- Stat card row: commits in last 48h, open plans count, active epics count, current branch, uncommitted file count
- "Recent Changes" section: table of commits and affected modules
- "Current State" section: branch status, open plans with progress
- "Epic Progress" section (if epics exist): phase completion fraction, current phase, rebase flag
- "Suggested Next Actions" section: 1-3 prioritized tasks

**Common Variations:**
- `/brief` — Full project briefing after returning from break
- Run on any branch to rebuild context before starting work
```

## Structured Output

When invoking @reporter, pass this JSON schema:

```json
{
  "report_type": "brief",
  "generated_at": "ISO8601 timestamp",
  "current_branch": "string",
  "stats": {
    "commits_48h": "number",
    "open_plans": "number",
    "active_epics": "number",
    "uncommitted_files": "number"
  },
  "recent_changes": [
    {
      "commit_hash": "string",
      "message": "string",
      "affected_modules": ["string"]
    }
  ],
  "current_state": {
    "branch_status": "string",
    "open_plans": [
      {
        "name": "string",
        "status": "IN_PROGRESS | IN_REVIEW",
        "progress": "string"
      }
    ]
  },
  "epic_progress": [
    {
      "name": "string",
      "total_phases": "number",
      "completed_phases": "number",
      "current_phase": "string",
      "needs_rebase": "boolean"
    }
  ],
  "architecture_flags": ["string"],
  "suggested_next_actions": ["string"]
}
```

### `/scope` — Phase 2: Scoping

```markdown
# .kilocode/commands/scope.md
---
description: Create a scoped implementation plan and branch for a task
mode: architect
---
I'm scoping a new task: "$ARGUMENTS"

Follow this process:

0. NAMING: From the task description above, derive:
   - A **slug** — a short kebab-case identifier (2-4 words max) suitable for branch names and filenames. Examples: `add-xer-parser`, `fix-mongo-timeout`, `refactor-auth-service`.
   - A **one-liner** — a single sentence description of the task.
   Use the slug for all branch names and file names below.

1. BRANCH: Determine the correct base branch:
   - Check if the task description references an existing epic. If so:
     - Check if an epic branch `epic/<epic-slug>` exists.
     - If the epic branch exists, determine the next phase number by reading the epic's plan doc and checking its phases table.
     - Branch from the epic branch: `git checkout epic/<epic-slug> && git pull && git checkout -b feat/<slug>`
   - If no epic reference, check if we're already on an epic branch. If so, create the plan as a phase of that epic.
   - Otherwise, branch from main: `git checkout main && git pull && git checkout -b feat/<slug>`
   - For bugfixes (description mentions fixing/resolving a bug), use `fix/<slug>` instead of `feat/<slug>`.

2. TEST BASELINE: Run the test suite on the fresh branch **before any changes** and save the results:
   - Read the test command from the `## Commands` section of `AGENTS.md`. Do not assume `npm test`.
   - Run tests and capture output: `<test command from AGENTS.md> 2>&1 | tee .plans/<slug>.baseline-tests.txt`
   - Record the pass/fail counts at the top of the file.
   - This baseline will be used by `/review` to distinguish pre-existing failures from regressions introduced by this task.
   - Commit the baseline: `git add .plans/<slug>.baseline-tests.txt && git commit -m "test baseline: <slug>"`

3. PLAN LOCATION: Determine where to save the plan:
   - If this is a phase of an existing epic:
     - The epic's plan doc is at `.plans/epics/<epic-slug>/README.md`
     - Save this phase's plan at `.plans/epics/<epic-slug>/<slug>.md`
   - Otherwise:
     - Save the plan at `.plans/<slug>.md`

4. PLAN: Create an implementation plan:
   - List the specific functions, methods, or components to create or modify.
   - List concrete test cases.
   - Identify which existing modules this touches and any potential side effects.
   - Check ARCHITECTURE.md and CONVENTIONS.md for relevant patterns to follow.
   - If the scope feels very large, suggest decomposing into an epic via `/epic` instead.

5. WRITE: Save the plan using the template format found in `.plans/_TEMPLATE.md` (if it exists) or this structure:
   - Status: IN_PROGRESS
   - Category: feature | bugfix | refactor | chore (infer from the task description)
   - Branch name
   - Parent branch (main or epic/<epic-slug>)
   - One-liner description
   - Scope, Files to Change, Test Cases, Implementation Notes, Future (Out of Scope)

   If this is a phase of an epic, add to the epic's phases table in `.plans/epics/<epic-slug>/README.md`:
   - Update the phase's status from PLANNING to IN_PROGRESS

6. COMMIT: Commit the plan document: `git add .plans/<slug>.md && git commit -m "plan: <slug>"`
   (If a phase, also add the updated epic README: `git add .plans/epics/<epic-slug>/README.md && git commit -m "epic: update <epic-slug> phases"`)

Present the derived slug and plan for my review before committing. Keep the planning focused — if you find yourself expanding scope, add items to the "Future" section instead.

## Examples

**Invocation:**
```
/scope Add user authentication with JWT tokens
```

**Expected Output:**
Terminal output showing:
- Derived **slug**: `add-user-authentication` (or similar)
- Branch creation: `git checkout -b feat/add-user-authentication`
- Test baseline captured to `.plans/add-user-authentication.baseline-tests.txt`
- Plan document created at `.plans/add-user-authentication.md`

**Plan structure produced:**
```markdown
# Plan: add-user-authentication

**Status:** IN_PROGRESS
**Category:** feature
**Branch:** feat/add-user-authentication
**Parent:** main
**One-liner:** Add user authentication with JWT tokens

## Scope
<generated description>

## Files to Change
- `src/auth/service.ts` — authentication logic
- `src/auth/middleware.ts` — JWT validation middleware

## Test Cases
- [ ] User can login with valid credentials
- [ ] Invalid credentials return 401
```

**Common Variations:**
- `/scope Add health check endpoint to the API` — creates `feat/add-health-check-endpoint`
- `/scope fix CSV export crash when file is empty` — creates `fix/csv-export-crash`
- `/scope refactor database connection pooling` — creates `feat/refactor-database-connection-pooling`
```

### `/epic` — Large Task Decomposition

```markdown
# .kilocode/commands/epic.md
---
description: Decompose a large task into an ordered sequence of scoped phases on a dedicated epic branch
mode: architect
---
I need to plan a large task: "$ARGUMENTS"

This is too big for a single plan. Decompose it into an epic with ordered phases.

Follow this process:

0. NAMING: From the task description above, derive:
   - An **epic slug** — a short kebab-case identifier (2-4 words max) for the epic. Examples: `refactor-auth`, `add-reporting-module`, `migrate-to-postgres`.
   - A **one-liner** — a single sentence describing the full outcome.
   Use the slug for all branch names and file names below.

1. BRANCH: Create a long-lived epic branch from main:
   Run: `git checkout main && git pull && git checkout -b epic/<slug>`
   This branch will accumulate all phase work. Individual phases will branch from and merge back into this epic branch. The epic branch merges to main only when the full epic is complete.

2. DIRECTORY: Create the epic's plan directory:
   Run: `mkdir -p .plans/epics/<slug>`
   This directory will contain the epic's README (the master plan) and all phase plans.

3. UNDERSTAND: Read ARCHITECTURE.md and CONVENTIONS.md to understand the current system.
   Identify all modules, files, and boundaries that this task will touch.

4. DECOMPOSE: Break the task into 2-6 phases, where each phase:
   - Can be implemented, reviewed, and merged independently into the epic branch
   - Leaves the epic branch in a working state after merging (no broken intermediate states)
   - Has clear boundaries — it's obvious what's in vs out of each phase

   Common decomposition strategies:
   - **Layer by layer:** Data model first → service layer → API → UI
   - **Module by module:** Refactor module A → module B → module C → update shared interfaces
   - **Vertical slice:** End-to-end for feature subset 1 → subset 2 → subset 3
   - **Strangler fig:** New implementation alongside old → migration → cleanup old code

5. ORDER: Arrange phases so dependencies flow forward. Each phase should list which earlier phases it depends on.

6. WRITE: Save the epic to `.plans/epics/<slug>/README.md` using the epic template format found in `.plans/_EPIC_TEMPLATE.md`.
   - Status: IN_PROGRESS
   - Epic branch: `epic/<slug>`
   - Fill in the phases table with plan names following the convention: `<slug>-phase-N-<short-description>`
   - Document architecture impact, risks, and exit criteria

7. COMMIT: `git add .plans/epics/<slug>/README.md && git commit -m "epic: <slug>"`

Present the derived slug and decomposition for my review before committing.

Then invoke @reporter to render an epic planning report.

The report should include:
- A stat card row: total phases, estimated files affected, modules touched, risk level
- A mermaid flowchart showing phase dependencies (which phases block which)
- A mermaid gitgraph showing the branching strategy: main → epic/<slug> → feat/phase-1 → merge back → feat/phase-2 → merge back → ... → merge epic to main
- A phase breakdown table with description, files affected, and dependency chain
- An architecture impact section with a before/after mermaid diagram if structural changes are involved
- A risks & open questions section

Save to: `.reports/epic-<slug>-<YYYY-MM-DD>.html`

Do NOT create the individual phase plan documents yet — those will be created one at a time via `/scope` as each phase is reached. The epic document is the roadmap; the phase plans are the turn-by-turn directions.

## Examples

**Invocation:**
```
/epic Refactor authentication system across all services
```

**Expected Output:**
HTML report at `.reports/epic-<slug>-<YYYY-MM-DD>.html` with:
- Stat card row: total phases, estimated files affected, modules touched, risk level
- Mermaid flowchart showing phase dependencies
- Mermaid gitgraph showing branching strategy
- Phase breakdown table
- Architecture impact section (if applicable)
- Risks & open questions

**Decomposition output structure:**
```markdown
# Epic: refactor-auth-system

**Status:** IN_PROGRESS
**Branch:** epic/refactor-auth-system

## Phases

| # | Plan | Description | Dependencies | Status |
|---|------|-------------|-------------|--------|
| 1 | `refactor-auth-system-phase-1-core` | Extract auth core | none | PLANNING |
| 2 | `refactor-auth-system-phase-2-service` | Update service layer | phase 1 | PLANNING |
| 3 | `refactor-auth-system-phase-3-api` | Update API consumers | phase 2 | PLANNING |
```

**Common Variations:**
- `/epic Migrate database from MongoDB to PostgreSQL`
- `/epic Add real-time notification system`
- `/epic Refactor legacy payment processing`
```

## Structured Output

When invoking @reporter, pass this JSON schema:

```json
{
  "report_type": "epic",
  "generated_at": "ISO8601 timestamp",
  "epic_name": "string",
  "epic_slug": "string",
  "stats": {
    "total_phases": "number",
    "estimated_files_affected": "number",
    "modules_touched": ["string"],
    "risk_level": "low | medium | high"
  },
  "phases": [
    {
      "number": "number",
      "plan_name": "string",
      "description": "string",
      "dependencies": ["string"],
      "status": "PLANNING",
      "files_affected": ["string"]
    }
  ],
  "branching_strategy": "gitgraph JSON for mermaid",
  "architecture_impact": {
    "before": "string (mermaid diagram)",
    "after": "string (mermaid diagram)",
    "modules_affected": ["string"]
  },
  "risks": ["string"],
  "open_questions": ["string"]
}
```

### `/implement` — Phase 3: Execute

```markdown
# .kilocode/commands/implement.md
---
description: Continue implementing an existing plan
mode: code
arguments:
  - plan_name
---
Continue implementing the plan in `.plans/$1.md`.

Process:
1. Read the plan document and check its current status.
2. **Check for a fix list:** If `.plans/$1.fixlist.md` exists, this is a post-review fix cycle.
   - Address each issue in the fix list.
   - Check off items as you fix them.
   - Delete the fixlist file when all issues are resolved.
   - Skip the rest of this process — focus only on the fixes.
3. Check git status and recent commits on this branch to understand what's already been done.
4. Run existing tests to see current state: what passes, what fails, what's missing.
5. Continue implementation from where it left off:
   - Write tests first, then implement to make them pass (TDD).
   - Follow patterns in CONVENTIONS.md.
   - Commit after each logical unit of progress with descriptive commit messages.
6. After completing a chunk of work, update the test case checkboxes in the plan doc.

Do NOT:
- Modify files outside the scope defined in the plan.
- Add features not in the plan (note them in the "Future" section instead).
- Refactor unrelated code, even if you notice opportunities.

If the plan turns out to be wrong or insufficient, stop and tell me what needs to change rather than improvising.

## Examples

**Invocation:**
```
/implement add-user-authentication
```

**Expected Output:**
Terminal output showing:
- Reading plan from `.plans/add-user-authentication.md`
- Current status check
- Test run results
- Implementation progress updates as commits are made
- Checkbox updates in plan doc as test cases pass

**Progress update format:**
```
[IN_PROGRESS] add-user-authentication
- Tests run: 12 passed, 2 failing
- Updated: src/auth/service.ts
- Commit: add auth service with JWT validation
- Test cases completed: 3/5
```

**Common Variations:**
- `/implement add-user-authentication` — Standard implementation
- When fixlist exists: `/implement` reads `.plans/add-user-authentication.fixlist.md` and enters fix mode
```

### `/review` — Phase 4a: Diff Review

```markdown
# .kilocode/commands/review.md
---
description: Review the current branch diff against main and generate a review report
mode: architect
arguments:
  - plan_name
---
Review the implementation on this branch against the plan in `.plans/$1.md`.

Gather the following data:

1. Run `git diff main..HEAD` and `git diff --stat main..HEAD` to see all changes.
2. Read the plan document to understand what was intended.
3. **Test suite with baseline comparison:**
   - Run the test suite using the test command from the `## Commands` section of `AGENTS.md` and capture current results.
   - Read the test baseline from `.plans/$1.baseline-tests.txt` (captured during `/scope`).
   - Compare: identify which failures are **pre-existing** (present in baseline) vs **new** (introduced by this branch).
   - Only flag new failures as blocking issues. Pre-existing failures should be noted but NOT count against the verdict.
4. Evaluate:
   - **Plan alignment:** Does the implementation match what was planned? List deviations.
   - **Scope check:** Are there changes to files NOT listed in the plan? Flag each.
   - **Code quality:** Check naming, patterns, consistency with CONVENTIONS.md.
   - **Test coverage:** Are all planned test cases present and passing?
   - **Completeness:** Any TODOs, placeholder code, or commented-out blocks?
   - **Edge cases:** Obvious error handling gaps or uncovered edge cases?
5. Determine a verdict: **PASS** or **NEEDS WORK**.

If NEEDS WORK, also write a fix list to `.plans/$1.fixlist.md`:
```
# Fix List: $1

## Issues to Address

- [ ] Issue 1: <description> — `file.ts:L42`
  **Why this failed:** <root cause explanation — what the code does wrong and why>
  **How to fix:** <specific guidance on the approach to take>

- [ ] Issue 2: <description> — `file.ts:L87`
  **Why this failed:** <root cause explanation>
  **How to fix:** <specific guidance>

## Context for Fix Session

<Overall explanation of what went wrong. Include:
- Which parts of the plan were implemented correctly
- Where the implementation diverged or fell short
- Any patterns in the issues (e.g., "all issues are related to null handling in the parser")
- Relevant code context the implementing agent needs to understand the fix>
```

Then invoke @reporter to render a review report.

The report should include:
- A stat card row: files changed, lines added/removed, new tests passing/failing (excluding pre-existing failures), verdict (PASS as green tag, NEEDS WORK as yellow tag)
- A "Diff Summary" table: filename, lines changed, planned vs unplanned (tag)
- A "Review Findings" section with numbered issues, each with file reference, severity tag (🔴 blocking, 🟡 minor, 🟢 suggestion), root cause, and fix guidance
- A "Test Results" section with two parts: **New/Changed** (tests affected by this branch — these determine the verdict) and **Pre-existing Failures** (failures present in the baseline — noted for awareness but not blocking). Show the baseline comparison clearly.
- If there are scope deviations, a mermaid diagram showing planned files vs actual files changed
- A "Live Verification" section if UI changes were involved — embed any screenshots from `.reports/screenshots/review-$1-*` as base64 images with captions describing what was verified
- A "Verdict" section with clear PASS/NEEDS WORK and summary

Save to: `.reports/review-$1-<YYYY-MM-DD>.html`

## Examples

**Invocation:**
```
/review add-user-authentication
```

**Expected Output:**
HTML report at `.reports/review-<plan-name>-<YYYY-MM-DD>.html` with:
- Stat card row: files changed, lines added/removed, new tests passing/failing (excluding pre-existing failures), verdict (PASS as green tag, NEEDS_WORK as yellow tag)
- "Diff Summary" table: filename, lines changed, planned vs unplanned (tag)
- "Review Findings" section with numbered issues, each with file reference, severity tag (🔴 blocking, 🟡 minor, 🟢 suggestion), root cause, and fix guidance
- "Test Results" section with baseline comparison
- "Verdict" section with clear PASS/NEEDS_WORK and summary

**PASS Verdict Example:**
```
[VERDICT: PASS] ✅

Stat cards show:
- 5 files changed
- +127 -43 lines
- 12/12 tests passing
- Tag: "PASS" (green)

Review Findings: No issues found. Implementation matches plan.
```

**NEEDS_WORK Verdict Example:**
```
[VERDICT: NEEDS_WORK] ⚠️

Stat cards show:
- 6 files changed (1 unplanned)
- +142 -67 lines
- 10/12 tests passing (2 new failures)
- Tag: "NEEDS_WORK" (yellow)

Review Findings:
1. 🔴 blocking: Null pointer in auth service — src/auth/service.ts:L42
   Why: Missing null check on user object
   How to fix: Add optional chaining or null guard

2. 🟡 minor: Unplanned file changed — src/utils/logger.ts
   Why: Not listed in plan's "Files to Change"
```

**Fixlist Output (`.plans/<plan-name>.fixlist.md`):**
```markdown
# Fix List: add-user-authentication

## Issues to Address

- [ ] Issue 1: Null pointer in auth service — `src/auth/service.ts:L42`
  **Why this failed:** Missing null check on user object
  **How to fix:** Add optional chaining or null guard

## Context for Fix Session

<Overall explanation of what went wrong>
```

**Common Variations:**
- `/review` — Review current branch against its plan
- When tests fail: verdict is NEEDS_WORK, fixlist.md is generated
- When scope deviations exist: unplanned files flagged in findings
```

## Structured Output

When invoking @reporter, pass this JSON schema:

```json
{
  "report_type": "review",
  "generated_at": "ISO8601 timestamp",
  "plan_name": "string",
  "verdict": "PASS | NEEDS_WORK",
  "stats": {
    "files_changed": "number",
    "lines_added": "number",
    "lines_removed": "number",
    "tests_passed": "number",
    "tests_failed": "number",
    "new_tests_passed": "number",
    "new_tests_failed": "number",
    "pre_existing_failures": "number"
  },
  "diff_summary": [
    {
      "file": "string",
      "lines_added": "number",
      "lines_removed": "number",
      "planned": "boolean"
    }
  ],
  "issues": [
    {
      "severity": "blocking | minor | suggestion",
      "file": "string",
      "line": "number",
      "description": "string",
      "root_cause": "string",
      "fix_guidance": "string"
    }
  ],
  "scope_deviations": ["string"],
  "live_verification": {
    "checked": ["string"],
    "passed": ["string"],
    "failed": ["string"]
  },
  "screenshots": ["path"],
  "pre_existing_failures_list": ["string"]
}
```

### `/sync` — Phase 4b: Post-Merge Doc Sync

```markdown
# .kilocode/commands/sync.md
---
description: Update project docs after merging a feature branch
mode: code
arguments:
  - plan_name
---
The feature branch for `$1` has been merged to main. Update the project knowledge base.

1. **ARCHITECTURE.md**: Read the current doc and the changes from this feature.
   - If any structural changes were made (new modules, changed boundaries, new dependencies), update the relevant sections.
   - If no structural changes, leave it alone — don't add noise.
   - Update the "Last updated" date only if you made changes.

2. **CONVENTIONS.md**: If any new patterns were established by this feature (new error handling approach, new testing pattern, new naming convention), document them.
   - Only add genuinely new conventions, not one-off decisions.

3. **Plan document** (`.plans/$1.md` or `.plans/epics/<epic>/$1.md`):
   - Set Status to COMPLETE.
   - Fill in the Completion Log section: date, actual changes summary, whether implementation deviated from plan, and impact on other modules.

   If this was a phase of an epic:
   - Update the epic's README.md phases table: set this phase's status to COMPLETE and fill in the "Completed" date.

4. **Backlog**: If the plan's "Future (Out of Scope)" section has items, append them to `.plans/backlog.md`.

5. **Commands Registry** (`.kilocode/commands/commands.md`):
   - If the merged feature added a new command to `.kilocode/commands/`, add it to the commands table in `commands.md`.
   - If a command was modified in a way that changes its description, update the corresponding entry.
   - If a command was removed, remove its entry.
   - Only make changes if the merge actually added, modified, or removed commands.

6. **Commit**: Stage and commit all doc changes: `git add -A && git commit -m "docs: sync after $1 merge"`

Be concise in all updates. These docs should be scannable, not exhaustive.

## Examples

**Invocation:**
```
/sync add-user-authentication
```

**Expected Output:**
Terminal output showing:
- ARCHITECTURE.md updates (if structural changes)
- CONVENTIONS.md updates (if new patterns)
- Plan document marked COMPLETE with completion log filled
- Backlog items appended (if any)
- Commands registry updated (if applicable)
- Commit: `docs: sync after add-user-authentication merge`

**Before/After Example (ARCHITECTURE.md):**
```markdown
<!-- Before -->
## Module Boundaries

| Module | Responsibility | Key Files |
|--------|---------------|-----------|
| auth | User authentication | auth/service.ts |

<!-- After -->
## Module Boundaries

| Module | Responsibility | Key Files |
|--------|---------------|-----------|
| auth | User authentication with JWT | auth/service.ts, auth/middleware.ts |
| middleware | Request/response middleware | middleware/jwt.ts |
```

**Before/After Example (CONVENTIONS.md):**
```markdown
<!-- Before (no existing convention) -->

<!-- After (new pattern added) -->
## Error Handling

- Use try/catch with typed errors in service layer
- Propagate errors with context, not raw messages
```

**Commit Message:**
```
docs: sync after add-user-authentication merge
```

**Common Variations:**
- `/sync` — Sync docs for most recently merged plan
- After epic phase merge: updates epic README phases table
- When no structural changes: minimal output, only plan completion
```

### `/hotfix` — Escape Hatch: Quick Fix

```markdown
# .kilocode/commands/hotfix.md
---
description: Quick fix on a hotfix branch — skip planning, still review+sync
mode: code
---
Quick hotfix: "$ARGUMENTS"

1. Derive a short kebab-case slug from the description above (e.g., `fix-null-pointer-auth`, `fix-csv-export-crash`).
2. Create branch: `git checkout main && git pull && git checkout -b fix/<slug>`
3. Implement the fix directly. Keep changes minimal and focused.
4. Run tests to confirm the fix works and nothing else broke.
5. Commit with message: `fix: <slug>`

After implementation, I'll run /review and /sync manually.
Do NOT expand scope beyond the immediate fix.

## Examples

**Invocation:**
```
/hotfix Fix null pointer crash in auth service
```

**Expected Output:**
Terminal output showing:
- Slug derivation: `fix-null-pointer-crash-auth-service`
- Branch creation: `git checkout -b fix/fix-null-pointer-crash-auth-service`
- Minimal implementation focused on the fix
- Test run: all tests passing
- Commit: `fix: fix-null-pointer-crash-auth-service`

**Slug Derivation Examples:**
```
"Fix CSV export crash" → fix-csv-export-crash
"Null pointer in auth handler" → fix-null-pointer-auth-handler
"Memory leak in worker" → fix-memory-leak-worker
```

**Minimal Flow:**
```
[1] Deriving slug: fix-null-pointer-crash-auth-service
[2] Creating branch: fix/fix-null-pointer-crash-auth-service
[3] Implementing fix (minimal changes only)
[4] Running tests: 24 passed, 0 failed
[5] Commit: fix: fix-null-pointer-crash-auth-service

Done. Run /review and /sync manually after merging.
```

**Common Variations:**
- `/hotfix Fix race condition in payment processor` → `fix-race-condition-payment-processor`
- Minimal scope: only the immediate bug, no feature expansion
```

### `/recap` — End-of-Session Summary

```markdown
# .kilocode/commands/recap.md
---
description: Generate an end-of-session summary report
mode: architect
---
Generate an end-of-session recap by gathering the following data:

1. **Git activity this session:**
   - Run `git log --oneline --since="4 hours ago"` (adjust if needed).
   - Summarize what was built, fixed, or changed.

2. **Plan progress:**
   - Check all `.plans/*.md` files for current status.
   - For any IN_PROGRESS plans: what's done, what's remaining.

3. **Unfinished threads:**
   - Any failing tests? (Run the test command from `AGENTS.md`, capture summary)
   - Any TODOs added this session? (`git diff main..HEAD | grep -i "TODO"`)
   - Any items added to the "Future" section of plan docs?

4. **Recommended next steps:**
   - What should the next session start with?
   - Any blockers or decisions needed before continuing?

Then invoke @reporter to render a recap report.

The report should include:
- A stat card row: commits this session, files changed, tests passing/failing, open plans count
- A "Session Activity" section with a timeline or table of commits
- An "Active Plans" section showing each plan's progress (use a progress indicator or checklist completion %)
- An "Unfinished Threads" section — only if there are failing tests, TODOs, or blockers
- A "Next Session" section with 1-3 prioritized recommended actions
- If multiple plans are active, a mermaid gantt or status diagram showing their states

Save to: `.reports/recap-<YYYY-MM-DD>.html`

## Examples

**Invocation:**
```
/recap
```

**Expected Output:**
HTML report at `.reports/recap-<YYYY-MM-DD>.html` with:
- Stat card row: commits this session, files changed, tests passing/failing, open plans count
- "Session Activity" section: timeline or table of commits
- "Active Plans" section: each plan's progress with progress indicator or checklist completion %
- "Unfinished Threads" section (if applicable): failing tests, TODOs, blockers
- "Next Session" section: 1-3 prioritized recommended actions

**HTML Report Structure:**
```html
<!-- Stat cards -->
<div class="stat-grid">
  <div class="stat-card">
    <div class="value">5</div>
    <div class="label">Commits</div>
  </div>
  <div class="stat-card">
    <div class="value">12</div>
    <div class="label">Files Changed</div>
  </div>
  <div class="stat-card">
    <div class="value">28/30</div>
    <div class="label">Tests Passing</div>
  </div>
  <div class="stat-card">
    <div class="value">2</div>
    <div class="label">Open Plans</div>
  </div>
</div>

<!-- Session Activity -->
<h2>Session Activity</h2>
<table>
  <tr><td>add auth service with JWT validation</td></tr>
  <tr><td>add middleware for request validation</td></tr>
  <tr><td>test baseline: add-user-authentication</td></tr>
</table>

<!-- Active Plans -->
<h2>Active Plans</h2>
<div>
  <h3>add-user-authentication</h3>
  <p>Progress: 4/5 test cases</p>
  <p>Status: IN_REVIEW</p>
</div>

<!-- Unfinished Threads (if any) -->
<h2>Unfinished Threads</h2>
<ul>
  <li>2 tests failing in auth/service.spec.ts</li>
</ul>

<!-- Next Session -->
<h2>Next Session</h2>
<ol>
  <li>Fix auth service null handling issue</li>
  <li>Re-run /review after fixes</li>
</ol>
```

**Common Variations:**
- `/recap` — End of work session summary
- Works best after 1+ hours of work with commits to analyze
```

## Structured Output

When invoking @reporter, pass this JSON schema:

```json
{
  "report_type": "recap",
  "generated_at": "ISO8601 timestamp",
  "stats": {
    "commits_this_session": "number",
    "files_changed": "number",
    "tests_passing": "string",
    "tests_failing": "number",
    "open_plans": "number"
  },
  "session_activity": [
    {
      "commit_hash": "string",
      "message": "string",
      "timestamp": "ISO8601"
    }
  ],
  "active_plans": [
    {
      "name": "string",
      "status": "IN_PROGRESS | IN_REVIEW",
      "progress": "string (e.g., '3/5 test cases')",
      "checklist_completion": "number (percentage)"
    }
  ],
  "unfinished_threads": {
    "failing_tests": ["string"],
    "new_todos": ["string"],
    "blockers": ["string"]
  },
  "recommended_next_actions": ["string"]
}
```

### `/impact` — Dependency Impact Analysis

```markdown
# .kilocode/commands/impact.md
---
description: Analyze and visualize the dependency impact of a planned change
mode: architect
arguments:
  - plan_name
---
Analyze the dependency impact of the plan in `.plans/$1.md`.

For each file listed in the plan's "Files to Change" section:

1. **Import graph:** What other files import from this file? Trace 2 levels deep.
2. **Export surface:** What functions, classes, or types does this file export that others depend on?
3. **Test coverage:** Which test files cover this module? Are there integration tests?
4. **Shared module risk:** Does this touch shared/core modules? If so, flag the blast radius.

Then invoke @reporter to render an impact analysis report.

The report should include:
- A stat card row: files in plan, direct dependents count, indirect dependents count, risk level (🟢/🟡/🔴 as colored tag)
- A **mermaid dependency graph** showing: planned files (highlighted) → direct dependents → indirect dependents. Use different node colors for planned changes vs affected files.
- A "File Impact Table": for each planned file, list its exports, direct dependents, and test coverage status
- A "Risk Assessment" section with per-file risk tags and explanation
- If risk is 🔴, a "Suggested Splits" section recommending how to break the plan into smaller pieces

Save to: `.reports/impact-$1-<YYYY-MM-DD>.html`

## Examples

**Invocation:**
```
/impact add-user-authentication
```

**Expected Output:**
HTML report at `.reports/impact-<plan-name>-<YYYY-MM-DD>.html` with:
- Stat card row: files in plan, direct dependents count, indirect dependents count, risk level (🟢/🟡/🔴 as colored tag)
- Mermaid dependency graph: planned files (highlighted) → direct dependents → indirect dependents
- "File Impact Table": for each planned file, list exports, direct dependents, test coverage status
- "Risk Assessment" section with per-file risk tags
- "Suggested Splits" section (if risk is 🔴)

**HTML Report Structure:**
```html
<!-- Stat cards -->
<div class="stat-grid">
  <div class="stat-card">
    <div class="value">3</div>
    <div class="label">Files in Plan</div>
  </div>
  <div class="stat-card">
    <div class="value">7</div>
    <div class="label">Direct Dependents</div>
  </div>
  <div class="stat-card">
    <div class="value">12</div>
    <div class="label">Indirect Dependents</div>
  </div>
  <div class="stat-card">
    <span class="tag tag-yellow">🟡 MEDIUM</span>
  </div>
</div>

<!-- Mermaid dependency graph -->
<pre class="mermaid">
graph TD
  auth/service.ts --> auth/middleware.ts
  auth/service.ts --> api/auth.ts
  auth/middleware.ts --> middleware/jwt.ts
  auth/middleware.ts --> middleware/logger.ts
</pre>

<!-- File Impact Table -->
<table>
  <tr><th>File</th><th>Exports</th><th>Dependents</th><th>Test Coverage</th></tr>
  <tr><td>auth/service.ts</td><td>AuthService, TokenPayload</td><td>7 files</td><td>auth/service.spec.ts</td></tr>
</table>

<!-- Risk Assessment -->
<h2>Risk Assessment</h2>
<p><span class="tag tag-yellow">🟡 auth/service.ts</span> — 7 direct dependents, changing JWT validation</p>
```

**Common Variations:**
- `/impact` — Analyze current branch plan for risk assessment
- High risk plans show "Suggested Splits" to break into smaller pieces
```

## Structured Output

When invoking @reporter, pass this JSON schema:

```json
{
  "report_type": "impact",
  "generated_at": "ISO8601 timestamp",
  "plan_name": "string",
  "stats": {
    "files_in_plan": "number",
    "direct_dependents": "number",
    "indirect_dependents": "number",
    "risk_level": "low | medium | high"
  },
  "dependency_graph": "mermaid graph TD format showing planned files → direct → indirect",
  "file_impact": [
    {
      "file": "string",
      "exports": ["string"],
      "direct_dependents": "number",
      "test_coverage": "string (file path or 'none')"
    }
  ],
  "risk_assessment": [
    {
      "file": "string",
      "risk_tag": "🟢 low | 🟡 medium | 🔴 high",
      "explanation": "string",
      "blast_radius": "string"
    }
  ],
  "suggested_splits": [
    {
      "phase": "string",
      "files": ["string"],
      "rationale": "string"
    }
  ]
}
```

### `/estimate` — Planned vs. Actual Analysis

```markdown
# .kilocode/commands/estimate.md
---
description: Analyze estimation accuracy across completed plans and generate a calibration report
mode: architect
---
Analyze completed plans to identify estimation patterns.

1. **Scan `.plans/` for all COMPLETE plans** that have a filled-in Completion Log.

2. **For each completed plan, extract:**
   - Category (feature, bugfix, refactor, chore)
   - Number of files planned vs. actually changed
   - Planned test cases vs. actual test count
   - Whether implementation deviated from plan (and how)
   - Time from plan creation to completion (from git dates)

3. **Identify patterns — group by category:**
   - Do plans consistently underestimate file count? By how much, per category?
   - Which categories deviate most from their plans?
   - Common categories of unplanned work?
   - Typical ratio of planned scope to actual scope, per category?
   - Are refactors consistently underestimated? Do bugfixes take longer than features of similar size?

Then invoke @reporter to render an estimation calibration report.

The report should include:
- A stat card row: total completed plans, average scope accuracy %, average deviation, most common surprise category
- A "Plan Accuracy" table: plan name, category (as colored tag), planned files vs actual, planned tests vs actual, deviated (yes/no), duration
- A mermaid pie chart showing plan distribution by category
- A category breakdown: for each category, show average accuracy, average duration, and common deviation patterns
- A "Patterns" section with specific calibration insights
- A "Calibration Advice" section with concrete rules of thumb for future scoping

Save to: `.reports/estimate-<YYYY-MM-DD>.html`

## Examples

**Invocation:**
```
/estimate
```

**Expected Output:**
HTML report at `.reports/estimate-<YYYY-MM-DD>.html` with:
- Stat card row: total completed plans, average scope accuracy %, average deviation, most common surprise category
- "Plan Accuracy" table: plan name, category (as colored tag), planned files vs actual, planned tests vs actual, deviated (yes/no), duration
- Mermaid pie chart showing plan distribution by category
- Category breakdown: for each category, show average accuracy, average duration, and common deviation patterns
- "Patterns" section with specific calibration insights
- "Calibration Advice" section with concrete rules of thumb for future scoping

**HTML Report Structure:**
```html
<!-- Stat cards -->
<div class="stat-grid">
  <div class="stat-card">
    <div class="value">15</div>
    <div class="label">Completed Plans</div>
  </div>
  <div class="stat-card">
    <div class="value">78%</div>
    <div class="label">Avg Scope Accuracy</div>
  </div>
  <div class="stat-card">
    <div class="value">+2.3</div>
    <div class="label">Avg Files Deviation</div>
  </div>
  <div class="stat-card">
    <span class="tag tag-red">🔴 refactor</span>
    <div class="label">Most Surprising</div>
  </div>
</div>

<!-- Plan Accuracy Table -->
<table>
  <tr><th>Plan</th><th>Category</th><th>Planned</th><th>Actual</th><th>Deviated</th></tr>
  <tr><td>add-user-authentication</td><td><span class="tag tag-green">feature</span></td><td>3 files</td><td>4 files</td><td>yes</td></tr>
  <tr><td>fix-csv-export-crash</td><td><span class="tag tag-blue">bugfix</span></td><td>1 file</td><td>1 file</td><td>no</td></tr>
</table>

<!-- Category breakdown -->
<h2>Category Breakdown</h2>
<p><strong>feature:</strong> 82% avg accuracy, 3.2 days avg duration</p>
<p><strong>bugfix:</strong> 91% avg accuracy, 1.1 days avg duration</p>
<p><strong>refactor:</strong> 65% avg accuracy, 5.8 days avg duration — commonly underestimated</p>

<!-- Calibration Advice -->
<h2>Calibration Advice</h2>
<ul>
  <li>Refactors take 1.5x longer than features of similar initial scope</li>
  <li>Add 1-2 buffer files to every feature plan for integration work</li>
  <li>Bugfixes are usually 1 file, but always check for test coverage gaps</li>
</ul>
```

**Common Variations:**
- `/estimate` — Full calibration report across all completed plans
- Works best when you have 5+ completed plans with filled-in Completion Logs
```

## Structured Output

When invoking @reporter, pass this JSON schema:

```json
{
  "report_type": "estimate",
  "generated_at": "ISO8601 timestamp",
  "stats": {
    "total_completed_plans": "number",
    "avg_scope_accuracy_percent": "number",
    "avg_files_deviation": "number",
    "most_common_surprise_category": "feature | bugfix | refactor | chore"
  },
  "plan_accuracy_table": [
    {
      "plan_name": "string",
      "category": "feature | bugfix | refactor | chore",
      "planned_files": "number",
      "actual_files": "number",
      "planned_tests": "number",
      "actual_tests": "number",
      "deviated": "boolean",
      "duration_days": "number"
    }
  ],
  "category_distribution": {
    "feature": "number",
    "bugfix": "number",
    "refactor": "number",
    "chore": "number"
  },
  "category_breakdown": [
    {
      "category": "string",
      "avg_accuracy_percent": "number",
      "avg_duration_days": "number",
      "common_deviation_pattern": "string"
    }
  ],
  "patterns": ["string"],
  "calibration_advice": ["string"]
}
```

---

## 4. Custom Subagents

Subagents are specialized agents defined as markdown files with YAML frontmatter. They run in isolated sessions with their own prompts, models, and permissions. Place them in `.kilo/agents/` (project-level) or `~/.config/kilo/agents/` (global). The filename (minus `.md`) becomes the agent name.

You can invoke them manually with `@agent-name` in the CLI, or primary agents can invoke them automatically via the Task tool.

### `reviewer` — Phase 4a: Diff Review Agent

```markdown
# .kilo/agents/reviewer.md
---
description: Reviews the current feature branch diff against main and its plan document. Includes live verification via agent-browser when a UI is involved. Read-only — cannot edit files.
mode: subagent
temperature: 0.1
permission:
  edit: deny
  bash:
    "*": deny
    "git diff*": allow
    "git log*": allow
    "git show*": allow
    "npm test*": allow
    "npm run test*": allow
    "npx jest*": allow
    "cargo test*": allow
    "pytest*": allow
    "python -m pytest*": allow
    "go test*": allow
    "make test*": allow
    "agent-browser *": allow
---

## Role

You are a senior code reviewer. Your job is to review the diff of the current feature branch against main, comparing the implementation to its plan document in `.plans/`. Read-only — cannot edit files.

## Expertise Areas

- Diff analysis and scope verification
- Test suite execution and coverage assessment
- Code quality evaluation (naming, patterns, consistency with CONVENTIONS.md)
- UI verification via agent-browser for visual components
- Edge case identification

## Review Process

Read the `## Commands` section of `AGENTS.md` to determine the correct test command for this project. Do not assume any specific test runner.

1. **UNDERSTAND** — Read the plan document to understand what was intended.
2. **DIFF** — Run `git diff main..HEAD` to see all changes.
3. **TEST** — Run the test suite to confirm tests pass.
4. **EVALUATE** — Evaluate the implementation against these criteria:

**Plan alignment:** Does the implementation match what was planned? List deviations.
**Scope check:** Are there changes to files NOT listed in the plan? Flag each one.
**Code quality:** Check naming, patterns, consistency with CONVENTIONS.md.
**Test coverage:** Are all planned test cases present and passing?
**Completeness:** Any TODOs, placeholder code, or commented-out blocks?
**Edge cases:** Obvious error handling gaps or uncovered edge cases?

5. **VERIFY** — **Live verification (when applicable):** If the plan involves UI changes (components, pages, styles, routes), verify them in the running app using agent-browser. Read the dev server URL from the `## Commands` section of `AGENTS.md`.

```bash
# Navigate to the relevant page (use dev server URL from AGENTS.md)
agent-browser open <dev-server-url>/<relevant-route>
agent-browser wait --load networkidle

# Take a screenshot as evidence for the review report
agent-browser screenshot .reports/screenshots/review-<plan-name>-<context>.png

# Snapshot interactive elements and verify the expected UI is present
agent-browser snapshot -i

# Check specific elements if the plan mentions them
agent-browser get text @e<n>

# If the feature involves forms, navigation, or interactions — walk through them
# Document what works and what doesn't
```

Skip live verification if:
- The plan is backend-only (no UI changes)
- The dev server is not running (note this in the review, don't fail for it)
- The changes are purely test/config/docs

When you take screenshots during live verification, save them to `.reports/screenshots/` — they will be embedded in the review report by @reporter.

6. **VERDICT** — Provide the final review assessment.

## Output Format

Provide a summary:
- **Verdict: PASS** or **NEEDS WORK**
- **Issues:** Numbered list with file and line references
- **Live verification result:** What was checked in the browser, what passed, what failed (if applicable)
- **Screenshots:** List paths to any screenshots taken
- **Suggestions:** Optional non-blocking improvements

Be thorough but pragmatic. Minor style issues are not blockers.

## Constraints

- Never edit or create files. Only read, analyze, and report.
- Always reference the plan document when evaluating.
- Always run the test suite as part of the review.
- Only use agent-browser for verification — never use it to make changes.
```

### `syncer` — Phase 4b: Post-Merge Doc Sync Agent

```markdown
# .kilo/agents/syncer.md
---
description: Updates ARCHITECTURE.md, CONVENTIONS.md, and plan docs after a feature branch merge. Only edits markdown files.
mode: subagent
permission:
  edit:
    "*": deny
    "*.md": allow
    "*.mdx": allow
  bash:
    "*": deny
    "git add*": allow
    "git commit*": allow
    "git log*": allow
    "git diff*": allow
---

## Role

You are a documentation maintainer. After a feature has been merged to main, you update the project's living documentation to reflect what was actually built.

## Expertise Areas

- ARCHITECTURE.md maintenance for structural changes
- CONVENTIONS.md updates for new patterns
- Plan document completion and status tracking
- Backlog management

## Sync Process

1. **DISCOVER** — Run `git log main~5..main --oneline` to see what was just merged.
2. **READ** — Read the relevant plan document in `.plans/`.
3. **UPDATE ARCHITECTURE** — If structural changes were made (new modules, changed boundaries, new dependencies), update ARCHITECTURE.md. If no structural changes, leave it alone. Update "Last updated" only if you made changes.
4. **UPDATE CONVENTIONS** — If new patterns were established (new error handling approach, testing pattern, naming convention), document them in CONVENTIONS.md. Only add genuinely new conventions, not one-off decisions.
5. **COMPLETE PLAN** — Set Status to COMPLETE. Fill in the Completion Log: date, actual changes summary, deviation from plan, impact on other modules.
6. **UPDATE BACKLOG** — If the plan's "Future (Out of Scope)" section has items, append them to `.plans/backlog.md`.
7. **COMMIT** — Run `git add -A && git commit -m "docs: sync after merge"`

## Output Format

Provide a brief summary of changes made:
- Which documents were updated (ARCHITECTURE.md, CONVENTIONS.md, plan document)
- Any items added to the backlog
- Confirmation of commit

## Constraints

- Only edit markdown files. Never touch source code.
- Be concise — these docs should be scannable, not exhaustive.
- Don't add noise. If nothing changed structurally, don't update ARCHITECTURE.md just to update it.
```

### `reporter` — Human Interface Report Generator

This is the most important subagent in the framework. Every piece of context that needs to be transferred from the agents to you — briefs, recaps, reviews, impact analyses, estimates — is rendered by the reporter as a polished HTML report. The reporter is the universal rendering layer between the BSER system and the human developer.

The slash commands (`/brief`, `/recap`, `/review`, `/impact`, `/estimate`) gather data, then invoke `@reporter` to render it. You can also invoke `@reporter` directly for ad-hoc reports.

```markdown
# .kilo/agents/reporter.md
---
description: The human interface layer for the BSER framework. Renders all developer-facing context (briefs, recaps, reviews, impact analyses, estimates) as polished HTML+CSS reports with mermaid diagrams. Invoked by other commands to present their findings visually. Can also be invoked directly for ad-hoc reports.
mode: subagent
permission:
  edit:
    "*": deny
    ".reports/*.html": allow
    ".reports/**/*.html": allow
  bash:
    "*": deny
    "git log*": allow
    "git diff*": allow
    "git shortlog*": allow
    "git rev-list*": allow
    "find*": allow
    "wc*": allow
    "grep*": allow
    "cat .plans/*": allow
    "cat .bser-version": allow
    "ls*": allow
    "mkdir*": allow
    "open *": allow
    "xdg-open *": allow
    "agent-browser *": allow
    "base64 *": allow
    "cat .reports/screenshots/*": allow
---

You are a report generator. You create polished, standalone HTML reports with embedded CSS and mermaid diagrams. Reports are self-contained single-file HTML documents that can be opened in any browser.

You also have access to `agent-browser` for capturing screenshots of live applications, dashboards, or any web content that should be included in a report. Use this when asked to include visual evidence or when generating reports that benefit from live screenshots (e.g., deployment verification, dashboard snapshots).

## Role

The human interface layer for the BSER framework. Every piece of context that needs to be transferred from the agents to the human developer — briefs, recaps, reviews, impact analyses, estimates — is rendered by the reporter as a polished HTML report. The reporter is the universal rendering layer between the BSER system and the human developer.

The slash commands (`/brief`, `/recap`, `/review`, `/impact`, `/estimate`) gather data, then invoke `@reporter` to render it. You can also invoke `@reporter` directly for ad-hoc reports.

## Expertise Areas

- HTML/CSS report generation with dark mode support
- Mermaid diagram rendering (architecture, timeline, flow, git graphs)
- Screenshot capture and base64 embedding
- Data visualization (stat cards, tables, charts)
- BSER version tracking and footer inclusion

## Report Process

1. Determine the report type from context or ask.
2. Gather required data from project files, git history, and screenshots.
3. Read the BSER version for the footer: `BSER_VERSION=$(cat .bser-version 2>/dev/null | grep "^version:" | cut -d' ' -f2 || echo "unknown")`
4. Build the HTML report using the Report Template.
5. Save to `.reports/` with a descriptive name.
6. Open the report in the default browser automatically.

## Output Format

**File location:** `.reports/` in the project root. Create the directory if it doesn't exist.
**Screenshot location:** `.reports/screenshots/`. Embed existing screenshots from reviewer agent.
**File naming:** `.reports/project-brief-2025-01-15.html`, `.reports/sprint-recap.html`, etc.

**Always open the report in the default browser after creating it:**
```bash
# macOS
open .reports/report-name.html

# Linux
xdg-open .reports/report-name.html
```
Detect the platform and use the appropriate command. This is non-negotiable — every report must be opened automatically so the developer sees it immediately.

## Report Template

Before generating a report, read the BSER version:

```bash
# Read the BSER version for inclusion in the footer
BSER_VERSION=$(cat .bser-version 2>/dev/null | grep "^version:" | cut -d' ' -f2 || echo "unknown")
```

Every report must use this base structure:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{{REPORT_TITLE}}</title>
  <script src="https://cdn.jsdelivr.net/npm/mermaid/dist/mermaid.min.js"></script>
  <script>mermaid.initialize({startOnLoad: true, theme: 'neutral'});</script>
  <style>
    :root {
      --bg: #ffffff;
      --fg: #1a1a2e;
      --accent: #e76f51;
      --accent2: #2d6a4f;
      --muted: #6c757d;
      --border: #dee2e6;
      --surface: #f8f9fa;
      --font: 'Segoe UI', system-ui, -apple-system, sans-serif;
      --mono: 'SF Mono', 'Fira Code', 'Consolas', monospace;
    }
    @media (prefers-color-scheme: dark) {
      :root {
        --bg: #1a1a2e;
        --fg: #e6e6e6;
        --accent: #e76f51;
        --accent2: #52b788;
        --muted: #9ca3af;
        --border: #2d2d44;
        --surface: #16213e;
      }
    }
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body {
      font-family: var(--font);
      background: var(--bg);
      color: var(--fg);
      line-height: 1.6;
      max-width: 900px;
      margin: 0 auto;
      padding: 2rem 1.5rem;
    }
    h1 { font-size: 1.8rem; margin-bottom: 0.25rem; }
    h2 { font-size: 1.3rem; margin-top: 2rem; margin-bottom: 0.75rem; color: var(--accent2); border-bottom: 2px solid var(--border); padding-bottom: 0.25rem; }
    h3 { font-size: 1.1rem; margin-top: 1.25rem; margin-bottom: 0.5rem; }
    p, li { margin-bottom: 0.5rem; }
    .subtitle { color: var(--muted); font-size: 0.95rem; margin-bottom: 2rem; }
    .stat-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(160px, 1fr)); gap: 1rem; margin: 1rem 0; }
    .stat-card { background: var(--surface); border: 1px solid var(--border); border-radius: 8px; padding: 1rem; text-align: center; }
    .stat-card .value { font-size: 1.8rem; font-weight: 700; color: var(--accent); }
    .stat-card .label { font-size: 0.8rem; color: var(--muted); text-transform: uppercase; letter-spacing: 0.05em; }
    table { width: 100%; border-collapse: collapse; margin: 1rem 0; font-size: 0.9rem; }
    th, td { padding: 0.5rem 0.75rem; border: 1px solid var(--border); text-align: left; }
    th { background: var(--surface); font-weight: 600; }
    .tag { display: inline-block; padding: 0.15rem 0.5rem; border-radius: 4px; font-size: 0.75rem; font-weight: 600; }
    .tag-green { background: #d4edda; color: #155724; }
    .tag-yellow { background: #fff3cd; color: #856404; }
    .tag-red { background: #f8d7da; color: #721c24; }
    .tag-blue { background: #d1ecf1; color: #0c5460; }
    code { font-family: var(--mono); font-size: 0.85em; background: var(--surface); padding: 0.15rem 0.35rem; border-radius: 3px; }
    pre { background: var(--surface); border: 1px solid var(--border); border-radius: 6px; padding: 1rem; overflow-x: auto; margin: 1rem 0; }
    pre code { background: none; padding: 0; }
    .mermaid { margin: 1.5rem 0; text-align: center; }
    .screenshot-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(400px, 1fr)); gap: 1rem; margin: 1rem 0; }
    .screenshot { border: 1px solid var(--border); border-radius: 6px; overflow: hidden; }
    .screenshot img { width: 100%; display: block; }
    .screenshot .caption { padding: 0.5rem 0.75rem; font-size: 0.8rem; color: var(--muted); background: var(--surface); }
    footer { margin-top: 3rem; padding-top: 1rem; border-top: 1px solid var(--border); font-size: 0.8rem; color: var(--muted); }
  </style>
</head>
<body>
  <!-- Report content here -->
  <footer>Generated by BSER Reporter · {{DATE}} · BSER {{BSER_VERSION}}</footer>
</body>
</html>
```

## Mermaid Diagrams

Use mermaid for all visual elements. Common patterns:

- **Architecture diagrams:** `graph TD` or `graph LR` for module relationships
- **Timelines:** `gantt` for plan progress over time
- **Status breakdowns:** `pie` for distribution of plan statuses
- **Flow diagrams:** `flowchart` for data flow or process visualization
- **Git history:** `gitgraph` for branch/merge visualization

Wrap mermaid in: `<pre class="mermaid">graph TD; A-->B;</pre>`

## Screenshots & Visual Evidence

Screenshots can come from two sources:
1. **Pre-captured by the reviewer agent** — saved to `.reports/screenshots/` during live verification
2. **Captured by you (the reporter)** — using `agent-browser` when building reports that need live visuals

### Embedding Screenshots

Embed screenshots as base64 data URIs so reports stay fully self-contained:

```bash
# Convert screenshot to base64 for embedding
base64 -i .reports/screenshots/review-feature-home.png
```

Then embed in HTML:
```html
<div class="screenshot-grid">
  <div class="screenshot">
    <img src="data:image/png;base64,{{BASE64_DATA}}" alt="Home page after changes">
    <div class="caption">Home page — post-implementation</div>
  </div>
</div>
```

### Capturing Screenshots with agent-browser

When you need live screenshots for a report (dashboards, deployed apps, external tools):

```bash
# Capture a specific page
# Capture a specific page (use dev server URL from AGENTS.md)
agent-browser open <dev-server-url>/dashboard
agent-browser wait --load networkidle
agent-browser screenshot .reports/screenshots/dashboard-current.png

# Full page capture
agent-browser screenshot --full .reports/screenshots/page-full.png

# Capture a specific section (scope to selector)
agent-browser snapshot -s "#metrics-panel"
agent-browser screenshot .reports/screenshots/metrics-panel.png

# Close when done
agent-browser close
```

Always close the browser session when finished capturing.

## Report Types

When asked to generate a report, infer the type from context or ask. Common types:

- **Project Brief:** Overview of architecture, module boundaries, recent activity, open plans
- **Sprint Recap:** What was completed, what's in progress, velocity metrics from plan history
- **Impact Analysis:** Visual dependency graph of a planned change's blast radius
- **Estimation Report:** Planned-vs-actual analysis with calibration insights
- **Architecture Map:** Visual module diagram with responsibilities and dependencies
- **Review Report:** Diff review findings with live verification screenshots, test results, and verdict

### JSON Input Schemas

```json
{
  "projectBrief": {
    "type": "object",
    "properties": {
      "projectName": { "type": "string" },
      "architecture": { "type": "string", "description": "Architecture summary" },
      "modules": { "type": "array", "items": { "type": "string" } },
      "recentActivity": { "type": "array", "items": { "type": "string" } },
      "openPlans": { "type": "array", "items": { "type": "string" } }
    },
    "required": ["projectName"]
  },
  "sprintRecap": {
    "type": "object",
    "properties": {
      "completed": { "type": "array", "items": { "type": "object", "properties": { "plan": { "type": "string" }, "status": { "type": "string" } } } },
      "inProgress": { "type": "array", "items": { "type": "object", "properties": { "plan": { "type": "string" }, "status": { "type": "string" } } } },
      "velocity": { "type": "object", "properties": { "planned": { "type": "number" }, "actual": { "type": "number" } } }
    }
  },
  "impactAnalysis": {
    "type": "object",
    "properties": {
      "planName": { "type": "string" },
      "changes": { "type": "array", "items": { "type": "string" } },
      "affectedModules": { "type": "array", "items": { "type": "string" } },
      "dependencies": { "type": "array", "items": { "type": "string" } }
    },
    "required": ["planName"]
  },
  "estimationReport": {
    "type": "object",
    "properties": {
      "planName": { "type": "string" },
      "estimated": { "type": "number", "description": "Estimated hours" },
      "actual": { "type": "number", "description": "Actual hours" },
      "deviations": { "type": "array", "items": { "type": "string" } },
      "calibrationNotes": { "type": "array", "items": { "type": "string" } }
    },
    "required": ["planName"]
  },
  "architectureMap": {
    "type": "object",
    "properties": {
      "modules": { "type": "array", "items": { "type": "object", "properties": { "name": { "type": "string" }, "responsibility": { "type": "string" }, "dependencies": { "type": "array", "items": { "type": "string" } } } } }
    }
  },
  "reviewReport": {
    "type": "object",
    "properties": {
      "planName": { "type": "string" },
      "verdict": { "type": "string", "enum": ["PASS", "NEEDS WORK"] },
      "issues": { "type": "array", "items": { "type": "object", "properties": { "file": { "type": "string" }, "line": { "type": "number" }, "description": { "type": "string" } } } },
      "liveVerification": { "type": "object", "properties": { "checked": { "type": "array", "items": { "type": "string" } }, "passed": { "type": "array", "items": { "type": "string" } }, "failed": { "type": "array", "items": { "type": "string" } } } },
      "screenshots": { "type": "array", "items": { "type": "string" } },
      "suggestions": { "type": "array", "items": { "type": "string" } }
    },
    "required": ["planName", "verdict"]
  }
}

## Constraints

- Reports must be fully self-contained (inline CSS, base64 images, no external dependencies except mermaid CDN).
- Embed screenshots as base64 data URIs — never use relative file paths for images.
- Use the CSS variables from the template for all colors — no hardcoded colors.
- Support both light and dark mode via the `prefers-color-scheme` media query.
- Keep reports scannable: use stat cards for key numbers, tables for details, diagrams for relationships.
- Never touch source code. Only read project files, capture screenshots, and write to `.reports/`.
- Always close agent-browser sessions after capturing screenshots.
```

---

## 5. Optional: Shell Aliases

Add to your `~/.bashrc`, `~/.zshrc`, or shell config for shortcuts outside the Kilo CLI:

```bash
# Start a Kilo session and immediately brief
alias kbrief='kilocode --mode architect -m "Run /brief"'

# Quick check on what plans are open
alias plans='ls -la .plans/ && echo "---" && grep -l "IN_PROGRESS\|IN_REVIEW" .plans/*.md 2>/dev/null || echo "No active plans"'

# Show current branch status vs main
alias branchstat='echo "Branch: $(git branch --show-current)" && echo "Commits ahead of main:" && git log main..HEAD --oneline'
```

---

## 6. Verification Checklist

After setup, confirm everything works:

- [ ] `AGENTS.md` exists at project root with BSER workflow section
- [ ] `.plans/` directory exists with `backlog.md` and `_TEMPLATE.md`
- [ ] `.kilo/agents/` directory exists with `reviewer.md`, `syncer.md`, and `reporter.md`
- [ ] `.reports/` directory exists with `screenshots/` subdirectory
- [ ] `ARCHITECTURE.md` exists (even if minimal)
- [ ] `CONVENTIONS.md` exists (even if minimal)
- [ ] `agent-browser` is installed (`agent-browser --help`)
- [ ] `/brief` command works in Kilo CLI (type `/brief` in interactive mode)
- [ ] `/scope` command works (test: `/scope test-setup`)
- [ ] `/implement` command works
- [ ] `/review` command works
- [ ] `/sync` command works
- [ ] `/hotfix` command works
- [ ] `/recap` command works
- [ ] `/impact` command works
- [ ] `/estimate` command works
- [ ] Reviewer agent is invocable (`@reviewer` in the CLI)
- [ ] Syncer agent is invocable (`@syncer` in the CLI)
- [ ] Reporter agent is invocable (`@reporter` in the CLI)
- [ ] Delete the test branch: `git checkout main && git branch -D feat/test-setup`

---

## 7. Migrating from a Previous BSER Version

If you have a repo already using an earlier version of the BSER framework, follow these steps to bring it up to date. Run these from the project root.

### Step 0: Update `.bser-version`

If you don't have a `.bser-version` file yet, create it:

```bash
touch .bser-version
```

Then update it with the new version:

```text
version: <new-version>
date-adopted: <YYYY-MM-DD>
notes: Migrated from <old-version> — see migration guide in setup.md
```

### Step 2: Scaffold Missing Directories

```bash
# These are idempotent — won't overwrite existing content
mkdir -p .kilo/agents
mkdir -p .reports/screenshots
mkdir -p .kilocode/commands
```

### Step 3: Remove Old Custom Modes

If you previously used `.kilocodemodes` or `custom_modes.yaml` for BSER modes (`reviewer`, `syncer`), these are now subagents:

```bash
# Check if old mode config exists
cat .kilocodemodes 2>/dev/null
cat ~/.config/kilo/custom_modes.yaml 2>/dev/null
```

If you find `reviewer` or `syncer` mode definitions in either file, remove them. They're replaced by the subagent markdown files in `.kilo/agents/`. If the files contain other non-BSER modes you still use, keep those — just remove the BSER-specific ones.

### Step 4: Install Subagents

Copy the subagent definitions from section 4 of this guide into `.kilo/agents/`:

- `.kilo/agents/reviewer.md` — the diff review agent (now with agent-browser live verification)
- `.kilo/agents/syncer.md` — the post-merge doc sync agent
- `.kilo/agents/reporter.md` — the HTML report generator (new — this is the human interface layer)

If you already have these files from a previous version, **replace them entirely** with the current versions from this guide. The reporter agent is new and must be added.

### Step 5: Update Slash Commands

Replace all command files in `.kilocode/commands/` with the current versions from section 3 of this guide. The key changes since earlier versions:

| Command | What Changed |
|---------|-------------|
| `/scope` | No longer takes a kebab-case argument. Accepts freeform description, derives slug. Uses `$ARGUMENTS` instead of `$1`. Now epic-branch-aware. |
| `/epic` | **New.** Freeform description input, creates `epic/` branch. |
| `/implement` | Now checks for `.fixlist.md` to enter targeted fix mode after review. |
| `/review` | Now writes a `.fixlist.md` on NEEDS WORK verdict. Renderer spec for `@reporter` with screenshot embedding. |
| `/sync` | Unchanged. |
| `/hotfix` | Now accepts freeform description, derives slug. |
| `/brief` | Now invokes `@reporter` — output is an HTML report, not terminal text. |
| `/recap` | **New.** End-of-session summary via `@reporter`. |
| `/impact` | **New.** Dependency impact analysis via `@reporter`. |
| `/estimate` | **New.** Planned-vs-actual calibration via `@reporter`. |

The safest approach is to delete all existing command files and re-create them from this guide:

```bash
rm .kilocode/commands/*.md
# Then create each file from section 3
```

### Step 6: Update Plan Templates

If you have `.plans/_TEMPLATE.md`, update it to include the new `Parent` field:

```markdown
**Status:** PLANNING | IN_PROGRESS | IN_REVIEW | COMPLETE
**Branch:** feat/<short-name>
**Parent:** main | epic/<epic-name>
**Created:** <date>
**One-liner:** <single sentence description>
```

Add the epic template if it doesn't exist:

```bash
# Check if epic template exists
ls .plans/_EPIC_TEMPLATE.md 2>/dev/null
```

If missing, create it from section 2 of this guide.

### Step 7: Update AGENTS.md

If your `AGENTS.md` exists but predates the current version, update the `## BSER Workflow` section to match the template in section 1 of this guide. Key additions:

- Reference to `@reporter` and `@syncer` subagents
- The workflow rules section (TDD, no scope expansion, stop-if-plan-is-wrong)
- Git conventions including `epic/` branch prefix

If `AGENTS.md` doesn't exist, create it from the template in section 1.

### Step 8: Update Existing Plan Docs

Existing `.plans/*.md` files from prior versions will still work — the commands read them by slug. But for consistency, you can optionally update completed plans:

```bash
# Find plans missing the Parent field
grep -rL "Parent:" .plans/*.md 2>/dev/null
```

For any active (IN_PROGRESS) plans, add `**Parent:** main` to the frontmatter so the review command knows the merge target. Completed plans don't need updating.

### Step 9: Install agent-browser

If not already installed:

```bash
npx skills add https://github.com/vercel-labs/agent-browser --skill agent-browser
```

This is used by the `reviewer` agent for live UI verification and the `reporter` agent for capturing screenshots.

### Step 10: Verify

Run through the verification checklist in section 6. Pay special attention to:

- [ ] `@reporter` is invocable and generates HTML reports to `.reports/`
- [ ] `/scope` accepts freeform descriptions (test: `/scope Add a health check endpoint to the API`)
- [ ] `/brief` produces an HTML report that auto-opens in the browser
- [ ] Old custom modes are removed and don't conflict with new subagents

### Migration Cleanup Checklist

```bash
# Remove artifacts from older BSER versions that are no longer used
rm -f .kilocodemodes                     # if it only contained BSER modes
rm -f .plans/*.fixlist.md                # clean up any leftover fixlists from interrupted reviews

# Verify no stale mode references in config
grep -r "reviewer\|syncer" ~/.config/kilo/custom_modes.yaml 2>/dev/null && echo "⚠️  Remove BSER modes from global custom_modes.yaml"

# Verify subagents are in place
ls .kilo/agents/reviewer.md .kilo/agents/syncer.md .kilo/agents/reporter.md
```

---

### Create the Quick Reference File

After completing setup, create `BSER-QUICKREF.md` with the following content:

```markdown
# BSER Quick Reference

**Brief → Scope → Execute → Review+Sync**

A structured methodology for human-in-the-loop agentic development with CLI coding agents.

---

## The Loop

```
BRIEF ──► SCOPE ──► EXECUTE ──► REVIEW+SYNC
   ▲                           │
   └───────────────────────────┘
```

| Phase | Duration | Your Role |
|-------|----------|-----------|
| **Brief** | ≤5 min | Read |
| **Scope** | ≤15 min | Decide |
| **Execute** | bulk of time | Steer |
| **Review+Sync** | 10-15 min | Approve |

Phases 1 and 4 are **agent-driven**. Phases 2 and 3 are **where you focus**.

---

## Slash Commands

| Command | Description |
|---------|-------------|
| `/brief` | Generate project briefing (≤5 min) |
| `/scope <description>` | Derive slug, create branch + plan |
| `/epic <description>` | Decompose large task into ordered phases |
| `/implement <slug>` | Continue building from plan |
| `/review <slug>` | Diff review against parent branch |
| `/sync <slug>` | Update docs post-merge |
| `/hotfix <description>` | Quick fix escape hatch |
| `/recap` | End-of-session summary |
| `/impact <slug>` | Dependency impact analysis |
| `/estimate` | Planned vs actual calibration |

### Common Usage

```bash
/brief                                    # Start of session - rebuild mental model
/scope Add XER file parsing support       # One sentence or it's too big
/epic Refactor authentication system     # Too big? Decompose into phases
/implement add-xer-parser                 # Continue from plan
/review add-xer-parser                    # Check against plan
# merge feat/add-xer-parser to main
/sync add-xer-parser                     # Update docs after merge
/recap                                   # End of session
```

---

## Subagents

| Agent | Invocation | Purpose |
|-------|-----------|---------|
| `@reviewer` | `@reviewer Review the diff...` | Read-only diff review, locked permissions |
| `@syncer` | `@syncer Sync docs after merge...` | Post-merge doc updates, markdown-only edits |
| `@reporter` | `@reporter Generate sprint recap...` | HTML+CSS+mermaid report generation |

---

## Rules

1. **One sentence or it's too big.** If you can't describe it in one sentence, use `/epic`.
2. **Plans are the contract.** Implement against the plan. Discovered something new? Add it to "Future" section and continue.
3. **Brief before every epic phase.** Your mental model goes stale fast during large refactors.
4. **Sync after every merge.** This is what keeps `/brief` accurate for your next session.
5. **Don't skip recap.** It takes 30 seconds and preserves session context.

---

## Escape Hatches

| Situation | Command |
|-----------|---------|
| Quick fix (<15 min) | `/hotfix <description>` |
| Exploration/spike | `git checkout -b spike/<name>` |
| Tiny fix (<5 min) | Do on main, `/sync` after |

**Never merge a spike branch.** Extract learnings into a `/scope` task.
```

---

## Updating the Framework

This setup is meant to evolve. Common adjustments:

- **Tweak command prompts** as you learn what works for your projects. The prompts in `.kilocode/commands/` are just markdown — edit freely.
- **Add project-specific commands** in `.kilocode/commands/` that override the global ones when a project needs different behavior (e.g., Apollo's `/brief` might include service health checks).
- **Add subagents** for other workflows you formalize (e.g., a `spike-explorer` agent with read-only permissions for codebase exploration before scoping).
- **Tune subagent permissions** — the `reviewer` and `syncer` permission blocks can be adjusted per project (e.g., allow `cargo test` instead of `npm test`).
- **Adjust the plan template** based on what information you actually reference during implementation.

Once setup is complete, you're ready to use BSER. The quick reference above summarizes the workflow — see **BSER-workflow.md** for detailed daily usage.