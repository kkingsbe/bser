---
description: Fetch latest BSER framework, detect local modifications, backup project-specific files, and perform full framework update
mode: code
---

# /update

Fetch the latest BSER bootstrap.md from GitHub, detect local modifications, backup project-specific content, and perform a full framework update.

## Process

### 1. FETCH LATEST BOOTSTRAP.MD

Use the webfetch tool to retrieve the current bootstrap.md from GitHub:

```
URL: https://github.com/kkingsbe/bser/blob/main/bootstrap.md
Format: markdown
```

Extract the version string from the fetched content. Look for a line like `version: <semver>` in the frontmatter or a `## Version X.Y.Z` header.

### 2. READ CURRENT VERSION

Check if `.bser-version` exists. If it does, read its contents to determine the currently installed version. Display the current version to the user.

```
[UPDATE] Current BSER version: <current-version>
[UPDATE] Latest BSER version: <latest-version>
```

### 3. DETECT LOCAL MODIFICATIONS

Before proceeding with any file operations, compare local BSER-managed files against their canonical sources in `setup/commands/`, `setup/agents/`, and `setup/templates/`.

**Commands comparison:**
```bash
# Compare each command file
diff .kilocode/commands/brief.md .bser/setup/commands/brief.md
diff .kilocode/commands/scope.md .bser/setup/commands/scope.md
diff .kilocode/commands/epic.md .bser/setup/commands/epic.md
diff .kilocode/commands/implement.md .bser/setup/commands/implement.md
diff .kilocode/commands/review.md .bser/setup/commands/review.md
diff .kilocode/commands/sync.md .bser/setup/commands/sync.md
diff .kilocode/commands/hotfix.md .bser/setup/commands/hotfix.md
diff .kilocode/commands/recap.md .bser/setup/commands/recap.md
diff .kilocode/commands/impact.md .bser/setup/commands/impact.md
diff .kilocode/commands/estimate.md .bser/setup/commands/estimate.md
```

**Agents comparison:**
```bash
diff .kilo/agents/reviewer.md .bser/setup/agents/reviewer.md
diff .kilo/agents/syncer.md .bser/setup/agents/syncer.md
diff .kilo/agents/reporter.md .bser/setup/agents/reporter.md
```

**Templates comparison:**
```bash
diff AGENTS.md .bser/setup/templates/AGENTS.md
diff ARCHITECTURE.md .bser/setup/templates/ARCHITECTURE.md
diff CONVENTIONS.md .bser/setup/templates/CONVENTIONS.md
```

If ANY differences are found, **STOP IMMEDIATELY** and display:

```
[UPDATE] Local modifications detected to the following files:
<list of modified files, one per line with leading dash>
[UPDATE] Cannot update — backup your changes and revert modified files before running /update again.
[UPDATE] To backup: cp <file> .plans/<file>.backup
[UPDATE] To revert: git checkout -- <file>
```

Do NOT proceed further. The user must handle their modifications manually.

### 4. BACKUP PROJECT-SPECIFIC FILES

Create `.plans/` directory if it doesn't exist and backup current project-specific files:

```bash
mkdir -p .plans
cp AGENTS.md .plans/AGENTS.md.backup
cp ARCHITECTURE.md .plans/ARCHITECTURE.md.backup
cp CONVENTIONS.md .plans/CONVENTIONS.md.backup
```

### 5. FULL REPLACEMENT OF ALL BSER FILES

Perform a complete replacement of all BSER framework files:

```bash
# Commands - full replacement
cp .bser/setup/commands/*.md .kilocode/commands/

# Agents - full replacement
cp .bser/setup/agents/*.md .kilo/agents/

# Templates - to their correct destinations
cp .bser/setup/templates/AGENTS.md .
cp .bser/setup/templates/ARCHITECTURE.md .
cp .bser/setup/templates/CONVENTIONS.md .
cp .bser/setup/templates/backlog-template.md .plans/backlog.md
cp .bser/setup/templates/plan-template.md .plans/_TEMPLATE.md
cp .bser/setup/templates/epic-template.md .plans/_EPIC_TEMPLATE.md
cp .bser/setup/templates/commands-registry-template.md .kilocode/commands/commands.md
cp .bser/setup/templates/quickref-template.md BSER-QUICKREF.md
```

### 6. FILL IN TEMPLATED SECTIONS

Read each `.backup` file from `.plans/` and extract project-specific information. Update the corresponding new files with extracted values.

**Extract from AGENTS.md.backup:**
- Project name (look for `project:` or `# <project-name>`)
- Repository URL (look for `repository:` or GitHub URL patterns)
- Custom instructions or workflow rules

**Extract from ARCHITECTURE.md.backup:**
- Project architecture overview
- Directory structure decisions
- Key technical choices

**Extract from CONVENTIONS.md.backup:**
- Code style conventions
- Naming conventions
- Git conventions (commit message format, branch naming)

**Update targets:**
- `<project-name>` → extracted project name
- `<repository-url>` → extracted repository URL
- `<custom-conventions>` → extracted convention details
- `<workflow-rules>` → extracted workflow rules

Use the edit tool to replace placeholder values in the newly copied files with the extracted project-specific content.

### 7. UPDATE VERSION FILE

Update `.bser-version` with the new version. If user confirms or if a newer version was fetched:

```
<version>|<YYYY-MM-DD>
```

Where version is from the fetched bootstrap.md and date is the current date.

### 8. DISPLAY COMPLETION SUMMARY

