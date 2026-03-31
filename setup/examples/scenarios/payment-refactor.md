# Scenario: Refactor Payment Processing Module

This scenario walks through a complex refactor of a payment processing module in a Node.js/e-commerce application. The refactor touches multiple services, requires data migration, and involves a "migrate-in-parallel" pattern to maintain backward compatibility during the transition.

## Project Context

**Tech Stack:** Node.js, Express, MongoDB, TypeScript
**Current State:** Payment processing is handled by a monolithic `PaymentService` class that directly integrates with Stripe and PayPal. The code is 3 years old and has accumulated technical debt.
**Risk Profile:** HIGH — This is a revenue-critical path. Mistakes could result in failed transactions or incorrect charges.

---

## Starting the Epic

The engineering team has identified payment processing as a priority refactor. The existing code:
- Directly couples payment providers (Stripe, PayPal) with business logic
- Has no interface abstractions, making provider swaps difficult
- Lacks proper error handling and retry logic
- Uses outdated callback patterns instead of async/await

The tech lead decides this warrants an epic due to the migration risk.

### Invoking `/epic`

```
/epic Refactor payment processing module to use provider interfaces, add retry logic, and improve error handling
```

The agent decomposes this, noting the high-risk nature:

---

**Epic: payment-refactor**

```markdown
# Epic: payment-refactor

**Status:** IN_PROGRESS
**Branch:** epic/payment-refactor
**Created:** 2024-02-01
**One-liner:** Refactor payment processing to use provider interfaces with improved error handling and retry logic

## Objective

Replace the monolithic `PaymentService` with a clean architecture using:
1. Provider interfaces (abstraction layer)
2. Individual provider implementations (Stripe, PayPal)
3. Retry logic with exponential backoff
4. Unified error handling
5. Transaction logging for auditing

The refactored system must maintain backward compatibility during migration—no breaking changes to API consumers.

## Phases

| # | Plan | Description | Dependencies | Status |
|---|------|-------------|-------------|--------|
| 1 | `payment-refactor-phase-1-interfaces` | Define provider interface and data models | none | PLANNING |
| 2 | `payment-refactor-phase-2-stripe` | Implement Stripe provider using new interface | phase 1 | PLANNING |
| 3 | `payment-refactor-phase-3-paypal` | Implement PayPal provider using new interface | phase 1 | PLANNING |
| 4 | `payment-refactor-phase-4-router` | Create PaymentRouter that routes to providers | phase 2, 3 | PLANNING |
| 5 | `payment-refactor-phase-5-migration` | Run both old and new in parallel, validate | phase 4 | PLANNING |
| 6 | `payment-refactor-phase-6-cutover` | Switch default to new implementation, remove old | phase 5 | PLANNING |

## Architecture Impact

**Before:**
```
PaymentService (monolith)
├── Stripe integration (embedded)
├── PayPal integration (embedded)
└── Direct DB calls
```

**After:**
```
PaymentController
└── PaymentRouter
    ├── StripeProvider (implements PaymentProvider)
    └── PayPalProvider (implements PaymentProvider)
```

Modules affected: `src/payments/`, `src/models/Payment.ts`, `src/services/PaymentService.ts`

## Risks & Open Questions

- ⚠️ HIGH: Must maintain backward compatibility during migration
- ⚠️ HIGH: Payment data migration may be needed
- Need to validate new implementation against old with same inputs
- Should we keep the old implementation as fallback during phase 5?
- Transaction rollback behavior differs between providers—how to unify?

## Exit Criteria

- All payment flows work identically with new implementation
- Old PaymentService is removed (code deleted, not commented)
- All tests pass (including new integration tests with payment provider mocks)
- No performance regression in payment processing latency
- Migration validated with 100% parity on test transactions
```

---

## Phase 1: Interfaces and Data Models

### Invoking `/scope`

```
/scope Define payment provider interface and unified data models
```

**Plan: payment-refactor-phase-1-interfaces**

