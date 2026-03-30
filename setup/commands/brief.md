# .kilocode/commands/brief.md
---
description: Generate a project briefing to rebuild mental model before starting work
mode: architect
---
Generate a development briefing for this project by gathering the following data:

1. RECENT CHANGES
   - Run `git log --oneline --since="48 hours ago"` (or since last merge to main if more recent).
   - For each commit, note: what changed, which modules/services were touched.
   - Check for any new dependencies or patterns introduced.

2. CURRENT STATE
   - What branch am I on? What is its status vs main?
   - Are there uncommitted changes?
   - Scan .plans/ for any documents with status IN_PROGRESS or IN_REVIEW.

3. EPIC STATUS
   - Check for any `epic/*` branches: `git branch --list 'epic/*'`
   - For each active epic, read its plan doc and summarize: how many phases total, how many complete, what's the current/next phase.
   - Flag if any epic branch has diverged significantly from main (more than 20 commits behind).

4. ARCHITECTURE SNAPSHOT
   - Read ARCHITECTURE.md. Flag anything in recent commits that contradicts, extends, or is missing from it.
   - List the top-level module boundaries and their responsibilities.

5. SUGGESTED NEXT ACTIONS
   - Based on open plans, epic progress, and recent momentum, identify 1-3 logical next tasks.
   - Flag anything that looks broken, half-finished, or blocking.

Once you have all the data, invoke @reporter to render it as a briefing report.

The report should include:
- A stat card row: commits in last 48h, open plans count, active epics count, current branch, uncommitted file count
- A "Recent Changes" section with a concise table of commits and affected modules
- A "Current State" section showing branch status and open plans with their progress
- An "Epic Progress" section (only if active epics exist) — for each epic show: phase completion as a progress bar or fraction, current phase name, and whether the epic branch needs rebasing against main
- An architecture snapshot section — only if there are flags to raise
- A "Suggested Next Actions" section with 1-3 prioritized tasks
- If there are 3+ recent commits touching different modules, include a mermaid graph showing which modules were affected

Save to: `.reports/brief-<YYYY-MM-DD>.html`

## Examples

**Invocation:**
```
/brief
```

**Expected Output:**
HTML report at `.reports/brief-<YYYY-MM-DD>.html` with:
- Stat card row: commits in last 48h, open plans count, active epics count, current branch, uncommitted file count
- "Recent Changes" section: table of commits and affected modules
- "Current State" section: branch status, open plans with progress
- "Epic Progress" section (if epics exist): phase completion fraction, current phase, rebase flag
- "Suggested Next Actions" section: 1-3 prioritized tasks

**Common Variations:**
- `/brief` — Full project briefing after returning from break
- Run on any branch to rebuild context before starting work
```

## Structured Output

When invoking @reporter, pass this JSON schema:

```json
{
  "report_type": "brief",
  "generated_at": "ISO8601 timestamp",
  "current_branch": "string",
  "stats": {
    "commits_48h": "number",
    "open_plans": "number",
    "active_epics": "number",
    "uncommitted_files": "number"
  },
  "recent_changes": [
    {
      "commit_hash": "string",
      "message": "string",
      "affected_modules": ["string"]
    }
  ],
  "current_state": {
    "branch_status": "string",
    "open_plans": [
      {
        "name": "string",
        "status": "IN_PROGRESS | IN_REVIEW",
        "progress": "string"
      }
    ]
  },
  "epic_progress": [
    {
      "name": "string",
      "total_phases": "number",
      "completed_phases": "number",
      "current_phase": "string",
      "needs_rebase": "boolean"
    }
  ],
  "architecture_flags": ["string"],
  "suggested_next_actions": ["string"]
}
```