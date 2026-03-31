# BSER Framework

**Brief → Scope → Execute → Review+Sync**

BSER is a structured methodology for human-in-the-loop agentic development with CLI coding agents. Built for [Kilo Code CLI](https://kilo.ai/cli), but the principles apply to any agent-assisted workflow.

## Self-Bootstrapping

This repository **is** the BSER framework. To use it in your own project, bootstrap it into your Kilo CLI instance:

1. Copy the contents of `bootstrap.md` from this repository into a fresh chat with your Kilo CLI agent
2. It will bootstrap and/or update and migrate your project to use the latest version of BSER

The bootstrap process copies all BSER components—slash commands, subagents, templates, and living documentation—into your project on session start.

## Feature Overview

### Workflow Methodology

BSER divides agentic work into four phases:

1. **Brief** — Rebuild your mental model. Understand what changed, what's in progress, what's next.
2. **Scope** — Decide what to build and plan the approach. Plans are the contract between you and the agent.
3. **Execute** — Steer the agent through implementation against the plan.
4. **Review+Sync** — Verify output and update documentation as a side effect of every change.

### Slash Commands

| Command | Purpose |
|---------|---------|
| `/brief` | Generate a visual snapshot of project state |
| `/scope` | Define and plan a task |
| `/epic` | Decompose large work into ordered phases |
| `/implement` | Execute implementation against a plan |
| `/review` | Verify output and run tests |
| `/sync` | Update living documentation |
| `/hotfix` | Quick-fix workflow with review |
| `/recap` | End-of-session summary |
| `/impact` | Analyze effect of changes |
| `/estimate` | Size task effort |

### Subagents

- **`@reviewer`** — Read-only code review with live browser verification. Checks that UI changes render correctly and screenshots are captured.
- **`@syncer`** — Post-merge documentation updates. Keeps ARCHITECTURE.md and CONVENTIONS.md current as a side effect of changes.
- **`@reporter`** — HTML report generator. Renders briefs, reviews, recaps, and analyses as polished self-contained HTML with stat cards, tables, mermaid diagrams, and embedded screenshots.

### Living Documents

- **AGENTS.md** — Project-wide agent behavior rules
- **ARCHITECTURE.md** — Auto-updated system design documentation
- **CONVENTIONS.md** — Auto-updated coding patterns and decisions

### Plan Documents

The `.plans/` directory contains per-task plans, epic decomposition docs, and a backlog. Every task gets a plan before implementation begins. The agent implements against the plan; scope creep gets captured in a "Future" section.

### Visual Reports

Every human-facing output renders as a polished HTML report. Reports open automatically in your browser and include mermaid diagrams, stat cards, tables, and embedded screenshots.

### Epic Branch Support

Multi-module refactors and large features use `/epic` to create a dedicated long-lived branch with ordered phases. Each phase follows the normal BSER loop and merges back into the epic branch. Main stays clean until the epic is complete.

## Getting Started

1. Copy `bootstrap.md` from this repository to `~/.kilo/bootstrap.md`
2. Start a new Kilo session in your project directory
3. Run `/brief` to begin

## Requirements

- [Kilo Code CLI](https://kilo.ai/cli)
- Git
- [agent-browser](https://github.com/vercel-labs/agent-browser) — for live UI verification and screenshot capture

## License

MIT
