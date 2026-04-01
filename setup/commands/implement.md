# .kilocode/commands/implement.md
---
description: Continue implementing an existing plan
mode: code
arguments:
  - plan_name
---
Continue implementing the plan. Plans are stored at:
- `.plans/<name>.md` for regular tasks
- `.plans/epics/<epic-slug>/<name>.md` for epic phases

The agent should first check `.plans/$1.md`, and if not found, search in `.plans/epics/*/$1.md`.

Process:
1. Read the plan document and check its current status.
2. **Check for a fix list:** If a fixlist exists alongside the plan (e.g., `.plans/$1.fixlist.md` or `.plans/epics/*/$1.fixlist.md`), this is a post-review fix cycle. For fixlist workflow, see `{{partials:fixlist-workflow}}`
3. Check git status and recent commits on this branch to understand what's already been done.
4. Run existing tests to see current state: what passes, what fails, what's missing.
5. Continue implementation from where it left off, following TDD workflow defined in `{{partials:tdd-workflow}}`

   Follow patterns in CONVENTIONS.md. Commit after each logical unit of progress with descriptive commit messages.

5a. **Context extraction:** If this plan is a phase of an epic (Parent: epic/<epic-slug>): For epic context workflow, see `{{partials:epic-context-workflow}}`

6. After completing a chunk of work, update the test case checkboxes in the plan doc (across all three categories: Happy Path, Error & Boundary, Integration).

Do NOT:
- Modify files outside the scope defined in the plan.
- Add features not in the plan (note them in the "Future" section instead).
- Refactor unrelated code, even if you notice opportunities.
- Leave learnings in implementation notes — extract discoveries to the epic's Context & Learnings section before completing

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
- Test cases completed: Happy Path 2/3, Error & Boundary 1/4, Integration 0/2
```

**Common Variations:**
- `/implement add-user-authentication` — Standard implementation
- When fixlist exists: `/implement` reads the fixlist alongside the plan and enters fix mode
```