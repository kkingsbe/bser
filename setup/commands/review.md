# .kilocode/commands/review.md
---
description: Review the current branch diff against main and generate a review report
mode: architect
arguments:
  - plan_name
---
Review the implementation on this branch against the plan in `.plans/$1.md`.

Gather the following data:

1. Run `git diff main..HEAD` and `git diff --stat main..HEAD` to see all changes.
2. Read the plan document to understand what was intended.
3. **Test suite with baseline comparison:**
   - Run the test suite using the test command from the `## Commands` section of `AGENTS.md` and capture current results.
   - Read the test baseline from `.plans/$1.baseline-tests.txt` (captured during `/scope`).
   - Compare: identify which failures are **pre-existing** (present in baseline) vs **new** (introduced by this branch).
   - Only flag new failures as blocking issues. Pre-existing failures should be noted but NOT count against the verdict.
4. Evaluate:
   - **Plan alignment:** Does the implementation match what was planned? List deviations.
   - **Scope check:** Are there changes to files NOT listed in the plan? Flag each.
   - **Code quality:** Check naming, patterns, consistency with CONVENTIONS.md.
   - **Test coverage:** Are all planned test cases present and passing?
   - **Completeness:** Any TODOs, placeholder code, or commented-out blocks?
   - **Edge cases:** Obvious error handling gaps or uncovered edge cases?
5. Determine a verdict: **PASS** or **NEEDS WORK**.

If NEEDS WORK, also write a fix list to `.plans/$1.fixlist.md`:
```
# Fix List: $1

## Issues to Address

- [ ] Issue 1: <description> — `file.ts:L42`
  **Why this failed:** <root cause explanation — what the code does wrong and why>
  **How to fix:** <specific guidance on the approach to take>

- [ ] Issue 2: <description> — `file.ts:L87`
  **Why this failed:** <root cause explanation>
  **How to fix:** <specific guidance>

## Context for Fix Session

<Overall explanation of what went wrong. Include:
- Which parts of the plan were implemented correctly
- Where the implementation diverged or fell short
- Any patterns in the issues (e.g., "all issues are related to null handling in the parser")
- Relevant code context the implementing agent needs to understand the fix>
```

Then invoke @reporter to render a review report.

The report should include:
- A stat card row: files changed, lines added/removed, new tests passing/failing (excluding pre-existing failures), verdict (PASS as green tag, NEEDS WORK as yellow tag)
- A "Diff Summary" table: filename, lines changed, planned vs unplanned (tag)
- A "Review Findings" section with numbered issues, each with file reference, severity tag (🔴 blocking, 🟡 minor, 🟢 suggestion), root cause, and fix guidance
- A "Test Results" section with two parts: **New/Changed** (tests affected by this branch — these determine the verdict) and **Pre-existing Failures** (failures present in the baseline — noted for awareness but not blocking). Show the baseline comparison clearly.
- If there are scope deviations, a mermaid diagram showing planned files vs actual files changed
- A "Live Verification" section if UI changes were involved — embed any screenshots from `.reports/screenshots/review-$1-*` as base64 images with captions describing what was verified
- A "Verdict" section with clear PASS/NEEDS WORK and summary

Save to: `.reports/review-$1-<YYYY-MM-DD>.html`

## Examples

**Invocation:**
```
/review add-user-authentication
```

**Expected Output:**
HTML report at `.reports/review-<plan-name>-<YYYY-MM-DD>.html` with:
- Stat card row: files changed, lines added/removed, new tests passing/failing (excluding pre-existing failures), verdict (PASS as green tag, NEEDS_WORK as yellow tag)
- "Diff Summary" table: filename, lines changed, planned vs unplanned (tag)
- "Review Findings" section with numbered issues, each with file reference, severity tag (🔴 blocking, 🟡 minor, 🟢 suggestion), root cause, and fix guidance
- "Test Results" section with baseline comparison
- "Verdict" section with clear PASS/NEEDS_WORK and summary

**PASS Verdict Example:**
```
[VERDICT: PASS] ✅

Stat cards show:
- 5 files changed
- +127 -43 lines
- 12/12 tests passing
- Tag: "PASS" (green)

Review Findings: No issues found. Implementation matches plan.
```

**NEEDS_WORK Verdict Example:**
```
[VERDICT: NEEDS_WORK] ⚠️

Stat cards show:
- 6 files changed (1 unplanned)
- +142 -67 lines
- 10/12 tests passing (2 new failures)
- Tag: "NEEDS_WORK" (yellow)

Review Findings:
1. 🔴 blocking: Null pointer in auth service — src/auth/service.ts:L42
   Why: Missing null check on user object
   How to fix: Add optional chaining or null guard

2. 🟡 minor: Unplanned file changed — src/utils/logger.ts
   Why: Not listed in plan's "Files to Change"
```

**Fixlist Output (`.plans/<plan-name>.fixlist.md`):**
```markdown
# Fix List: add-user-authentication

## Issues to Address

- [ ] Issue 1: Null pointer in auth service — `src/auth/service.ts:L42`
  **Why this failed:** Missing null check on user object
  **How to fix:** Add optional chaining or null guard

## Context for Fix Session

<Overall explanation of what went wrong>
```

**Common Variations:**
- `/review` — Review current branch against its plan
- When tests fail: verdict is NEEDS_WORK, fixlist.md is generated
- When scope deviations exist: unplanned files flagged in findings
```

## Structured Output

When invoking @reporter, pass this JSON schema:

```json
{
  "report_type": "review",
  "generated_at": "ISO8601 timestamp",
  "plan_name": "string",
  "verdict": "PASS | NEEDS_WORK",
  "stats": {
    "files_changed": "number",
    "lines_added": "number",
    "lines_removed": "number",
    "tests_passed": "number",
    "tests_failed": "number",
    "new_tests_passed": "number",
    "new_tests_failed": "number",
    "pre_existing_failures": "number"
  },
  "diff_summary": [
    {
      "file": "string",
      "lines_added": "number",
      "lines_removed": "number",
      "planned": "boolean"
    }
  ],
  "issues": [
    {
      "severity": "blocking | minor | suggestion",
      "file": "string",
      "line": "number",
      "description": "string",
      "root_cause": "string",
      "fix_guidance": "string"
    }
  ],
  "scope_deviations": ["string"],
  "live_verification": {
    "checked": ["string"],
    "passed": ["string"],
    "failed": ["string"]
  },
  "screenshots": ["path"],
  "pre_existing_failures_list": ["string"]
}
```