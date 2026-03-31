# Feature Workflow Example: User Profile Avatar Upload

This example demonstrates the complete BSER loop for a typical feature: **adding user profile avatar upload functionality**. It walks through each phase—Brief → Scope → Execute → Review+Sync—showing actual commands, expected outputs, and how the plan template (`setup/templates/plan-template.md`) structures the work.

**What this example teaches:**
- How `/brief` rebuilds context before starting
- How `/scope` derives a slug, creates a branch, and writes a plan
- How `/implement` executes with TDD and commits
- How `/review` evaluates the diff against the plan
- How `/sync` updates documentation after merge

---

## Phase 1: Brief

**Purpose:** Rebuild your mental model before starting. Read the briefing report, don't plan yet.

### Command

```
/brief
```

### Expected Output

The agent generates a briefing report at `.reports/brief-2026-03-31.html` and opens it in your browser. The report shows:

- **Stat cards:** 3 commits in last 48h, 2 open plans, 1 active epic, current branch, 0 uncommitted files
- **Recent Changes:** A table showing recent commits (e.g., "add user profile model", "update database migrations")
- **Current State:** Branch `main` is clean, plans `add-user-authentication` (COMPLETE), `fix-nav-menu-zindex` (IN_PROGRESS)
- **Epic Progress:** `refactor-payments-phase-2` — 2/3 phases complete, current phase is `refactor-payments-phase-2-api`
- **Suggested Next Actions:** 
  1. Continue `fix-nav-menu-zindex` implementation
  2. Start scoping avatar upload feature (you just returned to this project after a week)

### What You Do

Read the report. Notice you're on `main` with a clean state. The `fix-nav-menu-zindex` is nearly done but you want to start something fresh. Pick avatar upload.

---

## Phase 2: Scope

**Purpose:** Derive a slug, create the branch, write the plan. This is a 15-minute focused task.

### Command

```
/scope Add user profile avatar upload with S3 storage and thumbnail generation
```

### Expected Output

```
[0] Naming
  Derived slug: add-user-avatar-upload
  One-liner: Add user profile avatar upload with S3 storage and thumbnail generation

[1] Branch
  From: main (clean)
  Creating: feat/add-user-avatar-upload
  ✓ Branch created

[2] Test Baseline
  Running: npm test
  Result: 142 passed, 0 failed
  Saved to: .plans/add-user-avatar-upload.baseline-tests.txt
  ✓ Baseline committed

[3] Plan Location
  Saving to: .plans/add-user-avatar-upload.md

[4] Plan Created

# Plan: add-user-avatar-upload

**Status:** IN_PROGRESS
**Category:** feature
**Branch:** feat/add-user-avatar-upload
**Parent:** main
**Created:** 2026-03-31
**One-liner:** Add user profile avatar upload with S3 storage and thumbnail generation

## Scope

Allow users to upload a profile avatar image (JPEG, PNG, WebP up to 5MB). The image is stored in S3, a 150x150 thumbnail is generated, and both URLs are stored on the user profile.

## Files to Change

- `src/users/avatar.service.ts` — Upload logic, S3 client, thumbnail generation
- `src/users/avatar.controller.ts` — HTTP endpoint for upload
- `src/users/avatar.spec.ts` — Unit tests for upload service
- `src/users/avatar.controller.spec.ts` — Integration tests for endpoint
- `src/users/user.model.ts` — Add avatarUrl and thumbnailUrl fields
- `src/config/s3.ts` — S3 configuration (uses existing AWS SDK)

## Test Cases

- [ ] User can upload a valid JPEG/PNG/WebP image
- [ ] Upload rejects files larger than 5MB
- [ ] Upload rejects non-image files
- [ ] Thumbnail is generated at 150x150
- [ ] Both URLs are saved to user profile
- [ ] Upload fails gracefully when S3 is unavailable

## Implementation Notes

- Use `sharp` library for thumbnail generation (already in package.json)
- Reuse existing S3 client pattern from `src/config/s3.ts`
- Follow existing upload endpoint pattern from `src/uploads/`
- Store original and thumbnail keys with `avatars/{userId}/original.{ext}` and `avatars/{userId}/thumb.{ext}`

## Future (Out of Scope)

- Avatar cropping UI (defer to follow-up)
- Auto-delete old avatar when new one is uploaded
- Default avatar for users without photos
- Avatar upload progress indicator

---

[5] Commit
  ✓ plan: add-user-avatar-upload committed
  ✓ Branch ready at feat/add-user-avatar-upload
```

### What You Do

