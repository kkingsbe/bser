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

## Context & Learnings

> Updated as discoveries are made throughout the epic lifecycle. This is where you capture what you learn about the system, what assumptions you've made, key realizations, and open questions that may impact remaining phases.

### Discoveries
How the system actually works, built up across phases.

- **<date> (<phase-n>)**: <description of what you learned>

### Assumptions
Things you expect to be true but haven't validated yet.

- **<date> (<phase-n>)**: <assumption description>
  - **Status:** Unvalidated | Confirmed | Challenged | Invalidated

### Realizations
Key insights that changed your approach or understanding.

- **<date> (<phase-n>)**: <realization>

### Resolved Questions
Questions that were blocking decisions but are now answered.

- **<date> (<phase-n>)**: **Q:** <question>
  **A:** <answer>

### Open Questions
Still unresolved — may impact remaining phases.

- **<date> (<phase-n>)**: <question>

---

## Progress Log

| Phase | Started | Completed | Notes |
|-------|---------|-----------|-------|
| 1 | | | |
| 2 | | | |
| 3 | | | |
```