```markdown
# Plan: payment-refactor-phase-1-interfaces

**Status:** IN_PROGRESS
**Category:** refactor
**Branch:** feat/payment-refactor-phase-1-interfaces
**Parent:** epic/payment-refactor
**Created:** 2024-02-01
**One-liner:** Define PaymentProvider interface and unified data models

## Scope

Create the abstraction layer that defines how payment providers interact with the system. This is purely interface definition—no implementation yet.

## Files to Change

- `src/payments/interfaces/PaymentProvider.ts` — Provider interface
- `src/payments/interfaces/index.ts` — Export all interfaces
- `src/payments/models/PaymentMethod.ts` — Unified payment method model
- `src/payments/models/PaymentResult.ts` — Unified result model
- `src/payments/models/PaymentError.ts` — Unified error model
- `tests/unit/payment-provider-interface.spec.ts` — Interface contract tests

## Test Cases

- [ ] Interface defines charge(), refund(), getStatus() methods
- [ ] PaymentResult contains all common fields (id, status, amount, provider)
- [ ] PaymentError has provider, code, message, and isRetryable fields
- [ ] PaymentMethod supports card and PayPal account types

## Implementation Notes

The interface should look like:

```typescript
interface PaymentProvider {
  readonly providerName: 'stripe' | 'paypal';
  
  charge(request: ChargeRequest): Promise<PaymentResult>;
  refund(transactionId: string, amount?: number): Promise<PaymentResult>;
  getStatus(transactionId: string): Promise<PaymentStatus>;
}
```

Keep this focused—implementation details belong in later phases.
```

---

### Implementation

```
/implement payment-refactor-phase-1-interfaces
```

**Commits made:**
- `payments: define PaymentProvider interface`
- `payments: add unified PaymentMethod, PaymentResult, PaymentError models`
- `test: add interface contract tests`

---

### Review

```
/review payment-refactor-phase-1-interfaces
```

**Review Report:**
```
[VERDICT: PASS] ✅

- 6 files changed
- +189 -0 lines
- 4/4 tests passing
- Clean interface definitions
- No backward compatibility concerns (pure addition)
```

Phase 1 merges into epic.

---

## Phase 2: Stripe Provider Implementation

### Invoking `/scope`

```
/scope Implement Stripe payment provider using the new interface
```

**Plan: payment-refactor-phase-2-stripe**

```markdown
# Plan: payment-refactor-phase-2-stripe

**Status:** IN_PROGRESS
**Category:** refactor
**Branch:** feat/payment-refactor-phase-2-stripe
**Parent:** epic/payment-refactor
**Created:** 2024-02-02
**One-liner:** Implement StripeProvider class implementing PaymentProvider interface

## Scope

Implement the Stripe-specific provider. This class translates between our internal models and Stripe's API. It does NOT include retry logic—that belongs in the router layer.

## Files to Change

- `src/payments/providers/StripeProvider.ts` — Stripe implementation
- `src/payments/providers/index.ts` — Export providers
- `tests/unit/stripe-provider.spec.ts` — Unit tests with Stripe mock
- `tests/mocks/stripe-mock.ts` — Stripe API mock

## Test Cases

- [ ] StripeProvider.charge calls Stripe API with correct params
- [ ] StripeProvider.charge maps Stripe response to PaymentResult
- [ ] StripeProvider.charge throws PaymentError on Stripe failure
- [ ] StripeProvider.refund calls refund endpoint
- [ ] StripeProvider.getStatus retrieves charge status
- [ ] Provider name is 'stripe'

## Implementation Notes

- Use stripe Node.js library
- Use environment variable STRIPE_SECRET_KEY
- Map Stripe error codes to our PaymentError.isRetryable
- Card errors: insufficient_funds (retryable), card_declined (not retryable)
```

---

### Implementation

```
/implement payment-refactor-phase-2-stripe
```

The agent writes the Stripe provider with proper error mapping and commits.

---

### Review

```
/review payment-refactor-phase-2-stripe
```

Phase 2 merges.

---

## Phase 3: PayPal Provider Implementation

### Invoking `/scope`

```
/scope Implement PayPal payment provider using the new interface
```

**Plan: payment-refactor-phase-3-paypal**

Similar structure to Phase 2 but for PayPal.

Phase 3 merges after review.

---

## Phase 4: Payment Router

### Invoking `/scope`

```
/scope Create PaymentRouter that routes payment requests to the appropriate provider
```

**Plan: payment-refactor-phase-4-router**

