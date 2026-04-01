# .kilocode/commands/review.md
---
description: Review the current branch diff against main and generate a review report
mode: architect
arguments:
  - plan_name
---

## Chain of Thought

Follow this reasoning process for every review task:

1. **GATHER**: Collect all data before forming opinions:
   - Run `git diff main..HEAD` and `git diff --stat main..HEAD`
   - Read the plan document at `.plans/$1.md` (or `.plans/epics/*/$1.md` if epic phase)
   - Run the test suite and capture results
   - Read the test baseline from `.plans/$1.baseline-tests.txt`

2. **COMPARE**: Analyze the diff against the plan:
   - Which files in the diff are listed in the plan's "Files to Change"?
   - Which files in the diff are NOT listed? (scope deviations)
   - Do the implementation changes actually solve what the plan describes?

3. **TEST**: Evaluate test results with baseline comparison:
   - Separate failures into **pre-existing** (present in baseline) vs **new** (introduced by this branch)
   - Only new failures count against the verdict
   - Pre-existing failures should be noted but NOT blocking

4. **EVALUATE**: Assess each dimension:
   - **Plan alignment**: Does implementation match what was planned?
   - **Scope**: Are there unplanned file changes?
   - **Code quality**: Naming, patterns, consistency with CONVENTIONS.md
   - **Test quality**: Are tests testing *behavior* not just *execution*?
   - **Acceptance criteria**: Which are automated (have tests)? Which require manual verification?
   - **Completeness**: Any TODOs, placeholder code, or commented blocks?

