# Bugfix Workflow Example: Race Condition in File Upload Handler

This example demonstrates the **bugfix workflow** using `/hotfix`, which skips the planning phase for rapid fixes while still requiring Review+Sync. It shows how `/hotfix` differs from normal `/scope` + `/implement`.

**What this example teaches:**
- When to use `/hotfix` vs `/scope` for bugfixes
- How `/hotfix` creates a minimal `fix/` branch without a plan
- How to verify the fix with tests
- How `/review` and `/sync` still apply to hotfixes

**Key difference from normal workflow:**
- `/scope` creates a plan doc, commits it, allows scope discussion
- `/hotfix` goes directly to implementation on a `fix/` branch — no plan, no planning phase

---

## The Scenario

Your monitoring alerts fire: file uploads are occasionally corrupting. After investigation, you find a **race condition** in `src/uploads/handler.ts`. When multiple users upload files simultaneously with the same filename, the final state depends on which upload completes last — instead of being deterministic.

The bug is:
- **Reproducible:** Yes (under concurrent load)
- **Scope:** Small — single file, focused change
- **Risk:** High (data corruption)
- **Planning needed:** Minimal (root cause is clear)

This is a perfect candidate for `/hotfix`.

---

## Phase 1: Brief (Optional but Recommended)

Even for a hotfix, `/brief` helps if you've been away from the project.

### Command

```
/brief
```

### Expected Output

Brief report showing current state. You see you're on `main` with clean state. The bug needs fixing ASAP.

### What You Do

Acknowledge the issue. Move directly to `/hotfix`.

---

## Phase 2: Hotfix (Execute Immediately)

**Purpose:** Fix the bug directly without planning overhead. The `/hotfix` command creates a `fix/` branch and implements immediately.

### Command

```
/hotfix Fix race condition in file upload handler when concurrent uploads have same filename
```

### Expected Output

```
[1] Deriving slug
  slug: fix-upload-race-condition

[2] Creating branch
  git checkout main && git pull
  git checkout -b fix/fix-upload-race-condition
  ✓ Branch created: fix/fix-upload-race-condition

[3] Implementing fix (minimal changes)
  
  Reading src/uploads/handler.ts...
  
  Root cause identified:
  - uploads stored at: uploads/{userId}/{filename}
  - no locking mechanism
  - concurrent writes with same filename cause last-write-wins
  
  Fix approach:
  - Add UUID prefix to filename: uploads/{userId}/{uuid}-{filename}
  - This makes each upload unique regardless of original filename
  
  Changes:
  - src/uploads/handler.ts:21-34 — added uuid generation and prefix logic
  - src/uploads/handler.ts:40 — updated storage path construction

[4] Running tests
  npm test 2>&1 | tail -10
  Result: 143 passed, 0 failed
  
  Running specific upload tests:
  npm test -- --testPathPattern="upload" 2>&1
  Result: 12 passed, 0 failed

[5] Commit
  git add -A && git commit -m "fix: prevent race condition in upload handler with UUID prefix"
  ✓ Committed: fix: prevent race condition in upload handler with UUID prefix

Done. Run /review and /sync manually after merging.
```

### What You Do

Verify the fix makes sense. The output shows:
- Slug derived from the issue description
- Minimal branch: `fix/fix-upload-race-condition`
- Root cause identified inline (helpful for future reference)
- Fix applied directly — no separate plan doc created
- Tests passing

**Critical difference from `/scope`:** The `/hotfix` command does NOT create a plan document in `.plans/`. It goes straight to implementation.

---

## Merge (Before Review)

### Command

```bash
git checkout main
git pull
git merge fix/fix-upload-race-condition
git branch -d fix/fix-upload-race-condition
```

### Expected Output

```
Already on 'main'
Your branch is up to date with 'origin/main'.
Merge made by 'ort' strategy.
 1 file changed, +8, -4
 Deleted branch fix/fix-upload-race-condition (was a3f2b1c).
```

---

## Phase 3: Review

**Purpose:** Even hotfixes need review. Verify the fix is correct and doesn't introduce new issues.

### Command

```
/review fix-upload-race-condition
```

### Expected Output

The review command expects a plan document. Since hotfixes don't create plans, you need to specify what to review:

```
/review fix-upload-race-condition
```

The agent should handle this gracefully — it will:
1. Check if `.plans/fix-upload-race-condition.md` exists
2. If not, review the commit diff directly against main
3. Generate a review report