```markdown
# Plan: payment-refactor-phase-4-router

**Status:** IN_PROGRESS
**Category:** refactor
**Branch:** feat/payment-refactor-phase-4-router
**Parent:** epic/payment-refactor
**Created:** 2024-02-05
**One-liner:** Create PaymentRouter with provider selection and retry logic

## Scope

Create the router that:
1. Selects the appropriate provider based on payment method
2. Wraps calls with retry logic (exponential backoff)
3. Handles provider failures gracefully
4. Logs all payment operations

This is the "facade" that existing code will call.

## Files to Change

- `src/payments/PaymentRouter.ts` — Main router class
- `src/payments/RetryHandler.ts` — Retry logic
- `src/payments/PaymentLogger.ts` — Audit logging
- `tests/unit/payment-router.spec.ts` — Router unit tests
- `tests/unit/retry-handler.spec.ts` — Retry logic tests

## Test Cases

- [ ] Router.charge routes to Stripe for card payments
- [ ] Router.charge routes to PayPal for PayPal accounts
- [ ] Router retries on retryable errors (up to 3 times)
- [ ] Router does not retry on non-retryable errors
- [ ] Router logs all payment attempts
- [ ] Router throws after all retries exhausted

## Implementation Notes

- Retry config: max 3 attempts, exponential backoff (1s, 2s, 4s)
- Only retry on network errors and retryable payment errors
- Log every attempt with attempt number and outcome
```

---

### Implementation

```
/implement payment-refactor-phase-4-router
```

**Commits made:**
- `payments: add RetryHandler with exponential backoff`
- `payments: add PaymentLogger for audit trail`
- `payments: implement PaymentRouter with provider selection`
- `test: add retry handler unit tests`
- `test: add payment router unit tests`

---

### Review

```
/review payment-refactor-phase-4-router
```

Phase 4 merges.

---

## Phase 5: Parallel Run (Migration Pattern)

This is the critical migration phase. Both old and new implementations run in parallel to validate correctness.

### Invoking `/scope`

```
/scope Set up parallel execution mode to validate new payment implementation against production behavior
```

**Plan: payment-refactor-phase-5-migration**

```markdown
# Plan: payment-refactor-phase-5-migration

**Status:** IN_PROGRESS
**Category:** refactor
**Branch:** feat/payment-refactor-phase-5-migration
**Parent:** epic/payment-refactor
**Created:** 2024-02-07
**One-liner:** Run new implementation alongside old, validate parity

## Scope

Set up the system to run both payment implementations simultaneously in "shadow mode":
1. New implementation is called but results are not returned to callers
2. Results are compared internally
3. Discrepancies are logged for review
4. After validation period, proceed to cutover

This is the risk mitigation phase.

## Files to Change

- `src/payments/PaymentMigration.ts` — Shadow mode coordinator
- `src/config/payment.ts` — Feature flag for migration mode
- `tests/integration/migration-parity.spec.ts` — Parity validation tests

## Test Cases

- [ ] Shadow mode logs both results without returning new result
- [ ] Parity check detects differences in amount
- [ ] Parity check detects differences in status
- [ ] Feature flag enables/disables shadow mode
- [ ] Fallback to old implementation if new fails

## Migration Validation Checklist

Before moving to Phase 6, verify:
- [ ] 100% parity on 100+ test transactions
- [ ] No discrepancies in error handling
- [ ] Latency acceptable (<500ms p95)
- [ ] All edge cases tested (declined, insufficient funds, network errors)

## Implementation Notes

- Use PAYMENT_MIGRATION_MODE=true environment variable
- In migration mode, old implementation handles the response
- New implementation runs in background for comparison
- Discrepancies logged as CRITICAL for immediate review
```

---

### Implementation

```
/implement payment-refactor-phase-5-migration
```

**Commits made:**
- `payments: add PaymentMigration shadow mode coordinator`
- `config: add PAYMENT_MIGRATION_MODE feature flag`
- `test: add migration parity validation tests`

---

### The Migration Period

During this phase, the team runs the new implementation alongside the old in production with real (non-production volume) traffic. The shadow mode logs any differences.

**After 3 days of validation:**

```
Migration Report - 2024-02-10

Transactions processed: 147
Parity achieved: 147/147 (100%)
Discrepancies: 0

Latency comparison:
- Old implementation: 245ms average
- New implementation: 198ms average
- Improvement: 19%

Conclusion: New implementation is ready for cutover.
```

---

### Review

```
/review payment-refactor-phase-5-migration
```

Phase 5 merges.

---

## Phase 6: Cutover

### Invoking `/scope`

```
/scope Complete cutover to new payment implementation and remove legacy PaymentService
```

**Plan: payment-refactor-phase-6-cutover**

