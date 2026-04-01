# Plan: bser-refactor-planning

**Status:** IN_PROGRESS
**Category:** chore
**Branch:** docs/bser-refactor-planning
**Parent:** main
**Created:** 2026-04-01
**One-liner:** Document refactoring opportunities for the BSER prompt/markdown codebase

## Scope

This is a meta-plan documenting refactoring opportunities for the BSER framework's own markdown/prompt templates. It captures recommendations, referenced skills, and prioritized action items. No implementation occurs here — this document serves as the tracking artifact for subsequent refactoring work.

## Skills Available for Refactoring Work

| Skill | Path | Applicability |
|-------|------|---------------|
| prompt-engineering-patterns | `.agents/skills/prompt-engineering-patterns/SKILL.md` | High — Core prompts can benefit from structured output enforcement, chain-of-thought patterns, few-shot learning, and template optimization |
| prompt-optimizer | `.agents/skills/prompt-optimizer/SKILL.md` | High — Analyzes prompts for intent, gaps, missing context, and outputs optimized versions |
| ui-ux-pro-max | `.agents/skills/ui-ux-pro-max/SKILL.md` | Low — Not applicable to markdown/template refactoring |
| makefile | `.agents/skills/makefile/SKILL.md` | Low — Not applicable to markdown files |

---

## Refactoring Recommendations

### 1. DRY/Template Extraction

**Problem:** Several patterns are duplicated across commands and agents, making maintenance harder and risking inconsistency.

**Duplicated Patterns Identified:**

| Pattern | Files Where Found | Refactor Approach |
|---------|------------------|------------------|
| TDD/test-first guidance | `setup/commands/scope.md`, `setup/commands/implement.md` | Extract to shared partial: `setup/templates/partials/tdd-workflow.md` |
| Epic context extraction workflow | `setup/commands/scope.md`, `setup/commands/implement.md` | Extract to shared partial: `setup/templates/partials/epic-context-workflow.md` |
| Test case categories (Happy Path / Error & Boundary / Integration) | `setup/commands/scope.md`, `setup/templates/plan-template.md` | Define once in `setup/templates/partials/test-case-categories.md`, reference in both |
| Git branch naming and slug derivation | `setup/commands/scope.md` (multiple references) | Extract to shared partial: `setup/templates/partials/slug-derivation.md` |
| Fixlist handling | `setup/commands/implement.md` | Could be reusable sub-workflow in `setup/templates/partials/fixlist-workflow.md` |

**Benefit:** When TDD guidance changes, update one file instead of N. Ensures consistency across commands.

**Skills Referenced:** `prompt-engineering-patterns` (Template Systems section)

---

### 2. Prompt Optimization

**Problem:** Commands were written organically and may have gaps, ambiguous instructions, or suboptimal structure.

**Analysis Approach:**
1. Run each command through `prompt-optimizer` skill to get structured analysis
2. Review the 6-phase pipeline output (Intent Detection → Scope Assessment → ECC Component Matching → Missing Context Detection → Workflow & Model Recommendation)
3. Apply optimizations from Section 3 of `prompt-optimizer` skill

**Priority Commands to Optimize:**

| Command | File | Why |
|---------|------|-----|
| `/scope` | `setup/commands/scope.md` | Most complex, many branching decisions |
| `/epic` | `setup/commands/epic.md` | Complex multi-phase reasoning |
| `/implement` | `setup/commands/implement.md` | Core execution command, detailed TDD guidance |
| `/review` | `setup/commands/review.md` | Verification workflow needs clear pass/fail criteria |

**Specific Issues to Address:**

| Issue | Impact | Suggested Fix |
|-------|--------|---------------|
| No explicit "ask for clarification" guidance when context is ambiguous | May proceed with wrong assumptions | Add Phase 4-style missing context detection |
| Output format not machine-parseable for some commands | Hard to integrate with other tools | Add JSON output option where appropriate |
| Examples are text-only, no few-shot demonstrations | Inconsistent behavior on edge cases | Add concrete examples for common variations |

**Skills Referenced:** `prompt-optimizer` (full skill), `prompt-engineering-patterns` (Prompt Optimization section)

---

### 3. Consistency Standardization

**Problem:** Inconsistent YAML frontmatter, variable naming conventions, and section ordering across templates.

**Standardization Targets:**

| Target | Current State | Standard |
|--------|---------------|----------|
| YAML frontmatter fields | Inconsistent — some have `name`, `description`, `mode`, `arguments`, others missing fields | Require: `name`, `description`, `mode`; optional: `arguments`, `outputs`, `temperature`, `permission` |
| Variable interpolation format | Mix of `<angle-bracket-description>` and inconsistent formats | Standardize on `<angle-bracket-description>` per AGENTS.md convention |
| Section ordering | Varies between templates | Establish canonical order: Role → Expertise/Responsibilities → Process → Output Format → Constraints → Examples |
| Code block language tags | Inconsistent or missing | Always specify language for syntax highlighting |

