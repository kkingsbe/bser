# Scenario: Add User Authentication System

This scenario walks through implementing a comprehensive user authentication system for a Node.js/Express API using BSER. The feature touches multiple layers—database, service logic, API routes, and middleware—so it warrants decomposition into an epic with ordered phases.

## Project Context

**Tech Stack:** Node.js, Express, PostgreSQL, TypeScript
**Current State:** API has no authentication. All endpoints are publicly accessible.
**Team:** 2 developers

---

## Starting the Epic

The team lead has discussed adding JWT-based authentication to the API. The work is scoped and approved, but it's too large for a single plan—it touches the user model, password hashing, token generation/validation, middleware, and all existing routes that need protection.

### Invoking `/epic`

```
/epic Add complete user authentication system with JWT tokens, including user registration, login, password reset, and protected route middleware
```

The agent decomposes this into an epic:

---

**Phase Decomposition:**

```
Epic: user-auth-system
Branch: epic/user-auth-system

## Phases

| # | Plan | Description | Dependencies | Status |
|---|------|-------------|-------------|--------|
| 1 | `user-auth-system-phase-1-database` | User model, migration, and repository layer | none | PLANNING |
| 2 | `user-auth-system-phase-2-service` | Auth service with hashing, JWT generation, password reset tokens | phase 1 | PLANNING |
| 3 | `user-auth-system-phase-3-middleware` | Auth middleware for protected routes, token validation | phase 2 | PLANNING |
| 4 | `user-auth-system-phase-4-routes` | Registration, login, password reset API endpoints | phase 2, 3 | PLANNING |
| 5 | `user-auth-system-phase-5-integration` | Apply auth middleware to existing routes, test end-to-end | phase 4 | PLANNING |

## Risks & Open Questions

- Password reset flow requires email integration—will mock for now
- Session invalidation on password change needs strategy
- Should refresh tokens be implemented now or deferred?

## Exit Criteria

- All existing endpoints require valid JWT except /auth/register and /auth/login
- New users can register, login, and receive JWT
- Password reset flow completes (email mocked)
- No regression in existing functionality
```

---

## Phase 1: User Model and Database Layer

The epic branch is created. The agent begins Phase 1.

### Invoking `/scope` for Phase 1

```
/scope Create user model with migration and repository for database operations
```

The scope command creates a branch `feat/user-auth-system-phase-1-database` from `epic/user-auth-system` and produces a plan:

---

**Plan: user-auth-system-phase-1-database**

```markdown
# Plan: user-auth-system-phase-1-database

**Status:** IN_PROGRESS
**Category:** feature
**Branch:** feat/user-auth-system-phase-1-database
**Parent:** epic/user-auth-system
**Created:** 2024-01-15
**One-liner:** Create user model, database migration, and repository layer

## Scope

Create the foundational user data model with PostgreSQL migration, TypeScript interfaces, and a repository class providing CRUD operations. No auth logic yet—this is purely data access.

## Files to Change

- `src/models/user.ts` — UserType interface and User entity class
- `src/database/migrations/001_create_users.sql` — PostgreSQL migration
- `src/database/repositories/userRepository.ts` — User repository with CRUD
- `src/database/repositories/index.ts` — Export all repositories
- `tests/unit/userRepository.spec.ts` — Repository unit tests

## Test Cases

- [ ] UserRepository.findById returns user when exists
- [ ] UserRepository.findById returns null when not exists
- [ ] UserRepository.findByEmail returns user when exists
- [ ] UserRepository.create hashes password before saving
- [ ] UserRepository.update updates only provided fields

## Implementation Notes

- Use bcrypt for password hashing (import from 'bcrypt')
- User entity should have: id, email, passwordHash, createdAt, updatedAt, passwordResetToken, passwordResetExpires
- Use UUID for user IDs
- Follow existing migration pattern in the codebase

## Future (Out of Scope)

- Auth service logic
- JWT generation
- API routes
```

---

### Invoking `/implement`

```
/implement user-auth-system-phase-1-database
```

The agent:
1. Reads the plan
2. Runs baseline tests
3. Writes tests first (TDD)
4. Implements the model, migration, and repository
5. Commits after each logical unit

**Commits made:**
- `db: add users migration with password reset fields`
- `model: add UserType interface and User entity`
- `repo: implement UserRepository with CRUD operations`
- `test: add UserRepository unit tests`

After implementation, the plan doc is updated with completion info.

---

### Invoking `/review`

```
/review user-auth-system-phase-1-database
```

The reviewer agent:
1. Runs `git diff main..HEAD` to see all changes
2. Compares against the plan's "Files to Change"
3. Runs the test suite, comparing against baseline
4. Evaluates code quality and patterns

**Review Report Summary:**
```
[VERDICT: PASS] ✅

- 5 files changed
- +248 -12 lines
- 5/5 tests passing
- All planned files modified
- No scope deviations
```

