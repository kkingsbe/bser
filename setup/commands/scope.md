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

## Chain of Thought

Follow this reasoning process for every scoping task:

1. **UNDERSTAND**: Read ARCHITECTURE.md and CONVENTIONS.md to understand the current system boundaries and patterns. Identify which modules are likely to be affected.
2. **INTERPRET**: Parse the task description. What is the user trying to accomplish? What does success look like? Is this a feature, bugfix, refactor, or chore?
3. **DERIVE**: Convert the description into a short kebab-case slug (2-4 words max). Extract a one-liner that captures the essential goal.
4. **CONSTRAIN**: Identify the boundaries — what is explicitly NOT part of this task. Check the "Future (Out of Scope)" section of any existing related plans.
5. **DECOMPOSE**: Break into concrete deliverables: specific files to change, specific test cases to write, specific behavior to implement. If the list grows beyond 5-7 items, suggest decomposing into an epic instead.
6. **VALIDATE**: Confirm the plan is achievable in a single session. Check that all dependencies are available and that the test baseline will be clean.

Keep reasoning explicit in your output — show which step you're at as you work through it.