**Example Frontmatter Standard:**
```yaml
---
name: <command-name>
description: "<one-liner describing what this command does>"
mode: architect|code|review
arguments:
  - name: <argument-name>
    description: <what it means>
    required: true|false
outputs:
  format: markdown|json|html
  destination: <default output path if applicable>
---
```

**Skills Referenced:** `prompt-engineering-patterns` (Template Systems section)

---

### 4. Structured Output Enforcement

**Problem:** Commands that produce data for other agents (e.g., `brief` → `@reporter`, `review` → `@reporter`) use freeform markdown that requires fragile parsing.

**Affected Commands:**

| Command | Consumer | Current Output | Desired Output |
|---------|----------|----------------|----------------|
| `/brief` | `@reporter` agent | Markdown with tables | JSON matching explicit schema |
| `/review` | `@reporter` agent | Markdown summary | JSON with typed `issues[]`, `screenshots[]`, `verdict` |
| `/recap` | `@reporter` agent | Unknown | JSON with typed sections |
| `/impact` | `@reporter` agent | Unknown | JSON with typed impact areas |

**Benefits:**
- Machine-parseable for reliable tool integration
- Enables `@reporter` to render consistently
- Easier to test command output correctness
- Supports future programmatic analysis

**Implementation Approach:**
1. Define JSON schema for each command's output (inline in command file)
2. Include sample JSON in command documentation
3. Update `@reporter` agent to accept JSON input directly

**Skills Referenced:** `prompt-engineering-patterns` (Structured Outputs section), `prompt-optimizer` (Phase 3: ECC Component Matching)

---

### 5. Chain-of-Thought Patterns for Complex Commands

**Problem:** Complex commands like `/scope` and `/epic` require multi-step reasoning but don't explicitly guide the LLM through the reasoning process.

**Affected Commands:**

| Command | Current Reasoning | Improved Approach |
|---------|-------------------|-------------------|
| `/scope` | Implicit in long prompt | Add explicit numbered reasoning steps with phase markers |
| `/epic` | Implicit | Add "decomposition chain of thought" |
| `/implement` | TDD steps but no explicit reasoning | Add "verify before proceeding" checkpoints |

**Pattern to Apply (from `prompt-engineering-patterns` skill):**
```markdown
## Chain of Thought

Follow this reasoning process:

1. **UNDERSTAND** — [What to read, what to extract]
2. **INTERPRET** — [How to parse the input]
3. **DERIVE** — [How to produce output]
4. **CONSTRAIN** — [How to identify boundaries]
5. **DECOMPOSE** — [How to break into deliverable pieces]
6. **VALIDATE** — [How to confirm the plan is achievable]

Keep reasoning explicit in your output — show which step you're at as you work through it.
```

**Benefits:**
- More consistent output quality
- Easier to debug when something goes wrong
- Helps human review the reasoning process
- Reduces "jumping to conclusions" errors

**Skills Referenced:** `prompt-engineering-patterns` (Chain-of-Thought Prompting section)

---

## Files to Change

This plan, if executed, would touch:

| Category | Files |
|----------|-------|
| New partials | `setup/templates/partials/*.md` (new directory) |
| Commands | All files in `setup/commands/` |
| Agents | `setup/agents/reviewer.md`, `setup/agents/reporter.md`, `setup/agents/syncer.md` |
| Templates | `setup/templates/plan-template.md`, `setup/templates/epic-template.md` |
| Documentation | `setup.md` (update if workflow changes) |

---

## Test Cases

### Happy Path
- [ ] New partial files are created and referenced correctly in commands
- [ ] Optimized prompts produce same outputs with clearer reasoning traces
- [ ] YAML frontmatter passes schema validation
- [ ] JSON outputs parse correctly when consumed by agents

### Error & Boundary
- [ ] Commands with ambiguous input still produce helpful "need clarification" output
- [ ] Partial file missing: command falls back to inline content gracefully
- [ ] JSON output malformed: consumer agent handles gracefully with fallback

### Integration
- [ ] `/brief` → `@reporter` chain produces valid HTML report
- [ ] `/scope` produces plan that `/implement` can read and execute
- [ ] Epic phase context flows correctly between commands

---

## Acceptance Criteria

- Commands with shared partials behave identically to original versions
- Optimized prompts produce equivalent or improved output quality
- All YAML frontmatter follows the standard schema
- JSON-output commands produce parseable, schema-compliant output
- Chain-of-thought commands show explicit reasoning in output
- No duplication — each piece of guidance lives in exactly one place

---

## Implementation Notes

**Execution Order (recommended):**
1. Create partials first (foundation)
2. Update commands to reference partials (low risk, high reward)
3. Run `prompt-optimizer` on each command (analysis, then apply recommendations selectively)
4. Standardize frontmatter across all files
5. Add structured JSON output to report-generating commands
6. Add chain-of-thought sections to complex commands

**Each step can be a separate PR/phase for reviewability.**

---

## Future (Out of Scope)

- Refactoring the `setup/examples/` workflow demonstrations (separate effort)
- Creating a "prompt testing" framework to validate command outputs automatically
- Building a dashboard to visualize BSER adoption metrics across projects
- Writing a "prompt style guide" for future command authors