```markdown
# Plan: payment-refactor-phase-6-cutover

**Status:** IN_PROGRESS
**Category:** refactor
**Branch:** feat/payment-refactor-phase-6-cutover
**Parent:** epic/payment-refactor
**Created:** 2024-02-10
**One-liner:** Switch default to new implementation and remove old PaymentService

## Scope

Complete the migration:
1. Update PaymentController to use new PaymentRouter by default
2. Remove the legacy PaymentService (code deletion)
3. Clean up migration feature flags
4. Update ARCHITECTURE.md with new structure
5. Run full test suite

## Files to Change

- `src/payments/PaymentController.ts` — Update to use router
- `src/services/PaymentService.ts` — DELETE THIS FILE
- `src/config/payment.ts` — Remove migration flags
- `ARCHITECTURE.md` — Update payment module docs
- `tests/integration/payment-full.spec.ts` — Final integration tests

## Test Cases

- [ ] PaymentController uses PaymentRouter
- [ ] Legacy PaymentService is deleted
- [ ] No references to PaymentService remain
- [ ] Full integration tests pass
- [ ] ARCHITECTURE.md reflects new structure

## Cutover Steps

1. Deploy with PAYMENT_USE_NEW_ROUTER=true (but old still active)
2. Run smoke tests on production
3. Switch PAYMENT_USE_NEW_ROUTER=true, PAYMENT_MIGRATION_MODE=false
4. Deploy and monitor
5. Delete PaymentService.ts
6. Commit cleanup

## Rollback Plan

If issues arise after cutover:
1. Set PAYMENT_USE_NEW_ROUTER=false
2. Old PaymentService is still available (not deleted until step 5)
3. Deploy rollback version
```

---

### Implementation

```
/implement payment-refactor-phase-6-cutover
```

**Commits made:**
- `payments: update PaymentController to use router`
- `payments: remove migration mode flags`
- `refactor: delete legacy PaymentService`
- `docs: update ARCHITECTURE.md with new payment structure`
- `test: add final integration test suite`

---

### Review

```
/review payment-refactor-phase-6-cutover
```

**Review Report:**
```
[VERDICT: PASS] ✅

- 8 files changed (+1 deleted)
- +312 -847 lines (net reduction—debt paid!)
- 52/52 tests passing
- Legacy code removed
- Migration validated

Scope deviations:
- Added PaymentController refactor (was implicit in phase 4)
- No unplanned changes
```

Phase 6 merges.

---

## Epic Completion

### Invoking `/recap`

```
/recap payment-refactor
```

**Epic Summary:**
```
Epic: payment-refactor
Status: COMPLETE
Duration: 10 days
Risk Level: HIGH
Files changed: 28
Files deleted: 1
Lines added: 1,892
Lines removed: 2,341
Tests: 89 passing

Phase Timeline:
- Phase 1 (interfaces): Feb 1-2 — COMPLETE
- Phase 2 (Stripe): Feb 2-3 — COMPLETE
- Phase 3 (PayPal): Feb 3-4 — COMPLETE
- Phase 4 (router): Feb 5-6 — COMPLETE
- Phase 5 (migration): Feb 7-10 — COMPLETE
- Phase 6 (cutover): Feb 10-11 — COMPLETE

Migration Validation:
- 147 shadow transactions processed
- 100% parity achieved
- 19% latency improvement
- 0 discrepancies

Exit Criteria Status:
✅ All payment flows work identically with new implementation
✅ Old PaymentService is deleted (code removed)
✅ All tests pass
✅ No performance regression (19% improvement)
✅ Migration validated with 100% parity
```

### Final Merge

```bash
git checkout main && git pull && git merge epic/payment-refactor
```

---

## Lessons Learned

1. **Migration Pattern is Essential:** The shadow-mode parallel run in Phase 5 caught no issues in this case, but the pattern gave the team confidence to proceed and provided a rollback path.

2. **High-Risk Refactors Need Epic Structure:** Breaking this into 6 phases allowed each piece to be validated independently before the risky cutover.

3. **Interface-First Design:** Starting with the interface (Phase 1) made provider implementations straightforward and testable in isolation.

4. **Rollback Path Must Be Defined:** Phase 6's rollback plan was documented before implementation. When a late-stage issue arose during cutover, rollback took 10 minutes.

5. **Feature Flags Enable Safe Deployment:** The PAYMENT_USE_NEW_ROUTER flag allowed instant rollback without code changes.

6. **Net Code Reduction:** The refactor actually removed more code than it added—technical debt was genuinely paid down, not just restructured.

---

## Related Files

- Epic plan: `.plans/epics/payment-refactor/README.md`
- Phase plans: `.plans/epics/payment-refactor/`
- ARCHITECTURE.md (updated)
