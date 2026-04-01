## Test Case Categories

Every plan document organizes test cases into three categories. All three must be covered for complete test coverage.

### Happy Path
**Minimum: 2 tests**

Expected usage scenarios — what happens when things work correctly?
- Verify the code works with valid input
- Test the main user-facing functionality
- Verify successful edge cases (e.g., boundary values that are still valid)

### Error & Boundary
**Minimum: 2 tests**

What happens when things go wrong?
- Invalid input handling
- Empty data, null values, undefined values
- Out-of-range numbers or data that's too large
- Wrong type input
- Concurrent access or race conditions
- Missing dependencies
- Network failures or timeouts
- Malicious input (e.g., SQL injection, XSS)

For every function you write, ask: *What's the worst input someone could pass, and what should happen?*

### Integration
**Minimum: 1 test**

How do the changed components interact with their callers and dependencies?
- If module A calls module B and you're changing B's behavior, write a test that exercises A→B together
- Test the interaction between changed components and their dependencies
- End-to-end flows through the changed components
- Verify that callers still work correctly after the change
