# .kilocode/commands/impact.md
---
description: Analyze and visualize the dependency impact of a planned change
mode: architect
arguments:
  - plan_name
---
Analyze the dependency impact of the plan. Plans are stored at:
- `.plans/<name>.md` for regular tasks
- `.plans/epics/<epic-slug>/<name>.md` for epic phases

The agent should first check `.plans/$1.md`, and if not found, search in `.plans/epics/*/$1.md`.

For each file listed in the plan's "Files to Change" section:

1. **Import graph:** What other files import from this file? Trace 2 levels deep.
2. **Export surface:** What functions, classes, or types does this file export that others depend on?
3. **Test coverage:** Which test files cover this module? Are there integration tests?
4. **Shared module risk:** Does this touch shared/core modules? If so, flag the blast radius.

Then invoke @reporter to render an impact analysis report.

The report should include:
- A stat card row: files in plan, direct dependents count, indirect dependents count, risk level (🟢/🟡/🔴 as colored tag)
- A **mermaid dependency graph** showing: planned files (highlighted) → direct dependents → indirect dependents. Use different node colors for planned changes vs affected files.
- A "File Impact Table": for each planned file, list its exports, direct dependents, and test coverage status
- A "Risk Assessment" section with per-file risk tags and explanation
- If risk is 🔴, a "Suggested Splits" section recommending how to break the plan into smaller pieces

Save to: `.reports/impact-$1-<YYYY-MM-DD>.html`

## Examples

**Invocation:**
```
/impact add-user-authentication
```

**Expected Output:**
HTML report at `.reports/impact-<plan-name>-<YYYY-MM-DD>.html` with:
- Stat card row: files in plan, direct dependents count, indirect dependents count, risk level (🟢/🟡/🔴 as colored tag)
- Mermaid dependency graph: planned files (highlighted) → direct dependents → indirect dependents
- "File Impact Table": for each planned file, list exports, direct dependents, test coverage status
- "Risk Assessment" section with per-file risk tags
- "Suggested Splits" section (if risk is 🔴)

**HTML Report Structure:**
```html
<!-- Stat cards -->
<div class="stat-grid">
  <div class="stat-card">
    <div class="value">3</div>
    <div class="label">Files in Plan</div>
  </div>
  <div class="stat-card">
    <div class="value">7</div>
    <div class="label">Direct Dependents</div>
  </div>
  <div class="stat-card">
    <div class="value">12</div>
    <div class="label">Indirect Dependents</div>
  </div>
  <div class="stat-card">
    <span class="tag tag-yellow">🟡 MEDIUM</span>
  </div>
</div>

<!-- Mermaid dependency graph -->
<pre class="mermaid">
graph TD
  auth/service.ts --> auth/middleware.ts
  auth/service.ts --> api/auth.ts
  auth/middleware.ts --> middleware/jwt.ts
  auth/middleware.ts --> middleware/logger.ts
</pre>

<!-- File Impact Table -->
<table>
  <tr><th>File</th><th>Exports</th><th>Dependents</th><th>Test Coverage</th></tr>
  <tr><td>auth/service.ts</td><td>AuthService, TokenPayload</td><td>7 files</td><td>auth/service.spec.ts</td></tr>
</table>

<!-- Risk Assessment -->
<h2>Risk Assessment</h2>
<p><span class="tag tag-yellow">🟡 auth/service.ts</span> — 7 direct dependents, changing JWT validation</p>
```

**Common Variations:**
- `/impact` — Analyze current branch plan for risk assessment
- High risk plans show "Suggested Splits" to break into smaller pieces
```

## Chain of Thought

Follow this reasoning process for every impact analysis:

1. **READ**: Load the plan document and extract all files listed in "Files to Change". These are your blast radius anchor points.
2. **TRACE**: For each anchor file, trace imports 2 levels deep:
   - What files import FROM this file?
   - What does this file import?
   - Are any of these imports from shared/core modules?
3. **ASSESS**: For each dependent file, evaluate:
   - Is it a test file (lower risk — changes here validate behavior)?
   - Is it a leaf module (lower risk — few dependents)?
   - Is it a core/shared module (higher risk — many dependents)?
4. **CLASSIFY**: Assign risk levels per file:
   - 🟢 Low: Leaf modules, test files
   - 🟡 Medium: Modules with moderate dependents
   - 🔴 High: Core/shared modules with many dependents
5. **SYNTHESIZE**: Combine per-file risks into an overall plan risk level. A single 🔴 file elevates the whole plan to high risk.
6. **SUGGEST**: If risk is high, identify natural split points — where could the plan be divided into phases that reduce blast radius?

Keep reasoning explicit — show which step you're at as you work through it.

## Structured Output

When invoking @reporter, pass this JSON schema:

```json
{
  "report_type": "impact",
  "generated_at": "ISO8601 timestamp",
  "plan_name": "string",
  "stats": {
    "files_in_plan": "number",
    "direct_dependents": "number",
    "indirect_dependents": "number",
    "risk_level": "low | medium | high"
  },
  "dependency_graph": "mermaid graph TD format showing planned files → direct → indirect",
  "file_impact": [
    {
      "file": "string",
      "exports": ["string"],
      "direct_dependents": "number",
      "test_coverage": "string (file path or 'none')"
    }
  ],
  "risk_assessment": [
    {
      "file": "string",
      "risk_tag": "🟢 low | 🟡 medium | 🔴 high",
      "explanation": "string",
      "blast_radius": "string"
    }
  ],
  "suggested_splits": [
    {
      "phase": "string",
      "files": ["string"],
      "rationale": "string"
    }
  ]
}