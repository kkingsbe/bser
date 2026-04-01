# BSER Framework — Setup Guide

> One-time setup for the Brief → Scope → Execute → Review+Sync workflow.
> After completing this guide, see **BSER-workflow.md** for daily usage.

---

## Prerequisites

- Kilo Code CLI installed and configured
- Git initialized in your project
- A working Kilo config at `~/.config/kilo/` or `~/.kilocode/`
- agent-browser installed: `npx skills add https://github.com/vercel-labs/agent-browser --skill agent-browser`

---

## 1. Project File Structure

Create these directories and files in your project root:

```bash
mkdir -p .plans/epics
mkdir -p .kilocode/commands
mkdir -p .kilo/agents
mkdir -p .reports/screenshots

touch AGENTS.md
touch ARCHITECTURE.md
touch CONVENTIONS.md
touch .bser-version
touch .plans/backlog.md
touch .kilocode/commands/commands.md
touch BSER-QUICKREF.md
```

### .gitignore Additions

Add these to your project's `.gitignore`. Reports and screenshots are ephemeral outputs — they can always be regenerated. Fixlists are transient review artifacts. Test baselines *are* committed (they're part of the plan's branch).

```gitignore
# BSER ephemeral outputs
.reports/
.plans/*.fixlist.md
```

The following **should** be committed:
- `.bser-version` (framework version — see Versioning section)
- `.plans/*.md` (plan docs, backlog, templates)
- `.plans/epics/` (epic plan directories)
- `.plans/*.baseline-tests.txt` (test baselines — needed by `/review`)
- `AGENTS.md`, `ARCHITECTURE.md`, `CONVENTIONS.md`
- `BSER-QUICKREF.md` (quick reference for daily usage — keep open while working)
- `.kilocode/commands/commands.md` (commands registry)
- `.kilocode/commands/*.md` (slash commands — except commands.md which is the registry)
- `.kilo/agents/*.md` (subagent definitions)

### Versioning

The BSER framework uses a single `.bser-version` file at the project root to track which version of the framework is in use.

**Where the version is defined:** `.bser-version`

**Format:**
```text
version: 1.0
date-adopted: 2024-01-15
notes: Initial setup
```

**How to check the current version:**
```bash
cat .bser-version
```

