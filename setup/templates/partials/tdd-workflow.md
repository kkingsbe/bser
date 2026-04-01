## TDD Workflow

Follow test-driven development for every function or component:

### 1. Happy Path First
Write the expected usage test first. Make it fail, then implement until it passes.

### 2. Error & Boundary Tests
Before moving on, write tests for failure modes:
- What happens with null input?
- What happens with empty input?
- What happens with input that's too large, wrong type, or malicious?
- What happens with missing dependencies or network failures?

A test that only checks the happy path is incomplete — write at least one test for a failure mode before considering that function done.

### 3. Integration Tests
If this component has callers or dependencies, write an integration test that exercises the full interaction chain. If module A calls this code, test A→this→downstream together. Don't just test in isolation if the plan's integration test cases require it.

### 4. Acceptance Criteria Verification
Check the plan's acceptance criteria. If an acceptance criterion can be verified with an automated test (e.g., "API returns 400 with descriptive message for malformed input"), write that test. If it can't be automated (e.g., "page loads feel fast"), note it for manual verification.