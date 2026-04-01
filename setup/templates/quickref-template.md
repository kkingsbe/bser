```markdown
# BSER Quick Reference

**Brief → Scope → Execute → Review+Sync**

A structured methodology for human-in-the-loop agentic development with CLI coding agents.

---

## The Loop

```
BRIEF ──► SCOPE ──► EXECUTE ──► REVIEW+SYNC
   ▲                           │
   └───────────────────────────┘
```

| Phase | Duration | Your Role |
|-------|----------|-----------|
| **Brief** | ≤5 min | Read |
| **Scope** | ≤15 min | Decide |
| **Execute** | bulk of time | Steer |
| **Review+Sync** | 10-15 min | Approve |

Phases 1 and 4 are **agent-driven**. Phases 2 and 3 are **where you focus**.

---

## Slash Commands

| Command | Description |
|---------|-------------|
| `/brief` | Generate project briefing (≤5 min) |
| `/scope <description>` | Derive slug, create branch + plan |
| `/epic <description>` | Decompose large task into ordered phases |
| `/implement <slug>` | Continue building from plan |
| `/review <slug>` | Diff review against parent branch |
| `/sync` | Reconcile docs to current codebase state |
| `/hotfix <description>` | Quick fix escape hatch |
| `/recap` | End-of-session summary |
| `/impact <slug>` | Dependency impact analysis |
| `/estimate` | Planned vs actual calibration |
| `/update` | Fetch latest BSER bootstrap from GitHub |

### Common Usage

```bash
/brief                                    # Start of session - rebuild mental model
/scope Add XER file parsing support       # One sentence or it's too big
/epic Refactor authentication system     # Too big? Decompose into phases
/implement add-xer-parser                 # Continue from plan
/review add-xer-parser                    # Check against plan
# merge feat/add-xer-parser to main
/sync                                    # Reconcile docs to codebase
/recap                                   # End of session
```

---

## Subagents

| Agent | Invocation | Purpose |
|-------|-----------|---------|
| `@reviewer` | `@reviewer Review the diff...` | Read-only diff review, locked permissions |
| `@syncer` | `@syncer Sync docs to current state` | Post-merge doc reconciliation, markdown-only edits |
| `@reporter` | `@reporter Generate sprint recap...` | HTML+CSS+mermaid report generation |

---

## Rules

1. **One sentence or it's too big.** If you can't describe it in one sentence, use `/epic`.
2. **Plans are the contract.** Implement against the plan. Discovered something new? Add it to "Future" section and continue.
3. **Brief before every epic phase.** Your mental model goes stale fast during large refactors.
4. **Sync after every merge.** This is what keeps `/brief` accurate for your next session.
5. **Don't skip recap.** It takes 30 seconds and preserves session context.

---

## Escape Hatches

| Situation | Command |
|-----------|---------|
| Quick fix (<15 min) | `/hotfix <description>` |
| Exploration/spike | `git checkout -b spike/<name>` |
| Tiny fix (<5 min) | Do on main, `/sync` after |

**Never merge a spike branch.** Extract learnings into a `/scope` task.
```