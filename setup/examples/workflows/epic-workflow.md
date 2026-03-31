# Epic Workflow Example: Multi-Phase User Dashboard Revamp

This example demonstrates the **epic workflow** for a large feature that spans multiple phases. It shows how `/epic` decomposes a large task into ordered phases, and how each phase follows the normal BSER loop (Brief → Scope → Execute → Review → Sync).

**What this example teaches:**
- When to escalate from `/scope` to `/epic`
- How `/epic` creates a roadmap with ordered phases
- How each phase branches from and merges to the epic branch
- The importance of `/brief` between phases
- How the epic doc tracks overall progress

**The epic:** "Revamp the user dashboard with real-time data, new widgets, and improved performance"

---

## Recognizing You Need an Epic

During `/brief`, you assess the scope of "revamping the user dashboard":

- New real-time data layer (WebSocket integration)
- 5 new dashboard widgets (analytics, notifications, activity feed, quick actions, health metrics)
- Performance improvements (lazy loading, caching)
- UI/UX overhaul (new component library)
- Backend API changes (new endpoints, data aggregation)

**This is too big for `/scope`.** Your scoping session would exceed 15 minutes. You need to decompose.

---

## Phase 0: Create the Epic

### Command

```
/epic Revamp user dashboard with real-time data, new widgets, and improved performance
```

### Expected Output

```
[0] Naming
  Epic slug: dashboard-revamp
  One-liner: Revamp user dashboard with real-time data, new widgets, and improved performance

[1] Branch
  Creating: epic/dashboard-revamp from main
  ✓ Branch created

[2] Directory
  Creating: .plans/epics/dashboard-revamp/
  ✓ Directory created

[3] Understanding
  Reading ARCHITECTURE.md and CONVENTIONS.md...
  Identified modules: users, dashboard, widgets, api, websocket, realtime

[4] Decomposing into phases

  Phase 1: Core Data Layer
  - New dashboard API endpoints for widget data
  - WebSocket infrastructure for real-time updates
  - Depends on: nothing
  
  Phase 2: Widget Foundation
  - Base widget component (common structure for all widgets)
  - Widget registry system
  - Loading states, error boundaries
  - Depends on: phase 1 (API endpoints needed)
  
  Phase 3: Individual Widgets
  - Analytics widget (charts)
  - Notifications widget
  - Activity feed widget
  - Quick actions widget
  - Health metrics widget
  - Depends on: phase 2 (foundation needed)
  
  Phase 4: Performance & Polish
  - Lazy loading for off-screen widgets
  - Caching layer
  - Final UI polish
  - Depends on: phase 3 (all widgets built)

[5] Writing epic document...

# Epic: dashboard-revamp

**Status:** IN_PROGRESS
**Branch:** epic/dashboard-revamp
**Created:** 2026-03-31
**One-liner:** Revamp user dashboard with real-time data, new widgets, and improved performance

## Objective

Replace the legacy dashboard with a modern, real-time capable interface. Users see live data updates without page refreshes, can customize widget layout, and experience snappy performance even with slow network conditions. The new dashboard is built on a widget-based architecture that makes it easy to add new widgets in the future.

## Phases

| # | Plan | Description | Dependencies | Status |
|---|------|-------------|-------------|--------|
| 1 | `dashboard-revamp-phase-1-data-layer` | API endpoints + WebSocket infrastructure | none | PLANNING |
| 2 | `dashboard-revamp-phase-2-widget-foundation` | Base widget component + registry | phase 1 | PLANNING |
| 3 | `dashboard-revamp-phase-3-widgets` | 5 individual widgets | phase 2 | PLANNING |
| 4 | `dashboard-revamp-phase-4-performance` | Lazy loading, caching, polish | phase 3 | PLANNING |

## Architecture Impact

- **New modules:** `src/realtime/` (WebSocket handling), `src/widgets/` (widget system)
- **Modified modules:** `src/dashboard/` (replaces legacy), `src/api/` (new endpoints)
- **New dependencies:** `socket.io-client`, `@tanstack/react-query` (for caching)
- **Removed:** Legacy dashboard components (to be deleted in phase 4)

## Risks & Open Questions

- WebSocket reconnection strategy needs careful design — what if the server goes down mid-session?
- How should widget state persist across sessions? LocalStorage or server-side?
- The analytics widget requires a new charting library — need to evaluate options
- Performance targets: dashboard should render in < 2s on 3G

## Exit Criteria

- All 5 widgets render and update in real-time
- Lighthouse performance score > 85
- Dashboard loads in < 2s on simulated 3G
- No console errors during normal usage
- Legacy dashboard fully replaced and removed

---

[6] Commit
  ✓ epic: dashboard-revamp committed

[7] Invoking @reporter...
  ✓ Epic planning report generated: .reports/epic-dashboard-revamp-2026-03-31.html
```