Review the plan:
- The slug `add-user-avatar-upload` looks right
- The 6 test cases cover the core behavior
- Files to change are focused on avatar functionality
- The "Future" section correctly defers cropping, deletion, defaults

Approve or request adjustments. When satisfied, move to Execute.

---

## Phase 3: Execute (First Session)

**Purpose:** Build the feature with TDD. The agent reads the plan, writes tests first, then implements.

### Command

```
/implement add-user-avatar-upload
```

### Expected Output

```
[IN_PROGRESS] add-user-avatar-upload

[1] Reading plan from .plans/add-user-avatar-upload.md
[2] Status: IN_PROGRESS
[3] Running tests on feat/add-user-avatar-upload...
  npm test 2>&1 | tail -20
  Result: 142 passed, 0 failed

[4] Starting TDD cycle...

--- Test Case 1: User can upload a valid JPEG/PNG/WebP image ---
[WRITE] src/users/avatar.spec.ts
  - Adding test for successful upload with valid image

[IMPLEMENT] src/users/avatar.service.ts
  - Creating AvatarService with uploadAvatar(userId, file) method
  - Using sharp to process image
  - Uploading to S3 with proper key pattern

[COMMIT] add avatar service with S3 upload and thumbnail generation
  143 passed, 0 failed

--- Test Case 2: Upload rejects files larger than 5MB ---
[WRITE] src/users/avatar.spec.ts
  - Adding test for size validation

[IMPLEMENT] src/users/avatar.service.ts
  - Adding size check before processing

[COMMIT] add file size validation to avatar upload
  144 passed, 0 failed

--- Test Case 3: Upload rejects non-image files ---
[WRITE] src/users/avatar.spec.ts
  - Adding test for MIME type validation

[IMPLEMENT] src/users/avatar.service.ts
  - Adding MIME type check against allowed list ['image/jpeg', 'image/png', 'image/webp']

[COMMIT] add MIME type validation to avatar upload
  145 passed, 0 failed

--- Test Cases 4-6: Thumbnail generation, URL persistence, error handling ---
[WRITE + IMPLEMENT] Remaining test cases...
[COMMIT] add thumbnail generation and error handling
  148 passed, 0 failed

--- Controller layer ---
[WRITE] src/users/avatar.controller.spec.ts
[IMPLEMENT] src/users/avatar.controller.ts
[COMMIT] add avatar upload controller endpoint

--- Model update ---
[IMPLEMENT] src/users/user.model.ts
  - Adding avatarUrl?: string
  - Adding thumbnailUrl?: string

[COMMIT] add avatar fields to user model
  150 passed, 0 failed

[5] Updating plan checkboxes...
  ✓ 6/6 test cases passing

[6] Running full test suite...
  Result: 150 passed, 0 failed
  All planned test cases passing

Current progress: 6/6 test cases, 4/4 files changed
```

### What You Do

Monitor the agent's progress. The agent commits after each logical unit. You can verify:

```bash
git log --oneline -6
```

Output:
```
a3f2b1c add avatar upload controller endpoint
9d8e7f1 add thumbnail generation and error handling
5c4b3a2 add MIME type validation to avatar upload
2f1e9d3 add file size validation to avatar upload
8a7b6c4 add avatar service with S3 upload and thumbnail generation
f9e8d2a test baseline: add-user-avatar-upload
```

The implementation is complete. Move to Review.

---

## Phase 4a: Review

**Purpose:** Evaluate the diff against the plan. Verify scope alignment, test coverage, code quality.

### Command

```
/review add-user-avatar-upload
```

### Expected Output

The agent generates a review report at `.reports/review-add-user-avatar-upload-2026-03-31.html`:

```html
<!-- Stat cards -->
<div class="stat-grid">
  <div class="stat-card">
    <div class="value">5</div>
    <div class="label">Files Changed</div>
  </div>
  <div class="stat-card">
    <div class="value">+186 / -12</div>
    <div class="label">Lines</div>
  </div>
  <div class="stat-card">
    <div class="value">6/6</div>
    <div class="label">Tests Passing</div>
  </div>
  <div class="stat-card">
    <div class="value">PASS</div>
    <div class="label">Verdict</div>
  </div>
</div>

<!-- Diff Summary -->
<table>
  <tr><th>File</th><th>Lines</th><th>Planned</th></tr>
  <tr><td>src/users/avatar.service.ts</td><td>+89</td><td><span class="tag tag-green">planned</span></td></tr>
  <tr><td>src/users/avatar.controller.ts</td><td>+42</td><td><span class="tag tag-green">planned</span></td></tr>
  <tr><td>src/users/avatar.spec.ts</td><td>+38</td><td><span class="tag tag-green">planned</span></td></tr>
  <tr><td>src/users/avatar.controller.spec.ts</td><td>+24</td><td><span class="tag tag-green">planned</span></td></tr>
  <tr><td>src/users/user.model.ts</td><td>+3 / -2</td><td><span class="tag tag-green">planned</span></td></tr>
</table>

<!-- Review Findings -->
<h2>Review Findings</h2>
<p>No issues found. Implementation matches plan.</p>

<!-- Verdict -->
<h2>Verdict</h2>
<p><span class="tag tag-green">PASS</span> — All test cases passing, scope matches plan, no deviations.</p>
```

