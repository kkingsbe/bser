# BSER Framework — Bootstrap

This bootstraps the BSER (Brief → Scope → Execute → Review+Sync) framework into a new project.

## Process

1. **Clone this repo** to a temporary location:
   ```bash
   git clone --depth 1 https://github.com/kkingsbe/bser-workflow.git /tmp/bser-setup
   ```

2. **Read `setup.md`** from the cloned repo:
   ```bash
   cat /tmp/bser-setup/setup.md
   ```

3. **Follow the setup instructions** in `setup.md` to create the project structure, configuration files, and BSER workflow commands and agents.

The setup will create:
- `.plans/` directory with plan templates
- `.kilocode/commands/` with slash commands (brief, scope, epic, implement, review, sync, hotfix, recap, impact, estimate)
- `.kilo/agents/` with subagents (reviewer, syncer, reporter)
- Project documentation templates (AGENTS.md, ARCHITECTURE.md, CONVENTIONS.md, BSER-QUICKREF.md)

## Prerequisites

- Kilo Code CLI installed and configured
- Git initialized in your project
- A working Kilo config at `~/.config/kilo/` or `~/.kilocode/`
- agent-browser installed: `npx skills add https://github.com/vercel-labs/agent-browser --skill agent-browser`
