# Commands Registry

## Available Commands

| Command | Description |
|---------|-------------|
| `/brief` | Generate a project briefing |
| `/scope` | Create a scoped implementation plan |
| `/epic` | Decompose a large task into phases |
| `/implement` | Continue implementing a plan |
| `/review` | Review the current branch diff |
| `/sync` | Reconcile docs to current codebase state |
| `/hotfix` | Quick fix escape hatch |
| `/recap` | End-of-session summary |
| `/impact` | Dependency impact analysis |
| `/estimate` | Planned vs actual calibration |

## Adding Commands

When you add a new command to `.kilocode/commands/<name>.md`:
1. Add an entry to the table above with the command name and description
2. Run `/sync` to ensure this file is updated
```