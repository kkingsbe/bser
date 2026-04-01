## Slug Derivation

### What is a Slug?
A slug is a short kebab-case identifier (2-4 words max) suitable for:
- Branch names
- Filenames for plan documents
- References in commit messages

### Slug Examples
| Task | Slug |
|------|------|
| Add user authentication with JWT | `add-user-authentication` |
| Fix CSV export crash when file is empty | `fix-csv-export-crash` |
| Refactor database connection pooling | `refactor-database-connection-pooling` |
| Add health check endpoint | `add-health-check-endpoint` |

### Branch Naming Convention

| Task Type | Branch Prefix | Example |
|-----------|---------------|---------|
| Feature | `feat/` | `feat/add-user-authentication` |
| Bugfix | `fix/` | `fix/csv-export-crash` |
| Refactor | `feat/` (or `refactor/` if your team prefers) | `feat/refactor-auth-service` |
| Epic | `epic/` | `epic/user-authentication` |

### One-Liner
A one-liner is a single sentence description of the task, used in the plan document's header and commit messages.

### Rules
1. Keep slugs short (2-4 words max) but descriptive
2. Use kebab-case (lowercase with hyphens)
3. Derive the slug from the task description's main goal
4. Use the same slug consistently across branch, plan filename, and commits