### Epic Planning Report Contents

The `@reporter` agent generates a visual HTML report showing:

```html
<!-- Stat cards -->
<div class="stat-grid">
  <div class="stat-card"><div class="value">4</div><div class="label">Total Phases</div></div>
  <div class="stat-card"><div class="value">~15</div><div class="label">Estimated Files</div></div>
  <div class="stat-card"><div class="value">3</div><div class="label">Modules Touched</div></div>
  <div class="stat-card"><div class="value">Medium</div><div class="label">Risk Level</div></div>
</div>

<!-- Branching strategy gitgraph -->
<pre class="mermaid">
gitGraph
  commit id: "main"
  branch epic/dashboard-revamp
  commit id: "epic plan"
  branch feat/dashboard-revamp-phase-1-data-layer
  commit id: "phase 1 work"
  checkout epic/dashboard-revamp
  merge feat/dashboard-revamp-phase-1-data-layer id: "merge phase 1"
  branch feat/dashboard-revamp-phase-2-widget-foundation
  commit id: "phase 2 work"
  checkout epic/dashboard-revamp
  merge feat/dashboard-revamp-phase-2-widget-foundation id: "merge phase 2"
  branch feat/dashboard-revamp-phase-3-widgets
  commit id: "phase 3 work"
  checkout epic/dashboard-revamp
  merge feat/dashboard-revamp-phase-3-widgets id: "merge phase 3"
  branch feat/dashboard-revamp-phase-4-performance
  commit id: "phase 4 work"
  checkout epic/dashboard-revamp
  merge feat/dashboard-revamp-phase-4-performance id: "merge phase 4"
  checkout main
  merge epic/dashboard-revamp id: "epic complete"
</pre>

<!-- Phase breakdown table -->
<table>
  <tr><th>#</th><th>Plan</th><th>Description</th><th>Dependencies</th></tr>
  <tr><td>1</td><td>dashboard-revamp-phase-1-data-layer</td><td>API + WebSocket</td><td>none</td></tr>
  <tr><td>2</td><td>dashboard-revamp-phase-2-widget-foundation</td><td>Widget system</td><td>phase 1</td></tr>
  <tr><td>3</td><td>dashboard-revamp-phase-3-widgets</td><td>5 widgets</td><td>phase 2</td></tr>
  <tr><td>4</td><td>dashboard-revamp-phase-4-performance</td><td>Polish</td><td>phase 3</td></tr>
</table>
```

---

## Phase 1: Brief (Before Phase 1)

### Command

```
/brief
```

### Expected Output

Brief report shows:
- Current branch: `epic/dashboard-revamp`
- Epic status: `dashboard-revamp` — 0/4 phases complete
- Next action: Start phase 1 (`dashboard-revamp-phase-1-data-layer`)

### What You Do

Confirm you're on the epic branch. Read the epic doc to refresh your understanding. Now scope phase 1.

---

## Phase 1: Scope

### Command

```
/scope dashboard-revamp-phase-1-data-layer
```

### Expected Output

```
[0] Naming
  Phase detected: this is a phase of epic/dashboard-revamp
  Phase slug: dashboard-revamp-phase-1-data-layer

[1] Branch
  From: epic/dashboard-revamp
  Creating: feat/dashboard-revamp-phase-1-data-layer
  ✓ Branch created

[2] Test Baseline
  Running: npm test
  Result: 89 passed, 0 failed
  Saved to: .plans/epics/dashboard-revamp/dashboard-revamp-phase-1-data-layer.baseline-tests.txt
  ✓ Baseline committed

[3] Plan Location
  Saving to: .plans/epics/dashboard-revamp/dashboard-revamp-phase-1-data-layer.md

[4] Plan Created

# Plan: dashboard-revamp-phase-1-data-layer

**Status:** IN_PROGRESS
**Category:** feature
**Branch:** feat/dashboard-revamp-phase-1-data-layer
**Parent:** epic/dashboard-revamp
**One-liner:** Add dashboard API endpoints and WebSocket infrastructure for real-time updates

## Scope

Build the data layer for the new dashboard:
- New REST endpoints for widget data (analytics, notifications, activity, quick actions, health)
- WebSocket server setup for real-time push
- Initial data fetch pattern (REST on load, WebSocket for updates)

## Files to Change

- `src/api/dashboard.ts` — New API client functions
- `src/api/endpoints/dashboard.ts` — Express routes for dashboard data
- `src/realtime/websocket.ts` — WebSocket server handler
- `src/realtime/events.ts` — Event types for dashboard updates
- `src/realtime/client.ts` — Client-side WebSocket hook
- `src/config/realtime.ts` — WebSocket configuration

## Test Cases

- [ ] Dashboard API returns widget data for all 5 widgets
- [ ] WebSocket connection established on dashboard load
- [ ] Client receives real-time update when data changes
- [ ] WebSocket reconnects automatically on disconnect
- [ ] API falls back to REST when WebSocket unavailable

## Implementation Notes

- Use socket.io (already in dependencies)
- Follow existing API pattern in src/api/
- WebSocket events: 'widget:update', 'widget:refresh', 'dashboard:sync'
- Store WebSocket state in React context for easy access

## Future (Out of Scope)

- Widget-specific WebSocket logic (in phase 3)
- Widget state persistence
- Offline support

---

[5] Commit
  ✓ plan: dashboard-revamp-phase-1-data-layer committed
  ✓ epic: update dashboard-revamp phases (phase 1 → IN_PROGRESS)
```

