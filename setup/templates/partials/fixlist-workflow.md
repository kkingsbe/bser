## Fixlist Workflow

A fixlist is used for post-review fix cycles. When `/review` identifies issues that need to be addressed, it creates a fixlist alongside the plan.

### Detecting a Fixlist
When starting `/implement`, check for a fixlist file:
- `.plans/<name>.fixlist.md` for regular tasks
- `.plans/epics/<epic-slug>/<name>.fixlist.md` for epic phases

If a fixlist exists, the implementation enters fix mode.

### Fix Mode Process
1. **Read the fixlist** alongside the plan document
2. **Address each issue** in the fix list, one by one
3. **Check off items** as you fix them (update checkboxes in the fixlist)
4. **Run tests** after each fix to ensure no regressions
5. **Delete the fixlist file** when all issues are resolved

### Fix Mode Rules
- Focus ONLY on the issues in the fixlist
- Do not expand scope or add new features
- Do not refactor unrelated code
- Skip the normal implementation workflow — fix mode is a targeted correction cycle

### Exiting Fix Mode
When the fixlist is deleted, the next `/implement` call will proceed with normal workflow (checking git status, running tests, continuing from where implementation left off).