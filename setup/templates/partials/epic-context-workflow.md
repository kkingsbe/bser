## Epic Context Workflow

When implementing a phase that belongs to an epic (Parent: epic/<epic-slug>), follow this workflow to capture and preserve context.

### Before Starting Phase Work
1. Read the epic's plan doc at `.plans/epics/<epic-slug>/README.md`
2. Check the `## Context & Learnings` section for:
   - Relevant discoveries from previous phases
   - Open questions that may affect this phase
   - Assumptions that have been validated or challenged
3. Note any context that applies to the phase you're about to scope

### During Implementation
When you make discoveries, assumptions, realizations, or approach changes:
- Add relevant items to the epic's `## Context & Learnings` section (not to the phase's own notes)
- Keep detailed notes in your phase's Implementation Notes section

### At Phase Completion
Before marking the phase complete:
1. Review your work for any:
   - Discoveries about how the system works
   - Assumptions you made (validated or challenged)
   - Realizations that changed your approach
   - Questions that got answered
2. Extract relevant context to the epic's `## Context & Learnings` section
3. In the Completion Log's "Impact" field, reference any context you added (e.g., "Added discovery about X to epic context")