### Key Difference from Normal Scope

Notice:
- Branch is created FROM `epic/dashboard-revamp`, not from `main`
- Plan saved to `.plans/epics/dashboard-revamp/`, not `.plans/`
- Epic doc's phases table is updated automatically

---

## Phase 1: Execute

### Command

```
/implement dashboard-revamp-phase-1-data-layer
```

### Expected Output

```
[IN_PROGRESS] dashboard-revamp-phase-1-data-layer

[1] Reading plan from .plans/epics/dashboard-revamp/dashboard-revamp-phase-1-data-layer.md
[2] Status: IN_PROGRESS
[3] Running tests...
  89 passed, 0 failed

[4] Starting TDD cycle...

--- API Layer ---
[WRITE] src/api/dashboard.ts
[IMPLEMENT] API client functions for each widget type
[COMMIT] add dashboard API client with widget data functions

[WRITE] src/api/endpoints/dashboard.spec.ts
[IMPLEMENT] Express routes for dashboard endpoints
[COMMIT] add dashboard API endpoints

--- WebSocket Layer ---
[WRITE] src/realtime/events.spec.ts
[IMPLEMENT] Event type definitions
[COMMIT] add dashboard WebSocket event types

[WRITE] src/realtime/client.spec.ts
[IMPLEMENT] useWebSocket React hook
[COMMIT] add WebSocket client hook

[WRITE] src/realtime/websocket.spec.ts
[IMPLEMENT] WebSocket server handler with auto-reconnect logic
[COMMIT] add WebSocket server with reconnection handling

[CONFIG] src/config/realtime.ts
[COMMIT] add WebSocket configuration

--- Integration ---
[WRITE] src/realtime/client.integration.spec.ts
[COMMIT] add WebSocket client integration tests

[5] Updating plan checkboxes...
  ✓ 5/5 test cases passing

[6] Running full test suite...
  Result: 97 passed, 0 failed
  All planned test cases passing
```

### Verify Commits

```bash
git log --oneline -8
```

```
a3f2b1c add WebSocket client integration tests
9d8e7f1 add WebSocket server with reconnection handling
5c4b3a2 add WebSocket client hook
2f1e9d3 add dashboard WebSocket event types
8a7b6c4 add dashboard API endpoints
f9e8d2a add dashboard API client with widget data functions
7b6c4d2 test baseline: dashboard-revamp-phase-1-data-layer
e2f1a8b epic: dashboard-revamp
```

---

## Phase 1: Review

### Command

```
/review dashboard-revamp-phase-1-data-layer
```

### Expected Output

```
[VERDICT: PASS] ✅

<div class="stat-grid">
  <div class="stat-card"><div class="value">6</div><div class="label">Files Changed</div></div>
  <div class="stat-card"><div class="value">+234 / -18</div><div class="label">Lines</div></div>
  <div class="stat-card"><div class="value">97/97</div><div class="label">Tests Passing</div></div>
  <div class="stat-card"><div class="value">PASS</div><div class="label">Verdict</div></div>
</div>

Review Findings: No issues. Implementation matches plan. All test cases passing.
```

### What You Do

Merge phase 1 to epic branch.

---

## Phase 1: Merge to Epic

### Command

```bash
git checkout epic/dashboard-revamp
git pull
git merge feat/dashboard-revamp-phase-1-data-layer
git branch -d feat/dashboard-revamp-phase-1-data-layer
```

### Expected Output

