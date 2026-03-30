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