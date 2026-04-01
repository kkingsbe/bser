# .kilocode/commands/sync.md
---
description: Reconcile project documentation to match the current state of the codebase
mode: code
---
Sync all project documentation to the current state of the codebase.

This command takes no arguments. It automatically determines what changed since each document was last synced and updates accordingly.

## Process

### 1. DETERMINE DIFF RANGE

Each synced document tracks when it was last updated via an inline `sync-commit` metadata field in its header. For each document, compute the diff since its last sync:

```bash
# Read the last-synced commit from each doc
ARCH_SYNC=$(grep 'sync-commit:' ARCHITECTURE.md | sed 's/.*sync-commit: //' | tr -d '>' | xargs)
CONV_SYNC=$(grep 'sync-commit:' CONVENTIONS.md | sed 's/.*sync-commit: //' | tr -d '>' | xargs)
CURRENT=$(git rev-parse HEAD)
```

If a document has no `sync-commit` field (first sync, or legacy doc), treat it as needing a full reconciliation — read the entire codebase and rewrite the doc from scratch to match current reality.

For each document, the relevant diff is:
```bash
git diff <sync-commit>..HEAD --stat
git diff <sync-commit>..HEAD
```

### 2. DETECT STRUCTURAL CHANGES

Check whether the diff since last sync includes structural changes that warrant reading source files:

```bash
# Count new/deleted/moved files since last architecture sync
git diff $ARCH_SYNC..HEAD --diff-filter=ADRC --name-only
```

**Structural change indicators** (trigger a source-file read):
- New directories created under `src/`, `lib/`, `app/`, or equivalent top-level source directories
- Files deleted or renamed across module boundaries
- New entry points, routers, or service registrations
- Changes to dependency manifests (`package.json`, `Cargo.toml`, `go.mod`, `requirements.txt`, etc.)

If structural changes are detected, read the relevant source files to understand the current module structure. Focus on:
- Directory structure and top-level organization
- Public exports and module interfaces
- Import graphs between modules (which modules depend on which)
- Entry points and bootstrapping

If no structural changes, work from the diff and existing documentation only.

### 3. UPDATE ARCHITECTURE.md

Read ARCHITECTURE.md and the diff since its `sync-commit`.

- If structural changes were detected: read source files as described above, then rewrite the relevant sections of ARCHITECTURE.md to match the **current** state of the codebase — not just what changed in the diff.
- If no structural changes: check the diff for meaningful non-structural updates (new key decisions, changed data flow) and update only those sections.
- If nothing relevant changed: leave it alone — don't add noise.

When updating, replace the metadata line:
```markdown
> Last updated: <today's date> · sync-commit: <current HEAD hash>
```

### 4. UPDATE CONVENTIONS.md

Read CONVENTIONS.md and the diff since its `sync-commit`.

- Look for new patterns established across the diff: error handling approaches, testing patterns, naming conventions, configuration patterns.
- Only add genuinely new conventions — not one-off decisions or single-use patterns.
- If a convention in the doc contradicts current code, update the doc to match reality.

When updating, replace the metadata line:
```markdown
> Last updated: <today's date> · sync-commit: <current HEAD hash>
```

### 5. COMPLETE PLANS

Automatically detect which plans should be marked complete:

```bash
# Find plans that are IN_PROGRESS or IN_REVIEW
grep -rl "Status:.*IN_PROGRESS\|Status:.*IN_REVIEW" .plans/*.md .plans/epics/**/*.md 2>/dev/null
```

For each active plan:
- Check if its branch has been merged to main (or to its parent epic branch): `git branch --merged main | grep <branch-name>`
- If merged:
  - Set Status to COMPLETE
  - Fill in the Completion Log: date, actual changes summary, whether implementation deviated from plan, impact on other modules
  - If the plan's "Future (Out of Scope)" section has items, append them to `.plans/backlog.md` using the format: `- <item> (from: <plan-name>)`
- If this was an epic phase:
  - Update the epic's README.md phases table: set the phase status to COMPLETE and fill in the "Completed" date

### 6. UPDATE COMMANDS REGISTRY

Check if any commands in `.kilocode/commands/` were added, modified, or removed since the last sync:

```bash
git diff $ARCH_SYNC..HEAD --name-only -- .kilocode/commands/
```

If changes exist, update `.kilocode/commands/commands.md` to match.

### 7. COMMIT

```bash
git add -A && git commit -m "docs: sync to $(git rev-parse --short HEAD)"
```

Be concise in all updates. These docs should be scannable, not exhaustive.

## Examples

**Invocation:**
```
/sync
```

**Expected Output:**
Terminal output showing:
- Diff range for each doc (e.g., "ARCHITECTURE.md last synced at abc1234, 14 commits behind")
- Whether structural changes were detected (and if source files were read)
- Which documents were updated
- Which plans were auto-completed
- Any items added to backlog
- Commit confirmation

**Example — Multiple merges since last sync:**
```
[SYNC] ARCHITECTURE.md last synced at abc1234 (14 commits behind)
[SYNC] CONVENTIONS.md last synced at def5678 (8 commits behind)
[SYNC] Structural changes detected: new src/notifications/ directory, updated package.json
[SYNC] Reading source files for current module structure...
[SYNC] Updated ARCHITECTURE.md — added notifications module, updated dependency graph
[SYNC] Updated CONVENTIONS.md — added WebSocket event naming convention
[SYNC] Completed plan: add-notification-service (branch merged to main)
[SYNC] Completed plan: fix-auth-token-refresh (branch merged to main)
[SYNC] Moved 2 backlog items from plan future sections
[SYNC] Committed: docs: sync to a1b2c3d
```

**Example — Nothing meaningful changed:**
```
[SYNC] ARCHITECTURE.md last synced at abc1234 (2 commits behind)
[SYNC] CONVENTIONS.md last synced at abc1234 (2 commits behind)
[SYNC] No structural changes detected
[SYNC] No documentation updates needed
[SYNC] Completed plan: fix-csv-export-crash (branch merged to main)
[SYNC] Committed: docs: sync to a1b2c3d
```

**Example — First sync (no sync-commit metadata yet):**
```
[SYNC] ARCHITECTURE.md has no sync-commit — performing full reconciliation
[SYNC] CONVENTIONS.md has no sync-commit — performing full reconciliation
[SYNC] Reading source files for current module structure...
[SYNC] Rewrote ARCHITECTURE.md to match current codebase
[SYNC] Rewrote CONVENTIONS.md to match current codebase
[SYNC] Committed: docs: sync to a1b2c3d
```

**Common Variations:**
- `/sync` after merging one feature branch — updates relevant docs and completes the plan
- `/sync` after returning from vacation — catches up on everything that changed
- `/sync` on first use in a legacy project — full reconciliation, rewrites docs from scratch