```
[UPDATE] ======================================
[UPDATE] BSER Update Complete
[UPDATE] ======================================
[UPDATE] Previous version: <old-version>
[UPDATE] New version: <new-version>
[UPDATE]
[UPDATE] Files updated:
[UPDATE]   - .kilocode/commands/ (all command files)
[UPDATE]   - .kilo/agents/ (all agent files)
[UPDATE]   - AGENTS.md
[UPDATE]   - ARCHITECTURE.md
[UPDATE]   - CONVENTIONS.md
[UPDATE]   - .plans/backlog.md
[UPDATE]   - .plans/_TEMPLATE.md
[UPDATE]   - .plans/_EPIC_TEMPLATE.md
[UPDATE]   - .kilocode/commands/commands.md
[UPDATE]   - BSER-QUICKREF.md
[UPDATE]
[UPDATE] Backups saved to .plans/:
[UPDATE]   - AGENTS.md.backup
[UPDATE]   - ARCHITECTURE.md.backup
[UPDATE]   - CONVENTIONS.md.backup
[UPDATE]
[UPDATE] Review the updated files and restore any project-specific customizations.
```

## Examples

**Invocation:**
```
/update
```

---

**Example — Local modifications detected (ERROR):**

```
[UPDATE] Fetching latest BSER bootstrap from GitHub...
[UPDATE] Current BSER version: 1.2.0
[UPDATE] Latest BSER version: 1.3.0
[UPDATE] Checking for local modifications...
[UPDATE]
[UPDATE] Local modifications detected to the following files:
- .kilocode/commands/scope.md
- .kilocode/commands/implement.md
- AGENTS.md
[UPDATE] Cannot update — backup your changes and revert modified files before running /update again.
[UPDATE]
[UPDATE] To backup: cp <file> .plans/<file>.backup
[UPDATE] To revert: git checkout -- <file>
[UPDATE]
[UPDATE] Aborted. Resolve modifications and run /update again.
```

---

**Example — Successful update (no version change):**

```
[UPDATE] Fetching latest BSER bootstrap from GitHub...
[UPDATE] Current BSER version: 1.3.0
[UPDATE] Latest BSER version: 1.3.0
[UPDATE] Checking for local modifications...
[UPDATE] No local modifications detected.
[UPDATE] Backing up project-specific files to .plans/...
[UPDATE] Performing full framework update...
[UPDATE] Filling in project-specific content...
[UPDATE] Version file already current.
[UPDATE]
[UPDATE] ======================================
[UPDATE] BSER Update Complete
[UPDATE] ======================================
[UPDATE] Previous version: 1.3.0
[UPDATE] New version: 1.3.0
[UPDATE]
[UPDATE] Files updated:
[UPDATE]   - .kilocode/commands/ (all command files)
[UPDATE]   - .kilo/agents/ (all agent files)
[UPDATE]   - AGENTS.md
[UPDATE]   - ARCHITECTURE.md
[UPDATE]   - CONVENTIONS.md
[UPDATE]   - .plans/backlog.md
[UPDATE]   - .plans/_TEMPLATE.md
[UPDATE]   - .plans/_EPIC_TEMPLATE.md
[UPDATE]   - .kilocode/commands/commands.md
[UPDATE]   - BSER-QUICKREF.md
[UPDATE]
[UPDATE] Backups saved to .plans/:
[UPDATE]   - AGENTS.md.backup
[UPDATE]   - ARCHITECTURE.md.backup
[UPDATE]   - CONVENTIONS.md.backup
[UPDATE]
[UPDATE] You are running the latest version. No framework changes applied.
```

---

**Example — Successful update with version upgrade:**

```
[UPDATE] Fetching latest BSER bootstrap from GitHub...
[UPDATE] Current BSER version: 1.2.0
[UPDATE] Latest BSER version: 1.3.0
[UPDATE] Checking for local modifications...
[UPDATE] No local modifications detected.
[UPDATE] Backing up project-specific files to .plans/...
[UPDATE] Performing full framework update...
[UPDATE] Filling in project-specific content...
[UPDATE]   - Project name: my-awesome-project
[UPDATE]   - Repository: https://github.com/user/my-awesome-project
[UPDATE]   - Extracted custom conventions from backup
[UPDATE] Updating .bser-version to 1.3.0...
[UPDATE]
[UPDATE] ======================================
[UPDATE] BSER Update Complete
[UPDATE] ======================================
[UPDATE] Previous version: 1.2.0
[UPDATE] New version: 1.3.0
[UPDATE]
[UPDATE] Files updated:
[UPDATE]   - .kilocode/commands/ (all command files)
[UPDATE]   - .kilo/agents/ (all agent files)
[UPDATE]   - AGENTS.md
[UPDATE]   - ARCHITECTURE.md
[UPDATE]   - CONVENTIONS.md
[UPDATE]   - .plans/backlog.md
[UPDATE]   - .plans/_TEMPLATE.md
[UPDATE]   - .plans/_EPIC_TEMPLATE.md
[UPDATE]   - .kilocode/commands/commands.md
[UPDATE]   - BSER-QUICKREF.md
[UPDATE]
[UPDATE] Backups saved to .plans/:
[UPDATE]   - AGENTS.md.backup
[UPDATE]   - ARCHITECTURE.md.backup
[UPDATE]   - CONVENTIONS.md.backup
[UPDATE]
[UPDATE] New in version 1.3.0:
[UPDATE]   - Added /estimate command for effort estimation
[UPDATE]   - Updated /review with new verification checklist
[UPDATE]   - Improved error messages throughout
[UPDATE]
[UPDATE] Review the updated files and restore any project-specific customizations.
```

## Common Variations

- `/update` — Standard framework update with modification detection
- `/update --force` — Skip modification detection (use with caution; will overwrite local changes)
- After major BSER announcements — check if new features are available
- When onboarding to a new project — verify the project has the latest BSER version

## Requirements

- Git installed and available in PATH
- Write permissions to project root and `.plans/` directory
- Network access to fetch latest bootstrap from GitHub
