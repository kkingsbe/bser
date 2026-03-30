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

## Chain of Thought

Follow this reasoning process for every epic decomposition:

1. **UNDERSTAND**: Read ARCHITECTURE.md to map current module boundaries. Read any existing epic plans for context. Identify which parts of the system will be affected.
2. **ASSESS**: Evaluate the task size. Count the distinct functional areas involved. Estimate the number of commits needed. If it feels like it could be done in 1-2 days of focused work, it might not need to be an epic.
3. **DECOMPOSE**: Break the epic into 2-6 phases, where each phase:
   - Can be merged independently into the epic branch without breaking it
   - Leaves the epic branch in a working state
   - Has clear boundaries (obvious what's in vs out)
4. **ORDER**: Arrange phases so dependencies flow forward. List which earlier phases each later phase depends on.
5. **VALIDATE**: Run through the exit criteria — would completing these phases actually deliver the stated objective? Are there any gaps or circular dependencies?
6. **RISK**: Flag any phases that touch shared/core modules, involve migrations, or have uncertain requirements. Note these in the epic's Risks section.

Keep reasoning explicit — show which step you're at as you work through it.

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