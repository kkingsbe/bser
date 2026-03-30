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