```
Already on 'epic/dashboard-revamp'
Merge made by 'ort' strategy.
 6 files changed, +234, -18
 Deleted branch feat/dashboard-revamp-phase-1-data-layer (was a3f2b1c).
```

---

## Phase 1: Sync

### Command

```
/sync dashboard-revamp-phase-1-data-layer
```

### Expected Output

```
[SYNC] dashboard-revamp-phase-1-data-layer (merged to epic/dashboard-revamp)

[1] ARCHITECTURE.md
  Added new module entry:
  - realtime: WebSocket handling, event types, client hook | src/realtime/*

[2] CONVENTIONS.md
  Added convention:
  - "Dashboard uses WebSocket for real-time updates with REST fallback"

[3] Epic README: .plans/epics/dashboard-revamp/README.md
  Phase 1 status: COMPLETE
  Completed: 2026-03-31

[4] .plans/epics/dashboard-revamp/dashboard-revamp-phase-1-data-layer.md
  Status: COMPLETE
  Completion Log filled

[5] Commit
  ✓ docs: sync after dashboard-revamp-phase-1-data-layer merge
```

---

## Brief Between Phases (Critical!)

Before starting phase 2, you MUST run `/brief`.

### Command

```
/brief
```

### Expected Output

Brief report shows:
- Current branch: `epic/dashboard-revamp`
- Epic status: `dashboard-revamp` — 1/4 phases complete
- Phase 1 changes are now on `epic/dashboard-revamp`
- Next action: Start phase 2 (`dashboard-revamp-phase-2-widget-foundation`)

### Why This Matters

**The epic branch changed.** Phase 1 was merged into it. Your local mental model of the epic branch is now stale. Brief refreshes it.

Additionally:
- Re-read the epic doc. Does phase 2 still make sense?
- Did phase 1 reveal anything that changes phase 2?
- Any new risks or questions to add?

---

## Phase 2: Scope → Execute → Review → Sync

Repeat the normal BSER loop for phase 2. The pattern is identical:

### Scope

```
/scope dashboard-revamp-phase-2-widget-foundation
```

Creates `feat/dashboard-revamp-phase-2-widget-foundation` from `epic/dashboard-revamp`, saves plan to `.plans/epics/dashboard-revamp/`.

### Execute

```
/implement dashboard-revamp-phase-2-widget-foundation
```

Builds the widget system foundation.

### Review

```
/review dashboard-revamp-phase-2-widget-foundation
```

### Merge to Epic

```bash
git checkout epic/dashboard-revamp
git merge feat/dashboard-revamp-phase-2-widget-foundation
git branch -d feat/dashboard-revamp-phase-2-widget-foundation
```

### Sync

```
/sync dashboard-revamp-phase-2-widget-foundation
```

---

## Brief Between Phases Again

Before phase 3:

```
/brief
```

Epic status: 2/4 phases complete. Next: phase 3 (individual widgets).

---

## Phase 3: Widgets (Multiple Commits)

Phase 3 is the largest phase — 5 widgets. It still follows the BSER loop, but the implementation is larger.

### Scope

```
/scope dashboard-revamp-phase-3-widgets
```

### Execute

```
/implement dashboard-revamp-phase-3-widgets
```

The agent implements all 5 widgets:

```
--- Analytics Widget ---
[WRITE] src/widgets/analytics/analytics.spec.ts
[IMPLEMENT] src/widgets/analytics/analytics.tsx
[COMMIT] add analytics widget with chart

--- Notifications Widget ---
[WRITE] src/widgets/notifications/notifications.spec.ts
[IMPLEMENT] src/widgets/notifications/notifications.tsx
[COMMIT] add notifications widget

--- Activity Feed Widget ---
[WRITE] src/widgets/activity/activity.spec.ts
[IMPLEMENT] src/widgets/activity/activity.tsx
[COMMIT] add activity feed widget

--- Quick Actions Widget ---
[WRITE] src/widgets/quick-actions/quick-actions.spec.ts
[IMPLEMENT] src/widgets/quick-actions/quick-actions.tsx
[COMMIT] add quick actions widget

--- Health Metrics Widget ---
[WRITE] src/widgets/health/health.spec.ts
[IMPLEMENT] src/widgets/health/health.tsx
[COMMIT] add health metrics widget

--- Widget Integration ---
[WRITE] src/widgets/registry.spec.ts
[IMPLEMENT] src/widgets/registry.ts
[COMMIT] add widget registry system

[5] Updating plan checkboxes...
  ✓ 12/12 test cases passing
```

### Review

```
/review dashboard-revamp-phase-3-widgets
```

### Merge to Epic