### What You Do

Read the report. If **PASS**:
1. Merge the branch
2. Run `/sync add-user-avatar-upload`

If **NEEDS WORK**: The agent writes a `.plans/add-user-avatar-upload.fixlist.md` with specific issues. Run `/implement add-user-avatar-upload` again to address them.

---

## Merge

### Commands

```bash
git checkout main
git pull
git merge feat/add-user-avatar-upload
git branch -d feat/add-user-avatar-upload
```

### Expected Output

```
Already on 'main'
Your branch is up to date with 'origin/main'.
Merge made by 'ort' strategy.
 5 files changed, +186, -12
 Deleted branch feat/add-user-avatar-upload (was a3f2b1c).
```

---

## Phase 4b: Sync

**Purpose:** Update documentation after merge. Keep ARCHITECTURE.md and CONVENTIONS.md accurate.

### Command

```
/sync add-user-avatar-upload
```

### Expected Output

```
[SYNC] add-user-avatar-upload (merged to main)

[1] ARCHITECTURE.md
  ✓ No structural changes requiring updates

[2] CONVENTIONS.md
  Added new convention:
  - "Avatar uploads use sharp for processing, stored in S3 under avatars/{userId}/"

[3] .plans/add-user-avatar-upload.md
  Status: COMPLETE
  Completion Log filled:
  - Completed: 2026-03-31
  - Actual changes: Added avatar upload with S3 storage and 150x150 thumbnail generation
  - Deviated from plan: No
  - Impact: users module now handles avatar storage

[4] .plans/backlog.md
  ✓ No Future items to append

[5] Commit
  ✓ docs: sync after add-user-avatar-upload merge
```

---

## End of Session

### Command

```
/recap
```

### Expected Output

Report at `.reports/recap-2026-03-31.html`:

```html
<!-- Stat cards -->
<div class="stat-grid">
  <div class="stat-card"><div class="value">6</div><div class="label">Commits</div></div>
  <div class="stat-card"><div class="value">5</div><div class="label">Files Changed</div></div>
  <div class="stat-card"><div class="value">150/150</div><div class="label">Tests Passing</div></div>
  <div class="stat-card"><div class="value">0</div><div class="label">Open Plans</div></div>
</div>

<h2>Session Activity</h2>
<table>
  <tr><td>add avatar service with S3 upload and thumbnail generation</td></tr>
  <tr><td>add file size validation to avatar upload</td></tr>
  <tr><td>add MIME type validation to avatar upload</td></tr>
  <tr><td>add thumbnail generation and error handling</td></tr>
  <tr><td>add avatar upload controller endpoint</td></tr>
  <tr><td>docs: sync after add-user-avatar-upload merge</td></tr>
</table>

<h2>Next Session</h2>
<ol>
  <li>Continue with fix-nav-menu-zindex (nearly complete)</li>
  <li>Or pick next item from .plans/backlog.md</li>
</ol>
```

---

## Summary

The BSER loop for this feature:

| Phase | Command | Key Output |
|-------|---------|------------|
| Brief | `/brief` | `.reports/brief-<date>.html` |
| Scope | `/scope Add user profile avatar upload...` | `feat/add-user-avatar-upload` branch + `.plans/add-user-avatar-upload.md` |
| Execute | `/implement add-user-avatar-upload` | 6 commits, all test cases passing |
| Review | `/review add-user-avatar-upload` | `.reports/review-<plan>-<date>.html` with PASS verdict |
| Sync | `/sync add-user-avatar-upload` | Docs updated, plan marked COMPLETE |

**Templates referenced:**
- `setup/templates/plan-template.md` — Structure of the plan document
- `setup/templates/backlog-template.md` — Backlog file structure
- `setup/agents/reporter.md` — HTML report generation
- `setup/commands/brief.md`, `scope.md`, `implement.md`, `review.md`, `sync.md` — Command definitions
