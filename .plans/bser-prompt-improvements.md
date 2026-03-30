# Plan: Improve BSER Commands and Subagent Prompts Using Prompt Engineering Best Practices

**Status:** IN_PROGRESS
**Category:** refactor
**Branch:** feat/bser-prompt-improvements
**Parent:** main
**Created:** 2026-03-30
**One-liner:** Improve BSER commands and subagent prompts using prompt engineering best practices

## Scope

Refactor and enhance the BSER framework's command definitions and subagent prompts to follow prompt engineering best practices from the `.agents/skills/prompt-engineering-patterns/` skill. This includes adding few-shot examples, structured output schemas, chain-of-thought framing, error recovery mechanisms, and consolidating repeated patterns.

## Files to Change

- `setup.md` — Add few-shot examples to commands, structured output schemas, CoT framing, error recovery sections
- `workflows.md` — Minor updates if command behavior changes

### Subagent Definition Files (in `.kilo/agents/`)

- `.kilo/agents/reviewer.md` — Restructure with explicit role/expertise/constraints/format sections
- `.kilo/agents/syncer.md` — Restructure with explicit role/expertise/constraints/format sections
- `.kilo/agents/reporter.md` — Add JSON input schemas, break into modular template sections

## Test Cases

- [ ] All commands render correctly with examples when invoked
- [ ] Subagent outputs match the new structured format specifications
- [ ] Error recovery sections handle missing inputs gracefully
- [ ] Reporter accepts and renders all new structured JSON inputs correctly

## Implementation Notes

### Phase 1: Add Few-Shot Examples to Commands

For each command in setup.md section 3, add an "Examples" subsection showing:
- Typical invocation with arguments
- Expected output format
- Common variations

**Commands needing examples:**
- `/brief` — Show sample briefing invocation and stat card output
- `/scope` — Show task description → slug derivation → plan structure
- `/epic` — Show decomposition output with mermaid diagram
- `/implement` — Show expected progress update format
- `/review` — Show PASS and NEEDS_WORK verdict examples
- `/sync` — Show before/after doc state
- `/recap` — Show session summary structure
- `/impact` — Show dependency graph output
- `/estimate` — Show calibration report output
- `/hotfix` — Show minimal fix flow

### Phase 2: Restructure Subagent Definitions

Follow the system prompt pattern: `[Role] + [Expertise Areas] + [Behavioral Guidelines] + [Output Format] + [Constraints]`

**For `reviewer.md`:**
```
## Role
You are a senior code reviewer specializing in [tech stack]. Your role is to...

## Expertise Areas
- Diff analysis and change attribution
- Test coverage evaluation
- [Tech-specific] pattern recognition

## Review Process (numbered steps)
1. Read the plan document...
2. Run git diff main..HEAD...
[... continues with CoT framing]

## Output Format
Verdict: PASS | NEEDS_WORK
Issues: [numbered list with file:line references]
Live Verification: [if applicable]
Screenshots: [paths]
Suggestions: [optional]

## Constraints
- Read-only: cannot edit files
- Must run test suite
- Must reference plan document
```

**For `syncer.md`:** Similar structure focused on markdown-only edits

**For `reporter.md`:** Add explicit JSON input schemas for each report type

### Phase 3: Add Structured Output Schemas

Define JSON schemas for data flowing between commands and reporter:

**Review Output Schema:**
```json
{
  "report_type": "review",
  "plan_name": "string",
  "verdict": "PASS" | "NEEDS_WORK",
  "stats": {
    "files_changed": "number",
    "lines_added": "number",
    "lines_removed": "number",
    "tests_passed": "number",
    "tests_failed": "number"
  },
  "issues": [
    {
      "severity": "blocking" | "minor" | "suggestion",
      "file": "string",
      "line": "number",
      "description": "string",
      "root_cause": "string",
      "fix_guidance": "string"
    }
  ],
  "scope_deviations": ["string"],
  "screenshots": ["path"]
}
```

**Similar schemas for:** brief, epic, recap, impact, estimate

### Phase 4: Add Error Recovery Sections

For each command, add a "Error Recovery" subsection:

```
## Error Recovery

- **Missing plan name:** Respond "Usage: /implement <plan-name>"
- **Plan file not found:** Respond "Plan not found: .plans/<name>.md"
- **Test baseline missing:** Proceed without baseline comparison, note in report
- **Git operations fail:** Stop and report error, do not continue
- **agent-browser unavailable:** Skip live verification, note in report
```

### Phase 5: Add Chain-of-Thought Framing to Complex Commands

For `/scope`, `/epic`, `/impact`, `/estimate`:

```
Follow this reasoning process:

1. UNDERSTAND: Read [relevant docs] to understand current state...
2. DECOMPOSE: Break down the task by identifying...
3. ANALYZE: For each component, evaluate...
4. SYNTHESIZE: Combine findings into...
5. VALIDATE: Check against constraints...
```

### Phase 6: Consolidate Repeated Patterns

Extract common sections into include files or document once and reference:

- All commands that invoke @reporter: "After gathering data, invoke @reporter to render a [type] report"
- Error recovery patterns
- File naming conventions (.reports/..., .plans/...)
- Test command references

### Phase 7: Optimize Reporter Template

- Break the 1087-line template into modular sections
- Add a "lite" report variant for quick summaries
- Add CSS prefers-color-scheme consistency pass
- Consider prompt caching for base template structure

## Future (Out of Scope)

- Migrating to a different report rendering system
- Adding support for additional output formats (PDF, markdown)
- Automated testing of prompts against test cases

---

## Completion Log

**Completed:** 
**Actual changes:** 
**Deviated from plan:** 
**Impact:** 