Phase 1 is merged into the epic branch:

```bash
git checkout epic/user-auth-system && git merge feat/user-auth-system-phase-1-database
```

---

## Phase 2: Auth Service Layer

With the user model in place, the team moves to Phase 2.

### Invoking `/scope` for Phase 2

```
/scope Implement authentication service with password hashing, JWT generation, and password reset token logic
```

**Plan: user-auth-system-phase-2-service** is created:

```markdown
# Plan: user-auth-system-phase-2-service

**Status:** IN_PROGRESS
**Category:** feature
**Branch:** feat/user-auth-system-phase-2-service
**Parent:** epic/user-auth-system
**Created:** 2024-01-16
**One-liner:** Auth service with password hashing, JWT generation, and password reset

## Scope

Implement the auth service that handles password verification, JWT token generation/validation, and password reset token generation. This service will be used by the API routes.

## Files to Change

- `src/services/authService.ts` — Main auth service class
- `src/services/jwt.ts` — JWT token utilities
- `src/services/password.ts` — Password hashing utilities
- `tests/unit/authService.spec.ts` — Auth service tests
- `tests/unit/jwt.spec.ts` — JWT utility tests

## Test Cases

- [ ] AuthService.register creates user with hashed password
- [ ] AuthService.register rejects duplicate email
- [ ] AuthService.login returns JWT on valid credentials
- [ ] AuthService.login throws on invalid password
- [ ] AuthService.validateToken returns payload for valid JWT
- [ ] AuthService.validateToken throws for invalid/expired JWT
- [ ] AuthService.generatePasswordResetToken stores token in user record
- [ ] AuthService.resetPassword verifies token and updates password

## Implementation Notes

- Use jsonwebtoken library for JWT
- JWT expires in 24 hours
- Password reset tokens expire in 1 hour
- Store reset tokens as hashed values in DB
- Follow existing service pattern (dependency injection of UserRepository)
```

---

### Implementation Continues

```
/implement user-auth-system-phase-2-service
```

**Commits made:**
- `service: add password hashing utilities`
- `service: add JWT token generation and validation`
- `service: implement AuthService with register/login`
- `service: add password reset token generation and validation`
- `test: add auth service unit tests`

---

### Review

```
/review user-auth-system-phase-2-service
```

**Review Report Summary:**
```
[VERDICT: PASS] ✅

- 4 files changed
- +312 -28 lines
- 8/8 tests passing
- No issues
```

Phase 2 merges into epic:

```bash
git checkout epic/user-auth-system && git merge feat/user-auth-system-phase-2-service
```

---

## Phase 3: Auth Middleware

### Invoking `/scope`

```
/scope Create authentication middleware for protecting routes and validating JWT tokens
```

**Plan: user-auth-system-phase-3-middleware** is created and implemented.

This phase creates:
- `src/middleware/auth.ts` — JWT validation middleware
- `src/middleware/requestUser.ts` — Type augmentation for Express Request
- Tests for middleware

---

### Review

```
/review user-auth-system-phase-3-middleware
```

Phase 3 merges into epic.

---

## Phase 4: API Routes

### Invoking `/scope`

```
/scope Create authentication API endpoints: POST /auth/register, POST /auth/login, POST /auth/password-reset, POST /auth/reset-password
```

**Plan: user-auth-system-phase-4-routes:**

```markdown
# Plan: user-auth-system-phase-4-routes

**Status:** IN_PROGRESS
**Category:** feature
**Branch:** feat/user-auth-system-phase-4-routes
**Parent:** epic/user-auth-system
**Created:** 2024-01-18
**One-liner:** Auth API endpoints for registration, login, and password reset

## Scope

Create the Express routes for user authentication. All endpoints return JSON responses. Email sending is mocked for password reset.

## Files to Change

- `src/routes/auth.ts` — Auth route handlers
- `src/routes/index.ts` — Register auth routes
- `tests/integration/auth.spec.ts` — Integration tests for auth endpoints

## Test Cases

- [ ] POST /auth/register returns 201 with JWT on success
- [ ] POST /auth/register returns 400 on duplicate email
- [ ] POST /auth/login returns 200 with JWT on valid credentials
- [ ] POST /auth/login returns 401 on invalid password
- [ ] POST /auth/login returns 404 on unknown email
- [ ] POST /auth/password-reset returns 200 even if email not found (security)
- [ ] POST /auth/reset-password returns 200 with valid token
- [ ] POST /auth/reset-password returns 400 with expired token

## API Contract

### POST /auth/register
Request: `{ "email": "user@example.com", "password": "securePassword123" }`
Response: `{ "token": "jwt...", "user": { "id": "uuid", "email": "..." } }`

### POST /auth/login
Request: `{ "email": "user@example.com", "password": "securePassword123" }`
Response: `{ "token": "jwt...", "user": { "id": "uuid", "email": "..." } }`

### POST /auth/password-reset
Request: `{ "email": "user@example.com" }`
Response: `{ "message": "If the email exists, a reset link was sent" }`

### POST /auth/reset-password
Request: `{ "token": "reset-token", "newPassword": "newPassword123" }`
Response: `{ "message": "Password reset successful" }`
```