**How to upgrade:**
1. Update the `version` field in `.bser-version`
2. Run through the [Migration Guide](#7-migrating-from-a-previous-bser-version) if there are breaking changes
3. Update `notes` to record when and why the upgrade happened

The version follows [Semantic Versioning](https://semver.org/): `major.minor.patch`
- `major`: Breaking changes that require migration
- `minor`: New commands, agents, or features (backward compatible)
- `patch`: Bug fixes, documentation updates

### File Templates

For each of the following files, read the template from `setup/templates/` and write it to the target location:

| Target Location | Template Source |
|----------------|-----------------|
| `AGENTS.md` | `setup/templates/AGENTS.md` |
| `ARCHITECTURE.md` | `setup/templates/ARCHITECTURE.md` |
| `CONVENTIONS.md` | `setup/templates/CONVENTIONS.md` |
| `.plans/backlog.md` | `setup/templates/backlog-template.md` |
| `.kilocode/commands/commands.md` | `setup/templates/commands-registry-template.md` |

**Process:**
1. Read the template file from `setup/templates/`
2. Adapt the content for your specific project (fill in `<placeholder>` values)
3. Write to the target location

---

## 2. Plan Document Template

Reference these templates from `setup/templates/` for plan documents:

| Template | Purpose | Location |
|----------|---------|----------|
| `setup/templates/plan-template.md` | Standard implementation plan | `.plans/_TEMPLATE.md` |
| `setup/templates/epic-template.md` | Multi-phase epic plan | `.plans/_EPIC_TEMPLATE.md` |

**Process:**
1. Read `setup/templates/plan-template.md` and save as `.plans/_TEMPLATE.md`
2. Read `setup/templates/epic-template.md` and save as `.plans/_EPIC_TEMPLATE.md`

These templates are used by `/scope` and `/epic` commands to generate new plans.

---

## 3. Custom Slash Commands

Create each slash command by reading from `setup/commands/` and writing to `.kilocode/commands/`:

> **Examples:** See `setup/examples/` for walkthroughs showing each command in context. [Feature Workflow Example](../setup/examples/workflows/feature-workflow.md) demonstrates the full loop; [Epic Workflow Example](../setup/examples/workflows/epic-workflow.md) shows `/epic` decomposition.

| Command | Source | Target |
|---------|--------|--------|
| `/brief` | `setup/commands/brief.md` | `.kilocode/commands/brief.md` |
| `/scope` | `setup/commands/scope.md` | `.kilocode/commands/scope.md` |
| `/epic` | `setup/commands/epic.md` | `.kilocode/commands/epic.md` |
| `/implement` | `setup/commands/implement.md` | `.kilocode/commands/implement.md` |
| `/review` | `setup/commands/review.md` | `.kilocode/commands/review.md` |
| `/sync` | `setup/commands/sync.md` | `.kilocode/commands/sync.md` |
| `/hotfix` | `setup/commands/hotfix.md` | `.kilocode/commands/hotfix.md` |
| `/recap` | `setup/commands/recap.md` | `.kilocode/commands/recap.md` |
| `/impact` | `setup/commands/impact.md` | `.kilocode/commands/impact.md` |
| `/estimate` | `setup/commands/estimate.md` | `.kilocode/commands/estimate.md` |
| `/update` | `setup/commands/update.md` | `.kilocode/commands/update.md` |

**Process:**
1. For each command above, read the source file from `setup/commands/`
2. Write it to the target location in `.kilocode/commands/`

**Design principle:** Every command that produces human-facing context (briefs, recaps, reviews, impact analyses, estimates) gathers its data and then invokes `@reporter` to render a visual HTML report. Action commands (scope, implement, sync, hotfix) that produce code or docs stay as terminal output.

---

## 4. Custom Subagents

Create each subagent by reading from `setup/agents/` and writing to `.kilo/agents/`:

| Agent | Source | Target |
|-------|--------|--------|
| `reviewer` | `setup/agents/reviewer.md` | `.kilo/agents/reviewer.md` |
| `syncer` | `setup/agents/syncer.md` | `.kilo/agents/syncer.md` |
| `reporter` | `setup/agents/reporter.md` | `.kilo/agents/reporter.md` |

**Process:**
1. For each agent above, read the source file from `setup/agents/`
2. Write it to the target location in `.kilo/agents/`

Subagents are specialized agents defined as markdown files with YAML frontmatter. They run in isolated sessions with their own prompts, models, and permissions. The filename (minus `.md`) becomes the agent name.

---

## 5. Optional: Shell Aliases

Add to your `~/.bashrc`, `~/.zshrc`, or shell config for shortcuts outside the Kilo CLI:

```bash
# Start a Kilo session and immediately brief
alias kbrief='kilocode --mode architect -m "Run /brief"'

# Quick check on what plans are open
alias plans='ls -la .plans/ && echo "---" && grep -l "IN_PROGRESS\|IN_REVIEW" .plans/*.md 2>/dev/null || echo "No active plans"'

# Show current branch status vs main
alias branchstat='echo "Branch: $(git branch --show-current)" && echo "Commits ahead of main:" && git log main..HEAD --oneline'
```

---

## 6. Verification Checklist

After setup, confirm everything works:

- [ ] `AGENTS.md` exists at project root with BSER workflow section
- [ ] `.plans/` directory exists with `backlog.md` and `_TEMPLATE.md`
- [ ] `.kilo/agents/` directory exists with `reviewer.md`, `syncer.md`, and `reporter.md`
- [ ] `.reports/` directory exists with `screenshots/` subdirectory
- [ ] `ARCHITECTURE.md` exists (even if minimal)
- [ ] `CONVENTIONS.md` exists (even if minimal)
- [ ] `agent-browser` is installed (`agent-browser --help`)
- [ ] `/brief` command works in Kilo CLI (type `/brief` in interactive mode)
- [ ] `/scope` command works (test: `/scope test-setup`)
- [ ] `/implement` command works
- [ ] `/review` command works
- [ ] `/sync` command works
- [ ] `/hotfix` command works
- [ ] `/recap` command works
- [ ] `/impact` command works
- [ ] `/estimate` command works
- [ ] Reviewer agent is invocable (`@reviewer` in the CLI)
- [ ] Syncer agent is invocable (`@syncer` in the CLI)
- [ ] Reporter agent is invocable (`@reporter` in the CLI)
- [ ] Delete the test branch: `git checkout main && git branch -D feat/test-setup`

---

## 7. Migrating from a Previous BSER Version

The `/update` command is the migration mechanism for bringing an existing BSER project up to date. It handles the entire upgrade process automatically.

### What `/update` Does

When you run `/update`, it:

1. **Detects local modifications** — scans your existing `AGENTS.md`, `ARCHITECTURE.md`, and `CONVENTIONS.md` to identify any customizations you've made
2. **Creates a full backup** — copies all three files to `.plans/` before making any changes
3. **Replaces all commands and agents** — copies fresh versions of every command (`setup/commands/`) and subagent (`setup/agents/`) into your project
4. **Fills in templated sections** — reads your backups and intelligently fills in template placeholders (like project name, branch conventions) from your existing content
5. **Updates the version file** — bumps `.bser-version` to the latest version

### Preserving Customizations

The template filling is designed to preserve your customizations automatically. If you've edited `AGENTS.md`, `ARCHITECTURE.md`, or `CONVENTIONS.md`, the `/update` command reads your content and carries it forward into the fresh templates.

**If you have intentional customizations you want to discard** and start fresh with the new templates, run:

```bash
/update --force
```

This bypasses the template-filling step and replaces everything with clean copies from the framework.

### Version File

The `.bser-version` file tracks which framework version your project is using:

```text
version: <new-version>
date-adopted: <YYYY-MM-DD>
notes: Migrated from <old-version>
```

If you need to manually update the version (for example, if you skipped a version), edit `.bser-version` directly.

### Install agent-browser

If not already installed:

```bash
npx skills add https://github.com/vercel-labs/agent-browser --skill agent-browser
```

This is used by the `reviewer` agent for live UI verification and the `reporter` agent for capturing screenshots.

### Verification Checklist

After migration, confirm everything works:

- [ ] `/update` runs without errors
- [ ] `.bser-version` reflects the new version
- [ ] All slash commands are present in `.kilocode/commands/`
- [ ] All subagents are present in `.kilo/agents/`
- [ ] `@reviewer`, `@syncer`, and `@reporter` are all invocable
- [ ] `/brief` produces an HTML report
- [ ] `/scope` accepts freeform descriptions
- [ ] `agent-browser` is installed and working

---

## 8. Quick Reference

Create `BSER-QUICKREF.md` by reading from `setup/templates/quickref-template.md`:

```bash
cp setup/templates/quickref-template.md BSER-QUICKREF.md
```

This file serves as the daily usage reference. Keep it open while working.

---

## 9. Updating the Framework

This setup is meant to evolve. Common adjustments:

- **Tweak command prompts** as you learn what works for your projects. The prompts in `.kilocode/commands/` are just markdown — edit freely.
- **Add project-specific commands** in `.kilocode/commands/` that override the global ones when a project needs different behavior (e.g., Apollo's `/brief` might include service health checks).
- **Add subagents** for other workflows you formalize (e.g., a `spike-explorer` agent with read-only permissions for codebase exploration before scoping).
- **Tune subagent permissions** — the `reviewer` and `syncer` permission blocks can be adjusted per project (e.g., allow `cargo test` instead of `npm test`).
- **Adjust the plan template** based on what information you actually reference during implementation.

Once setup is complete, you're ready to use BSER. The quick reference above summarizes the workflow — see **BSER-workflow.md** for detailed daily usage.
