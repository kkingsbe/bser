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

### Happy Path
- [ ] <expected usage scenario 1>
- [ ] <expected usage scenario 2>

### Error & Boundary
- [ ] <invalid input handling>
- [ ] <boundary condition / edge case>
- [ ] <missing or null data handling>

### Integration
- [ ] <interaction with dependent module or caller>
- [ ] <end-to-end flow through the changed components>

## Acceptance Criteria

<What must be true for this task to be considered done — from the user's perspective, not the test runner's. These describe observable behavior, not implementation details.>

- <criterion 1: e.g., "User sees a confirmation toast after saving">
- <criterion 2: e.g., "API returns 400 with a descriptive message for malformed input">
- <criterion 3: e.g., "Existing import flows continue to work unchanged">

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

## Context

> **For standalone phases:** This document owns its own context. Use the categories below to track discoveries, assumptions, and realizations during this phase.

> **For phases linked to an epic:** Context lives in the epic document, not here. Do NOT add context entries below. Instead, when you complete this phase's work (before marking COMPLETE), extract relevant context to the epic's `## Context & Learnings` section and reference it in your Completion Log.