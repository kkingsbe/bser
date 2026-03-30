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