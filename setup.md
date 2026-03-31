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

If you have a repo already using an earlier version of the BSER framework, follow these steps to bring it up to date. Run these from the project root.

### Step 0: Update `.bser-version`

If you don't have a `.bser-version` file yet, create it:

```bash
touch .bser-version
```

Then update it with the new version:

```text
version: <new-version>
date-adopted: <YYYY-MM-DD>
notes: Migrated from <old-version> — see migration guide in setup.md
```

### Step 1: Scaffold Missing Directories

```bash
# These are idempotent — won't overwrite existing content
mkdir -p .kilo/agents
mkdir -p .reports/screenshots
mkdir -p .kilocode/commands
```

### Step 2: Remove Old Custom Modes

If you previously used `.kilocodemodes` or `custom_modes.yaml` for BSER modes (`reviewer`, `syncer`), these are now subagents:

```bash
# Check if old mode config exists
cat .kilocodemodes 2>/dev/null
cat ~/.config/kilo/custom_modes.yaml 2>/dev/null
```

If you find `reviewer` or `syncer` mode definitions in either file, remove them. They're replaced by the subagent markdown files in `.kilo/agents/`. If the files contain other non-BSER modes you still use, keep those — just remove the BSER-specific ones.

### Step 3: Install Subagents

Copy the subagent definitions from `setup/agents/` into `.kilo/agents/`:

```bash
# Read from setup/agents/ and write to .kilo/agents/
cp setup/agents/reviewer.md .kilo/agents/reviewer.md
cp setup/agents/syncer.md .kilo/agents/syncer.md
cp setup/agents/reporter.md .kilo/agents/reporter.md
```

- `.kilo/agents/reviewer.md` — the diff review agent (now with agent-browser live verification)
- `.kilo/agents/syncer.md` — the post-merge doc sync agent
- `.kilo/agents/reporter.md` — the HTML report generator (new — this is the human interface layer)

If you already have these files from a previous version, **replace them entirely** with the current versions from `setup/agents/`. The reporter agent is new and must be added.

### Step 4: Update Slash Commands

Replace all command files in `.kilocode/commands/` with the current versions from `setup/commands/`. The key changes since earlier versions:

| Command | What Changed |
|---------|-------------|
| `/scope` | No longer takes a kebab-case argument. Accepts freeform description, derives slug. Uses `$ARGUMENTS` instead of `$1`. Now epic-branch-aware. |
| `/epic` | **New.** Freeform description input, creates `epic/` branch. |
| `/implement` | Now checks for `.fixlist.md` to enter targeted fix mode after review. |
| `/review` | Now writes a `.fixlist.md` on NEEDS WORK verdict. Renderer spec for `@reporter` with screenshot embedding. |
| `/sync` | Unchanged. |
| `/hotfix` | Now accepts freeform description, derives slug. |
| `/brief` | Now invokes `@reporter` — output is an HTML report, not terminal text. |
| `/recap` | **New.** End-of-session summary via `@reporter`. |
| `/impact` | **New.** Dependency impact analysis via `@reporter`. |
| `/estimate` | **New.** Planned-vs-actual calibration via `@reporter`. |

Copy the commands from `setup/commands/`:

```bash
# Replace all existing commands with current versions from setup/commands/
cp setup/commands/brief.md .kilocode/commands/brief.md
cp setup/commands/scope.md .kilocode/commands/scope.md
cp setup/commands/epic.md .kilocode/commands/epic.md
cp setup/commands/implement.md .kilocode/commands/implement.md
cp setup/commands/review.md .kilocode/commands/review.md
cp setup/commands/sync.md .kilocode/commands/sync.md
cp setup/commands/hotfix.md .kilocode/commands/hotfix.md
cp setup/commands/recap.md .kilocode/commands/recap.md
cp setup/commands/impact.md .kilocode/commands/impact.md
cp setup/commands/estimate.md .kilocode/commands/estimate.md
```

### Step 5: Update Plan Templates

If you have `.plans/_TEMPLATE.md`, update it to include the new `Parent` field. Read the current template from `setup/templates/plan-template.md`:

```bash
cp setup/templates/plan-template.md .plans/_TEMPLATE.md
```

Add the epic template if it doesn't exist:

```bash
# Check if epic template exists
ls .plans/_EPIC_TEMPLATE.md 2>/dev/null

# If missing, copy from setup/templates/
cp setup/templates/epic-template.md .plans/_EPIC_TEMPLATE.md
```

### Step 6: Update AGENTS.md

If your `AGENTS.md` exists but predates the current version, update the `## BSER Workflow` section to match the template from `setup/templates/AGENTS.md`. Key additions:

- Reference to `@reporter` and `@syncer` subagents
- The workflow rules section (TDD, no scope expansion, stop-if-plan-is-wrong)
- Git conventions including `epic/` branch prefix

```bash
# Update from template
cp setup/templates/AGENTS.md AGENTS.md
# Then adapt for your project
```

If `AGENTS.md` doesn't exist, create it from the template and adapt for your project.

### Step 7: Update Existing Plan Docs

Existing `.plans/*.md` files from prior versions will still work — the commands read them by slug. But for consistency, you can optionally update completed plans:

```bash
# Find plans missing the Parent field
grep -rL "Parent:" .plans/*.md 2>/dev/null
```

For any active (IN_PROGRESS) plans, add `**Parent:** main` to the frontmatter so the review command knows the merge target. Completed plans don't need updating.

### Step 8: Install agent-browser

If not already installed:

```bash
npx skills add https://github.com/vercel-labs/agent-browser --skill agent-browser
```

This is used by the `reviewer` agent for live UI verification and the `reporter` agent for capturing screenshots.

### Step 9: Verify

Run through the verification checklist in section 6. Pay special attention to:

- [ ] `@reporter` is invocable and generates HTML reports to `.reports/`
- [ ] `/scope` accepts freeform descriptions (test: `/scope Add a health check endpoint to the API`)
- [ ] `/brief` produces an HTML report that auto-opens in the browser
- [ ] Old custom modes are removed and don't conflict with new subagents

### Migration Cleanup Checklist

```bash
# Remove artifacts from older BSER versions that are no longer used
rm -f .kilocodemodes                     # if it only contained BSER modes
rm -f .plans/*.fixlist.md                # clean up any leftover fixlists from interrupted reviews

# Verify no stale mode references in config
grep -r "reviewer\|syncer" ~/.config/kilo/custom_modes.yaml 2>/dev/null && echo "⚠️  Remove BSER modes from global custom_modes.yaml"

# Verify subagents are in place
ls .kilo/agents/reviewer.md .kilo/agents/syncer.md .kilo/agents/reporter.md
```

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
