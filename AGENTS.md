# AGENTS.md

> **You are working on the BSER framework repository itself — not a project that uses BSER.**
>
> This repo is a **template factory**. It contains markdown files that get copied into other projects via a bootstrap process. Your job is to **edit these templates**, not execute them.

## What This Repo Is

BSER (Brief → Scope → Execute → Review+Sync) is a structured methodology for human-in-the-loop agentic development. This repository contains:

- **Template files** that get copied into target projects during bootstrap
- **Documentation** describing the methodology
- **No application code** — everything here is markdown

When a user adopts BSER, they copy `bootstrap.md` into their Kilo CLI config. On session start, the CLI clones this repo to a temp directory, reads `setup.md`, and copies the templates into the user's project. Then the temp directory is deleted. That's the entire "deployment" — there is no build step, no compilation, no runtime.

## Critical: Do NOT Do These Things

- **Do NOT run `bootstrap.md`** — it contains instructions for Kilo CLI to execute in a *different* project, not here.
- **Do NOT create `.plans/`, `.kilocode/`, `.kilo/`, or `.reports/` directories in this repo** — those are created inside target projects during bootstrap. They do not belong here.
- **Do NOT treat files in `setup/commands/` as live slash commands** — they are templates that will be copied to `.kilocode/commands/` in a target project.
- **Do NOT treat files in `setup/agents/` as invocable subagents** — they are templates that will be copied to `.kilo/agents/` in a target project.
- **Do NOT treat `setup/templates/AGENTS.md` as this repo's AGENTS.md** — that file is a template for target projects. *This* file (`./AGENTS.md` at repo root) is the one that governs work on this repo.

## Skills

Relevant skills for working on BSER can be found in `.agents/skills/`.

## Repo Structure

```
bser/
├── AGENTS.md                  ← YOU ARE HERE — governs work on this repo
├── README.md                  ← Public-facing project description
├── bootstrap.md               ← Entry point: Kilo CLI reads this to set up a target project
├── setup.md                   ← Step-by-step setup instructions (executed in target projects)
├── workflows.md               ← Daily usage guide for BSER users
└── setup/                     ← All template files that get copied into target projects
    ├── commands/              ← Slash command templates → copied to .kilocode/commands/
    │   ├── brief.md
    │   ├── scope.md
    │   ├── epic.md
    │   ├── implement.md
    │   ├── review.md
    │   ├── sync.md
    │   ├── hotfix.md
    │   ├── recap.md
    │   ├── impact.md
    │   └── estimate.md
    ├── agents/                ← Subagent templates → copied to .kilo/agents/
    │   ├── reviewer.md
    │   ├── syncer.md
    │   └── reporter.md
    └── templates/             ← Doc templates → copied to project root and .plans/
        ├── AGENTS.md          ← Template for target project's AGENTS.md (NOT this file)
        ├── ARCHITECTURE.md
        ├── CONVENTIONS.md
        ├── backlog-template.md
        ├── commands-registry-template.md
        ├── epic-template.md
        ├── plan-template.md
        └── quickref-template.md
```

## How Changes Propagate

When you edit a file in this repo, here's what it affects:

| You edit... | It changes... |
|-------------|---------------|
| `setup/commands/scope.md` | What `/scope` does in every future bootstrapped project |
| `setup/agents/reviewer.md` | How `@reviewer` behaves in every future bootstrapped project |
| `setup/templates/plan-template.md` | The structure of every new plan doc created via `/scope` |
| `setup/templates/AGENTS.md` | The default AGENTS.md scaffold for new projects |
| `setup.md` | The bootstrap process itself — what gets created and where |
| `bootstrap.md` | The entry point that Kilo CLI reads to kick off setup |
| `workflows.md` | The daily usage guide that users reference |
| `README.md` | The public-facing project description (GitHub, etc.) |

**Existing projects are NOT affected by changes here.** Changes only apply to newly bootstrapped projects (or projects that manually re-run setup/migration).

## How to Test Changes

Changes to this repo cannot be validated inside this repo. You must bootstrap into a test project:

```bash
# 1. Create a temporary test project
mkdir /tmp/bser-test && cd /tmp/bser-test
git init

# 2. Copy the bootstrap file (pointing to your local repo, not GitHub)
#    Edit bootstrap.md temporarily to clone from your local path instead of GitHub
cp /path/to/bser/bootstrap.md .

# 3. Run through the setup manually or via Kilo CLI
#    Read setup.md and execute the steps against the test project

# 4. Verify the expected files were created
ls -la .plans/ .kilocode/commands/ .kilo/agents/ AGENTS.md ARCHITECTURE.md

# 5. Test specific commands if you changed them
#    (requires Kilo CLI configured in the test project)

# 6. Clean up
rm -rf /tmp/bser-test
```

For smaller changes (fixing a typo in a command prompt, adjusting wording), you can review the diff directly. For structural changes (new commands, new agents, changes to setup.md flow), always bootstrap into a test project.

## Making Changes

### Adding a new slash command

1. Create `setup/commands/<command-name>.md` with the command definition (use existing commands as reference for format — YAML frontmatter + prompt body)
2. Add the command to the table in `setup.md` (Section 3: Custom Slash Commands)
3. Add it to `setup/templates/commands-registry-template.md`
4. Add it to the command reference tables in `workflows.md` and `setup/templates/quickref-template.md`
5. If the command produces HTML reports, document the report type in `setup/agents/reporter.md`

### Adding a new subagent

1. Create `setup/agents/<agent-name>.md` with the agent definition (YAML frontmatter with permissions + role/process description)
2. Add the agent to the table in `setup.md` (Section 4: Custom Subagents)
3. Add it to the subagent reference tables in `workflows.md` and `setup/templates/quickref-template.md`

### Modifying a template

1. Edit the file in `setup/templates/`
2. If you changed the structure (added/removed sections), update any commands that reference that template (e.g., if you change the plan template, check `/scope` and `/implement` which both read it)
3. If migrating from a previous version, add migration notes to `setup.md` Section 7

### Changing the bootstrap/setup flow

1. Edit `bootstrap.md` and/or `setup.md`
2. Update the verification checklist in `setup.md` Section 6
3. Always test by bootstrapping into a fresh test project

## Code Style (for this repo)

- All files are markdown — no application code
- Command files use YAML frontmatter (`---` delimited) for metadata
- Use ATX-style headers (`#`, `##`, `###`)
- Tables use pipe syntax with header separators
- Code blocks specify language for syntax highlighting
- Keep command prompts focused — if a prompt exceeds ~150 lines, consider whether it's trying to do too much
- Mermaid diagrams use the `mermaid` code fence language tag
- Placeholder values in templates use `<angle-bracket-description>` format

## Git Conventions

- Primary branch: `main`
- Branch naming: `feat/<description>`, `fix/<description>`
- Commit messages: imperative mood, lowercase, no period
    - `add /verify command for post-manual-test feedback`
    - `fix scope command edge case handling guidance`
    - `update plan template with acceptance criteria section`
- Prefix docs-only commits with `docs:` — e.g., `docs: clarify epic decomposition in workflows.md`

## Dependencies

This repo has no runtime dependencies. It is pure markdown.

The **target projects** that bootstrap from this repo may depend on:
- [Kilo Code CLI](https://kilo.ai/cli) — the agent CLI that reads these commands and agents
- [agent-browser](https://github.com/vercel-labs/agent-browser) — used by `@reviewer` for live UI verification
- Git — for branch management