5. **CLASSIFY**: Categorize each issue found:
   - **🔴 blocking**: Must fix before merge (new test failures, plan misalignment, broken functionality)
   - **🟡 minor**: Should fix (scope deviations, code style issues, test quality gaps)
   - **🟢 suggestion**: Nice to have (improvements that weren't in scope)

6. **VERDICT**: Determine outcome using the criteria below, then verify your verdict by checking it against the criteria again.

---

## Verdict Determination

Use these criteria to determine the verdict. When uncertain, lean toward the stricter interpretation.

### PASS — All must be true:
- [ ] Zero **new** test failures (pre-existing failures are excluded from this check)
- [ ] No 🔴 blocking issues found
- [ ] Implementation aligns with plan's core intent and all planned files are changed
- [ ] No unplanned files that introduce new functionality (scope deviations)
- [ ] All acceptance criteria are either met or marked for manual verification

### NEEDS_WORK — Any must be true:
- [ ] One or more **new** test failures
- [ ] One or more 🔴 blocking issues
- [ ] Core implementation does not match plan's stated intent
- [ ] Unplanned files introduce significant new functionality
- [ ] Acceptance criteria fundamentally unmet

### When Diff Is Unclear

If you cannot determine intent from the diff alone:

1. **Check the plan first** — read the plan's "Scope" and "Implementation Notes" sections
2. **Check git history** — `git log main..HEAD --oneline` shows commit sequence
3. **Check for related plans** — maybe this is part of a multi-phase epic
4. **Flag ambiguity as blocking** — if you cannot determine if something is correct, treat it as a blocking issue requiring clarification
5. **Ask via fixlist** — document the uncertainty in the fixlist with a question rather than a directive

---

## Review Process

Review the implementation on this branch against the plan. Plans are stored at:
- `.plans/<name>.md` for regular tasks
- `.plans/epics/<epic-slug>/<name>.md` for epic phases

The agent should first check `.plans/$1.md`, and if not found, search in `.plans/epics/*/$1.md`.

### Data Gathering

1. **Diff collection:**
   ```bash
   git diff main..HEAD
   git diff --stat main..HEAD
   ```

2. **Plan reading:** Read `.plans/$1.md` (or epic variant)

3. **Test execution with baseline comparison:**
   - Run the test suite using the command from `## Commands` in `AGENTS.md`
   - Read the test baseline from `.plans/$1.baseline-tests.txt`
   - Compare line-by-line: which failures are **pre-existing** vs **new**?
   - Only flag new failures as blocking issues

4. **Code quality check:**
   - Read CONVENTIONS.md for project standards
   - Check naming consistency, pattern adherence, error handling

5. **Test quality assessment:**
   - For test case category definitions, see `{{partials:test-case-categories}}`
   - A test that calls a function and asserts `!= null` without checking actual behavior is **superficial** — flag it
   - Tests should verify behavior, not just execution

6. **Acceptance criteria verification:**
   - Automatable criteria: verify corresponding tests exist and pass
   - Non-automatable criteria (UX, performance): mark as requiring manual verification

7. **Completeness check:**
   - Any TODOs in implementation code?
   - Any placeholder or commented-out blocks?
   - Any incomplete error handling?

### Verification Checkpoints

Before finalizing the verdict, verify:

- [ ] **Diff matches plan scope** — all changed files are either planned or justified
- [ ] **Tests are comprehensive** — Happy Path + Error & Boundary + Integration covered
- [ ] **No superficial tests** — each test checks actual behavior, not just "function returns something"
- [ ] **Baseline comparison correct** — pre-existing failures excluded from verdict
- [ ] **Issue severity consistent** — blocking issues clearly distinguished from minor
- [ ] **Fixlist is actionable** — each issue has specific file:line reference and clear fix guidance

---

## Fixlist Generation (NEEDS_WORK only)

If verdict is NEEDS_WORK, write a fix list to the same directory as the plan, named `<plan-name>.fixlist.md`:

```markdown
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

---

## Report Generation

After review, invoke @reporter to render a review report.

The report must include:

- **Stat card row:** files changed, lines added/removed, new tests passing/failing (excluding pre-existing), verdict (PASS as green tag, NEEDS_WORK as yellow tag)
- **Diff Summary table:** filename, lines changed, planned vs unplanned (tag)
- **Review Findings section:** numbered issues with file reference, severity tag (🔴 blocking, 🟡 minor, 🟢 suggestion), root cause, and fix guidance
- **Test Results section:**
  - **New/Changed tests:** affected by this branch — these determine the verdict
  - **Pre-existing Failures:** present in baseline — noted but NOT blocking
  - Show baseline comparison clearly
- **Scope deviations:** if any, include a mermaid diagram showing planned vs actual files
- **Live Verification section:** if UI changes involved — embed screenshots from `.reports/screenshots/review-$1-*` as base64 with captions
- **Verdict section:** clear PASS/NEEDS_WORK with summary

Save to: `.reports/review-$1-<YYYY-MM-DD>.html`

---

## Examples

**Invocation:**
```
/review add-user-authentication
```

### PASS Verdict Example

```
[VERDICT: PASS] ✅

Stat cards show:
- 5 files changed
- +127 -43 lines
- 12/12 tests passing (baseline had 10/12, 2 pre-existing failures)
- Tag: "PASS" (green)

Review Findings: No issues found. Implementation matches plan.

Test Results:
- New/Changed tests: 12 passing, 0 failing ✅
- Pre-existing failures: 2 (auth timeout edge case, known issue)
```

### NEEDS_WORK Verdict Example

```
[VERDICT: NEEDS_WORK] ⚠️

Stat cards show:
- 6 files changed (1 unplanned)
- +142 -67 lines
- 10/12 tests passing (2 new failures)
- Tag: "NEEDS_WORK" (yellow)

Review Findings:
1. 🔴 blocking: Null pointer in auth service — src/auth/service.ts:L42
   Why: Missing null check on user object causes crash when user lookup returns undefined
   How to fix: Add optional chaining (?.) before accessing user properties

2. 🟡 minor: Unplanned file changed — src/utils/logger.ts
   Why: Not listed in plan's "Files to Change"
   How to fix: Remove from this PR or document justification in plan

3. 🟢 suggestion: Test coverage gap — src/auth/middleware.spec.ts
   Why: Missing test for malformed JWT handling
   How to fix: Add test case for malformed token scenarios
```

### Verdict Edge Cases

**Case A: Pre-existing failures but no new issues**
```
Verdict: PASS (with caveats)

- Baseline had 3 failures
- Current branch adds 0 new failures
- Pre-existing failures noted in report but do not block
- Implementation correctly matches plan
```

**Case B: Tests pass but scope deviation**
```
Verdict: NEEDS_WORK

- All tests passing
- But src/billing/invoice-generator.ts was modified without being in plan
- This introduces new functionality outside original scope
- Requires scope expansion or reverting the unplanned change
```

**Case C: Implementation incomplete (no test failures)**
```
Verdict: NEEDS_WORK

- Tests pass but implementation has TODOs and placeholder code
- auth/service.ts:L42 has "// TODO: implement token refresh"
- Plan promised full token refresh functionality
- Blocking: incomplete feature, not just broken code
```

---

## Structured Output

When invoking @reporter, pass this JSON schema:

```json
{
  "report_type": "review",
  "generated_at": "ISO8601 timestamp",
  "plan_name": "string",
  "verdict": "PASS | NEEDS_WORK",
  "verdict_summary": "string (2-3 sentence summary of why)",
  "stats": {
    "files_changed": "number",
    "files_planned": "number",
    "files_unplanned": "number",
    "lines_added": "number",
    "lines_removed": "number",
    "tests_passed": "number",
    "tests_failed": "number",
    "tests_new_passed": "number",
    "tests_new_failed": "number",
    "pre_existing_failures": "number"
  },
  "diff_summary": [
    {
      "file": "string",
      "lines_added": "number",
      "lines_removed": "number",
      "planned": "boolean",
      "reason_if_unplanned": "string (only if planned is false)"
    }
  ],
  "issues": [
    {
      "severity": "blocking | minor | suggestion",
      "file": "string",
      "line": "number | null",
      "description": "string",
      "root_cause": "string",
      "fix_guidance": "string",
      "verdict_impact": "string (why this affects the verdict)"
    }
  ],
  "test_results": {
    "new_tests": {
      "passed": ["string (test name)"],
      "failed": ["string (test name)"]
    },
    "pre_existing_failures": ["string (test name and reason)"]
  },
  "scope_deviations": [
    {
      "file": "string",
      "reason": "string"
    }
  ],
  "acceptance_criteria_status": [
    {
      "criterion": "string",
      "automated": "boolean",
      "status": "met | unmet | manual_verification_required",
      "evidence": "string (test name or 'manual check required')"
    }
  ],
  "live_verification": {
    "checked": ["string (what was verified)"],
    "passed": ["string"],
    "failed": ["string"]
  },
  "screenshots": [
    {
      "path": "string",
      "caption": "string"
    }
  ]
}
```

---

## Common Variations

| Scenario | Command |
|----------|---------|
| Review current branch against its plan | `/review <plan-name>` |
| Review when tests fail | verdict → NEEDS_WORK, fixlist.md generated |
| Review when scope deviations exist | unplanned files flagged as blocking |
| Review epic phase | `/review <epic-slug>/<phase-name>` |
| Review with pre-existing failures | PASS if no new failures added |