```bash
git checkout epic/dashboard-revamp
git merge feat/dashboard-revamp-phase-3-widgets
git branch -d feat/dashboard-revamp-phase-3-widgets
```

### Sync

```
/sync dashboard-revamp-phase-3-widgets
```

---

## Brief Before Final Phase

```
/brief
```

Epic status: 3/4 phases complete. Phase 4 is performance + polish. This is the final stretch.

---

## Phase 4: Performance & Polish

This phase focuses on:
- Lazy loading for off-screen widgets
- Caching layer with React Query
- Final UI polish
- Removing legacy dashboard

### Scope

```
/scope dashboard-revamp-phase-4-performance
```

### Execute

```
/implement dashboard-revamp-phase-4-performance
```

Implements:
- `React.lazy()` for each widget component
- Intersection Observer for off-screen widget loading
- React Query caching configuration
- CSS polish and animations
- Deletion of legacy dashboard files

### Review

```
/review dashboard-revamp-phase-4-performance
```

### Merge to Epic

```bash
git checkout epic/dashboard-revamp
git merge feat/dashboard-revamp-phase-4-performance
git branch -d feat/dashboard-revamp-phase-4-performance
```

### Sync

```
/sync dashboard-revamp-phase-4-performance
```

---

## Epic Completion: Merge to Main

All 4 phases are complete. Now merge the epic to main.

### Command

```bash
git checkout main
git pull
git merge epic/dashboard-revamp
git branch -d epic/dashboard-revamp
```

### Expected Output

```
Already on 'main'
Merge made by 'ort' strategy.
 24 files changed, +1847, -892
 Deleted branch epic/dashboard-revamp (was b4c2d3e).
```

### Final Sync

```
/sync dashboard-revamp
```

Since this is the epic merge (not a phase), the sync:
1. Marks the epic README as COMPLETE
2. Updates ARCHITECTURE.md with final state
3. Updates CONVENTIONS.md with any final patterns
4. Removes the epic branch tracking

---

## Epic Progress After Each Phase

The epic README at `.plans/epics/dashboard-revamp/README.md` tracks progress:

```markdown
## Progress Log

| Phase | Started | Completed | Notes |
|-------|---------|-----------|-------|
| 1 | 2026-03-31 | 2026-03-31 | API + WebSocket infrastructure complete |
| 2 | 2026-04-01 | 2026-04-01 | Widget foundation complete |
| 3 | 2026-04-02 | 2026-04-03 | All 5 widgets implemented |
| 4 | 2026-04-04 | 2026-04-04 | Performance optimizations complete |
```

---

## Summary: Epic Workflow

| Step | Command | Key Difference from Normal |
|------|---------|---------------------------|
| Create Epic | `/epic <description>` | Creates `epic/<slug>` branch and roadmap |
| Brief | `/brief` | Must refresh mental model before each phase |
| Scope Phase N | `/scope <epic>-phase-N-<desc>` | Branches from `epic/<slug>`, saves to epic directory |
| Implement | `/implement <phase-slug>` | Same as normal |
| Review | `/review <phase-slug>` | Same as normal |
| Merge to Epic | `git checkout epic/<slug> && git merge feat/<phase-slug>` | Phase merges to epic, not to main |
| Sync | `/sync <phase-slug>` | Updates epic README phases table |
| Brief | `/brief` | CRITICAL: Refresh before next phase |
| Repeat | For each phase... | Until all phases complete |
| Final Merge | `git checkout main && git merge epic/<slug>` | Epic merges to main only when complete |
| Final Sync | `/sync <epic-slug>` | Marks epic as COMPLETE |

---

## Templates Referenced

- `setup/templates/epic-template.md` — Epic document structure
- `setup/templates/plan-template.md` — Phase plan structure
- `setup/commands/epic.md` — Epic command definition
- `setup/commands/scope.md` — Scope command (with epic awareness)
- `setup/commands/brief.md` — Brief command (epic status tracking)
- `setup/commands/implement.md` — Implement command
- `setup/commands/review.md` — Review command
- `setup/commands/sync.md` — Sync command (with epic phase table updates)
- `setup/agents/reporter.md` — Report generation (epic planning report)

## Key Takeaways

1. **Epics are for big tasks.** If scoping takes > 15 minutes, consider an epic.
2. **One phase at a time.** Don't start phase 2 while phase 1 is unmerged.
3. **Brief between every phase.** Your mental model goes stale. The epic branch changed.
4. **Phases merge to epic, not main.** Main only sees the finished epic.
5. **The epic doc is a living roadmap.** Update it as you learn things.
6. **Exit criteria matter.** Define how you know the epic is done before starting.