```html
<!-- Simplified report for hotfix -->
<div class="stat-grid">
  <div class="stat-card">
    <div class="value">1</div>
    <div class="label">Files Changed</div>
  </div>
  <div class="stat-card">
    <div class="value">+8 / -4</div>
    <div class="label">Lines</div>
  </div>
  <div class="stat-card">
    <div class="value">143/143</div>
    <div class="label">Tests Passing</div>
  </div>
  <div class="stat-card">
    <div class="value">PASS</div>
    <div class="label">Verdict</div>
  </div>
</div>

<h2>Review Findings</h2>
<table>
  <tr><th>File</th><th>Change</th><th>Assessment</th></tr>
  <tr><td>src/uploads/handler.ts</td><td>+8 / -4</td><td><span class="tag tag-green">appropriate</span></td></tr>
</table>

<h2>Fix Analysis</h2>
<p><strong>Root cause:</strong> Concurrent uploads with same filename caused last-write-wins behavior</p>
<p><strong>Fix applied:</strong> UUID prefix on filename prevents collision</p>
<p><strong>Scope:</strong> Minimal — only changed what was necessary</p>
<p><strong>Risk:</strong> Low — UUID generation is deterministic, no external dependencies added</p>

<h2>Verdict</h2>
<p><span class="tag tag-green">PASS</span> — Hotfix is appropriate for this issue. Minimal scope, clear root cause, tests pass.</p>
```

### What You Do

Read the report. Since it's PASS, move to Sync.

---

## Phase 4: Sync

**Purpose:** Document the fix for future reference. Even though there's no plan doc, the sync should note the fix in the relevant docs.

### Command

```
/sync fix-upload-race-condition
```

### Expected Output

```
[SYNC] fix-upload-race-condition (merged to main)

Note: No plan document found (hotfix). Creating post-fix record...

[1] ARCHITECTURE.md
  ✓ No structural changes

[2] CONVENTIONS.md
  Added note to "File Handling" section:
  - "Upload handlers must use UUID prefixes to prevent race conditions under concurrent access"
  - Or: "All file storage paths must include a uniqueness mechanism (e.g., UUID)"

[3] Bug Log (if exists: .plans/bugs.md)
  Creating: .plans/bugs.md
  Entry:
  ## fix-upload-race-condition
  **Fixed:** 2026-03-31
  **Symptom:** Race condition in src/uploads/handler.ts under concurrent uploads
  **Root cause:** No uniqueness mechanism on filename
  **Fix:** Added UUID prefix to storage path
  **Reviewed:** 2026-03-31 (PASS)

[4] Commit
  ✓ docs: sync after fix-upload-race-condition merge
```

### Note on Hotfix Sync

Hotfixes don't have plan documents, so `/sync` adapts:
- If a bug log exists, append to it
- If not, create a standard bug log at `.plans/bugs.md`
- Update CONVENTIONS.md with the lesson learned
- The commit message still follows the pattern: `docs: sync after <slug> merge`

---

## Summary: Hotfix vs Normal Workflow

| Aspect | `/scope` + `/implement` | `/hotfix` |
|--------|-------------------------|-----------|
| **When to use** | Features, refactors, planned changes | Urgent bug fixes with clear root cause |
| **Branch name** | `feat/<slug>` or `fix/<slug>` | `fix/<slug>` |
| **Plan document** | Yes — `.plans/<slug>.md` | No |
| **Planning phase** | Yes — scope discussion with user | No — direct implementation |
| **TDD approach** | Recommended | Recommended |
| **Review** | Required | Required |
| **Sync** | Required | Required (but adapts to no-plan scenario) |
| **Time investment** | 15-30 min for scoping | ~5 min to fix |

### When to Choose Hotfix

Use `/hotfix` when ALL of these are true:
- The bug has a clear, identifiable root cause
- The fix is small and focused (1-3 files)
- No new functionality is being added
- No architectural decisions need discussion
- You don't need to create a test baseline (the existing tests cover the regression)

Use `/scope` when ANY of these are true:
- The root cause is unclear and needs investigation
- The fix might touch multiple modules
- There's architectural discussion needed
- You need to defer some items to "Future"
- The fix is part of a larger change

---

## Templates Referenced

- `setup/commands/hotfix.md` — Hotfix command definition
- `setup/commands/review.md` — Review command (works for hotfixes too)
- `setup/commands/sync.md` — Sync command (adapts to no-plan scenario)
- `setup/agents/reviewer.md` — Read-only reviewer agent
- `setup/agents/reporter.md` — Report generation

## Key Takeaways

1. **Hotfix is for small, clear bugs.** Don't use it as an excuse to skip thinking.
2. **Review still applies.** The hotfix branch gets reviewed just like a feature branch.
3. **Sync adapts.** Without a plan doc, sync creates a bug log entry instead.
4. **The commit is the record.** Hotfix commits should be descriptive: `fix: <slug>` with clear description.
5. **Consider converting to scope later.** If a hotfix reveals a larger pattern, create a plan for the bigger fix.