---

### Implementation

```
/implement user-auth-system-phase-4-routes
```

The agent creates routes, tests them with supertest for integration testing, and commits.

---

### Review

```
/review user-auth-system-phase-4-routes
```

Phase 4 merges into epic.

---

## Phase 5: Integration with Existing Routes

### Invoking `/scope`

```
/scope Apply authentication middleware to all existing routes, requiring JWT for access
```

**Plan: user-auth-system-phase-5-integration:**

```markdown
# Plan: user-auth-system-phase-5-integration

**Status:** IN_PROGRESS
**Category:** feature
**Branch:** feat/user-auth-system-phase-5-integration
**Parent:** epic/user-auth-system
**Created:** 2024-01-19
**One-liner:** Apply auth middleware to existing routes and add new auth endpoints

## Scope

Protect all existing API routes with the auth middleware. Public routes remain accessible without authentication. This is the final integration piece.

## Files to Change

- `src/routes/users.ts` — Add auth middleware
- `src/routes/products.ts` — Add auth middleware
- `src/routes/orders.ts` — Add auth middleware
- `src/app.ts` — Register auth routes at /auth
- `tests/integration/protected-routes.spec.ts` — Verify protection works
- `tests/integration/public-routes.spec.ts` — Verify public routes still work

## Test Cases

- [ ] GET /users returns 401 without token
- [ ] GET /users returns 200 with valid token
- [ ] GET /products returns 401 without token
- [ ] GET /products returns 200 with valid token
- [ ] GET /orders returns 401 without token
- [ ] GET /orders returns 200 with valid token
- [ ] GET /health returns 200 without token (public)
- [ ] POST /auth/register returns 200 without token (public)
- [ ] POST /auth/login returns 200 without token (public)

## Implementation Notes

- Only /health, /auth/register, and /auth/login are public
- All other routes require Bearer token in Authorization header
- Use app.use('/auth', authRoutes) before the protected routes
```

---

### Implementation

```
/implement user-auth-system-phase-5-integration
```

**Commits made:**
- `routes: add auth middleware to users routes`
- `routes: add auth middleware to products routes`
- `routes: add auth middleware to orders routes`
- `app: register auth routes`
- `test: add integration tests for protected routes`
- `test: add integration tests for public routes`

---

### Review

```
/review user-auth-system-phase-5-integration
```

**Review Report Summary:**
```
[VERDICT: PASS] ✅

- 7 files changed
- +156 -23 lines
- 14/14 tests passing
- All planned files modified
- No scope deviations
```

Phase 5 merges into epic.

---

## Epic Completion

With all phases complete, the epic is ready to merge to main.

### Invoking `/recap` for the Epic

```
/recap user-auth-system
```

The recap shows the full journey:

```
Epic: user-auth-system
Status: COMPLETE
Phases: 5
Duration: 5 days
Files changed: 25
Lines added: 1,247
Lines removed: 89
Tests: 45 passing

Phase Timeline:
- Phase 1 (database): Jan 15 — COMPLETE
- Phase 2 (service): Jan 16 — COMPLETE
- Phase 3 (middleware): Jan 17 — COMPLETE
- Phase 4 (routes): Jan 18-19 — COMPLETE
- Phase 5 (integration): Jan 19-20 — COMPLETE

Exit Criteria Status:
✅ All existing endpoints require valid JWT except /auth/register and /auth/login
✅ New users can register, login, and receive JWT
✅ Password reset flow completes (email mocked)
✅ No regression in existing functionality
```

### Final Merge

```bash
git checkout main && git pull && git merge epic/user-auth-system
```

The epic branch is deleted after successful merge:

```bash
git branch -d epic/user-auth-system
```

---

## Lessons Learned

1. **Decomposition Strategy:** Layer-by-layer (model → service → middleware → routes → integration) worked well for auth because each phase built on the previous.

2. **Epic Branch Discipline:** Keeping phase branches short-lived and merging quickly prevented integration nightmares.

3. **Test Coverage:** Each phase had unit tests; Phase 5 added integration tests that verified the full stack.

4. **Future Enhancement:** Refresh tokens were intentionally deferred—the epic captured this in risks and it was added to the backlog.

---

## Related Files

- Epic plan: `.plans/epics/user-auth-system/README.md`
- Phase plans: `.plans/epics/user-auth-system/`
