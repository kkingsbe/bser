# .kilocode/commands/recap.md
---
description: Generate an end-of-session summary report
mode: architect
---
Generate an end-of-session recap by gathering the following data:

1. **Git activity this session:**
   - Run `git log --oneline --since="4 hours ago"` (adjust if needed).
   - Summarize what was built, fixed, or changed.

2. **Plan progress:**
   - Check all `.plans/*.md` files for current status.
   - For any IN_PROGRESS plans: what's done, what's remaining.

3. **Unfinished threads:**
   - Any failing tests? (Run the test command from `AGENTS.md`, capture summary)
   - Any TODOs added this session? (`git diff main..HEAD | grep -i "TODO"`)
   - Any items added to the "Future" section of plan docs?

4. **Recommended next steps:**
   - What should the next session start with?
   - Any blockers or decisions needed before continuing?

Then invoke @reporter to render a recap report.

The report should include:
- A stat card row: commits this session, files changed, tests passing/failing, open plans count
- A "Session Activity" section with a timeline or table of commits
- An "Active Plans" section showing each plan's progress (use a progress indicator or checklist completion %)
- An "Unfinished Threads" section — only if there are failing tests, TODOs, or blockers
- A "Next Session" section with 1-3 prioritized recommended actions
- If multiple plans are active, a mermaid gantt or status diagram showing their states

Save to: `.reports/recap-<YYYY-MM-DD>.html`

## Examples

**Invocation:**
```
/recap
```

**Expected Output:**
HTML report at `.reports/recap-<YYYY-MM-DD>.html` with:
- Stat card row: commits this session, files changed, tests passing/failing, open plans count
- "Session Activity" section: timeline or table of commits
- "Active Plans" section: each plan's progress with progress indicator or checklist completion %
- "Unfinished Threads" section (if applicable): failing tests, TODOs, blockers
- "Next Session" section: 1-3 prioritized recommended actions

**HTML Report Structure:**
```html
<!-- Stat cards -->
<div class="stat-grid">
  <div class="stat-card">
    <div class="value">5</div>
    <div class="label">Commits</div>
  </div>
  <div class="stat-card">
    <div class="value">12</div>
    <div class="label">Files Changed</div>
  </div>
  <div class="stat-card">
    <div class="value">28/30</div>
    <div class="label">Tests Passing</div>
  </div>
  <div class="stat-card">
    <div class="value">2</div>
    <div class="label">Open Plans</div>
  </div>
</div>

<!-- Session Activity -->
<h2>Session Activity</h2>
<table>
  <tr><td>add auth service with JWT validation</td></tr>
  <tr><td>add middleware for request validation</td></tr>
  <tr><td>test baseline: add-user-authentication</td></tr>
</table>

<!-- Active Plans -->
<h2>Active Plans</h2>
<div>
  <h3>add-user-authentication</h3>
  <p>Progress: 4/5 test cases</p>
  <p>Status: IN_REVIEW</p>
</div>

<!-- Unfinished Threads (if any) -->
<h2>Unfinished Threads</h2>
<ul>
  <li>2 tests failing in auth/service.spec.ts</li>
</ul>

<!-- Next Session -->
<h2>Next Session</h2>
<ol>
  <li>Fix auth service null handling issue</li>
  <li>Re-run /review after fixes</li>
</ol>
```

**Common Variations:**
- `/recap` — End of work session summary
- Works best after 1+ hours of work with commits to analyze
```

## Structured Output

When invoking @reporter, pass this JSON schema:

```json
{
  "report_type": "recap",
  "generated_at": "ISO8601 timestamp",
  "stats": {
    "commits_this_session": "number",
    "files_changed": "number",
    "tests_passing": "string",
    "tests_failing": "number",
    "open_plans": "number"
  },
  "session_activity": [
    {
      "commit_hash": "string",
      "message": "string",
      "timestamp": "ISO8601"
    }
  ],
  "active_plans": [
    {
      "name": "string",
      "status": "IN_PROGRESS | IN_REVIEW",
      "progress": "string (e.g., '3/5 test cases')",
      "checklist_completion": "number (percentage)"
    }
  ],
  "unfinished_threads": {
    "failing_tests": ["string"],
    "new_todos": ["string"],
    "blockers": ["string"]
  },
  "recommended_next_actions": ["string"]
}