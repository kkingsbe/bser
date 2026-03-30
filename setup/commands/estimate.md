# .kilocode/commands/estimate.md
---
description: Analyze estimation accuracy across completed plans and generate a calibration report
mode: architect
---
Analyze completed plans to identify estimation patterns.

1. **Scan `.plans/` for all COMPLETE plans** that have a filled-in Completion Log.

2. **For each completed plan, extract:**
   - Category (feature, bugfix, refactor, chore)
   - Number of files planned vs. actually changed
   - Planned test cases vs. actual test count
   - Whether implementation deviated from plan (and how)
   - Time from plan creation to completion (from git dates)

3. **Identify patterns — group by category:**
   - Do plans consistently underestimate file count? By how much, per category?
   - Which categories deviate most from their plans?
   - Common categories of unplanned work?
   - Typical ratio of planned scope to actual scope, per category?
   - Are refactors consistently underestimated? Do bugfixes take longer than features of similar size?

Then invoke @reporter to render an estimation calibration report.

The report should include:
- A stat card row: total completed plans, average scope accuracy %, average deviation, most common surprise category
- A "Plan Accuracy" table: plan name, category (as colored tag), planned files vs actual, planned tests vs actual, deviated (yes/no), duration
- A mermaid pie chart showing plan distribution by category
- A category breakdown: for each category, show average accuracy, average duration, and common deviation patterns
- A "Patterns" section with specific calibration insights
- A "Calibration Advice" section with concrete rules of thumb for future scoping

Save to: `.reports/estimate-<YYYY-MM-DD>.html`

## Examples

**Invocation:**
```
/estimate
```

**Expected Output:**
HTML report at `.reports/estimate-<YYYY-MM-DD>.html` with:
- Stat card row: total completed plans, average scope accuracy %, average deviation, most common surprise category
- "Plan Accuracy" table: plan name, category (as colored tag), planned files vs actual, planned tests vs actual, deviated (yes/no), duration
- Mermaid pie chart showing plan distribution by category
- Category breakdown: for each category, show average accuracy, average duration, and common deviation patterns
- "Patterns" section with specific calibration insights
- "Calibration Advice" section with concrete rules of thumb for future scoping

**HTML Report Structure:**
```html
<!-- Stat cards -->
<div class="stat-grid">
  <div class="stat-card">
    <div class="value">15</div>
    <div class="label">Completed Plans</div>
  </div>
  <div class="stat-card">
    <div class="value">78%</div>
    <div class="label">Avg Scope Accuracy</div>
  </div>
  <div class="stat-card">
    <div class="value">+2.3</div>
    <div class="label">Avg Files Deviation</div>
  </div>
  <div class="stat-card">
    <span class="tag tag-red">🔴 refactor</span>
    <div class="label">Most Surprising</div>
  </div>
</div>

<!-- Plan Accuracy Table -->
<table>
  <tr><th>Plan</th><th>Category</th><th>Planned</th><th>Actual</th><th>Deviated</th></tr>
  <tr><td>add-user-authentication</td><td><span class="tag tag-green">feature</span></td><td>3 files</td><td>4 files</td><td>yes</td></tr>
  <tr><td>fix-csv-export-crash</td><td><span class="tag tag-blue">bugfix</span></td><td>1 file</td><td>1 file</td><td>no</td></tr>
</table>

<!-- Category breakdown -->
<h2>Category Breakdown</h2>
<p><strong>feature:</strong> 82% avg accuracy, 3.2 days avg duration</p>
<p><strong>bugfix:</strong> 91% avg accuracy, 1.1 days avg duration</p>
<p><strong>refactor:</strong> 65% avg accuracy, 5.8 days avg duration — commonly underestimated</p>

<!-- Calibration Advice -->
<h2>Calibration Advice</h2>
<ul>
  <li>Refactors take 1.5x longer than features of similar initial scope</li>
  <li>Add 1-2 buffer files to every feature plan for integration work</li>
  <li>Bugfixes are usually 1 file, but always check for test coverage gaps</li>
</ul>
```

**Common Variations:**
- `/estimate` — Full calibration report across all completed plans
- Works best when you have 5+ completed plans with filled-in Completion Logs
```

## Chain of Thought

Follow this reasoning process for every calibration analysis:

1. **GATHER**: Scan `.plans/` for all COMPLETE plans. Read the Completion Log from each — this is where the actual vs planned data lives. Skip plans that don't have the Completion Log filled in.
2. **EXTRACT**: For each completed plan, pull:
   - Category (feature, bugfix, refactor, chore)
   - Files planned (from "Files to Change") vs files actually changed (from git)
   - Test cases planned vs tests actually written
   - Deviation notes from the Completion Log
   - Duration (from plan creation date to completion date)
3. **CALCULATE**: Compute per-plan accuracy:
   - File accuracy = actual_files / planned_files (1.0 = perfect)
   - Scope drift = actual_files - planned_files
   - Group by category to see patterns
4. **IDENTIFY**: Look for patterns across categories:
   - Which categories are most underestimated (high actual vs planned)?
   - What types of work tend to expand beyond the original scope?
   - Are there consistent ratios (e.g., refactors always need 1.5x the planned files)?
5. **SYNTHESIZE**: Combine patterns into actionable calibration advice — concrete rules of thumb for future scoping.
6. **VALIDATE**: Check that the advice makes sense against your own experience. If numbers contradict intuition, dig deeper — the data might be sparse or there might be outliers skewing it.

Keep reasoning explicit — show which step you're at as you work through it.

## Structured Output

When invoking @reporter, pass this JSON schema:

```json
{
  "report_type": "estimate",
  "generated_at": "ISO8601 timestamp",
  "stats": {
    "total_completed_plans": "number",
    "avg_scope_accuracy_percent": "number",
    "avg_files_deviation": "number",
    "most_common_surprise_category": "feature | bugfix | refactor | chore"
  },
  "plan_accuracy_table": [
    {
      "plan_name": "string",
      "category": "feature | bugfix | refactor | chore",
      "planned_files": "number",
      "actual_files": "number",
      "planned_tests": "number",
      "actual_tests": "number",
      "deviated": "boolean",
      "duration_days": "number"
    }
  ],
  "category_distribution": {
    "feature": "number",
    "bugfix": "number",
    "refactor": "number",
    "chore": "number"
  },
  "category_breakdown": [
    {
      "category": "string",
      "avg_accuracy_percent": "number",
      "avg_duration_days": "number",
      "common_deviation_pattern": "string"
    }
  ],
  "patterns": ["string"],
  "calibration_advice": ["string"]
}
```