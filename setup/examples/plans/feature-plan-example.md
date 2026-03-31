# Plan: Add OAuth2 Social Login Support

**Status:** IN_PROGRESS
**Category:** feature
**Branch:** feat/oauth2-social-login
**Parent:** main
**Created:** 2026-03-15
**One-liner:** Implement OAuth2 flow supporting Google and GitHub providers with JWT session tokens

## Scope

Adds social login functionality via Google and GitHub OAuth2 providers. Users can link/unlink social accounts to their existing account. New users signing up via social login get an account created automatically. Returns a JWT token upon successful authentication.

> **Note:** Why JWT instead of sessions? This API will be consumed by mobile clients and potentially third-party SPAs. Sessions work poorly across origins. JWT allows stateless auth with refresh token rotation.

## Files to Change

- `src/auth/oauth2-provider.ts` — OAuth2 provider abstraction (Google, GitHub)
- `src/auth/oauth2.service.ts` — Token exchange, user creation/lookup logic
- `src/auth/jwt.service.ts` — JWT generation and validation (existing, extended)
- `src/auth/strategies/google.strategy.ts` — Google OAuth2 strategy implementation
- `src/auth/strategies/github.strategy.ts` — GitHub OAuth2 strategy implementation
- `src/user/social-link.entity.ts` — Database entity for social account links
- `src/user/user.entity.ts` — Extended with socialLink relation
- `src/auth/auth.controller.ts` — New `/auth/oauth2/:provider/callback` endpoint
- `src/auth/auth.service.ts` — Extended with `linkSocialAccount()` and `unlinkSocialAccount()`
- `src/auth/auth.module.ts` — Import OAuth2 modules
- `migrations/20260315_add_social_links.sql` — Schema migration
- `src/auth/oauth2.service.spec.ts` — Unit tests
- `src/auth/oauth2.service.integration.spec.ts` — Integration tests with mocked OAuth2 servers
- `e2e/social-login.spec.ts` — End-to-end social login flow

> **Note:** Why separate strategy files? Each provider has slightly different userinfo endpoints and claim structures. Keeping them separate makes it easier to add providers later without touching core OAuth2 logic.

## Test Cases

- [ ] OAuth2 redirect URL is constructed correctly for Google
- [ ] OAuth2 redirect URL is constructed correctly for GitHub
- [ ] Token exchange succeeds with valid authorization code
- [ ] Token exchange fails gracefully with invalid code (returns 401)
- [ ] New user is created when social account has no existing link
- [ ] Existing user is returned when social account is already linked
- [ ] User can link multiple social accounts to same user
- [ ] User cannot link a social account already linked to another user
- [ ] User can unlink a social account (must retain password login or another social link)
- [ ] User cannot unlink last login method if no password set
- [ ] JWT token is returned with correct claims (userId, email, providers)
- [ ] Expired OAuth2 tokens are rejected with appropriate error
- [ ] Rate limiting prevents brute-force token exchange attempts

> **Note:** The "cannot unlink last login method" test is critical for security. We don't want users locking themselves out of accounts with no recovery path.

## Implementation Notes

- Use `axios` for OAuth2 token exchange (already in project dependencies)
- Store only `provider` + `providerUserId` + `userId` in social_links table, not OAuth tokens
  > **Note:** We intentionally do NOT store access/refresh tokens. Social login is for authentication only, not API access. Storing tokens creates security surface area with no benefit.
- Use existing JWT service for token generation (don't create new auth mechanism)
- OAuth2 state parameter will be a signed JWT containing `returnTo` URL to prevent CSRF
- Provider userinfo endpoints: Google (`https://www.googleapis.com/oauth2/v2/userinfo`), GitHub (`https://api.github.com/user`)

### Environment Variables Needed

```
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=
GITHUB_CLIENT_ID=
GITHUB_CLIENT_SECRET=
OAUTH2_REDIRECT_BASE=https://app.example.com
JWT_SECRET=<already exists>
```

### Gotchas

- GitHub's userinfo endpoint returns `id` as integer, Google returns `sub` as string. Normalize to string.
- GitHub emails require `user:email` scope. Request it explicitly even though it's default for most OAuth flows.
- Google returns `email_verified: true` for verified emails only. Handle unverified gracefully (reject or flag).

## Future (Out of Scope)

The following came up during planning but are NOT part of this task:

- **Other providers (Apple, Facebook, Twitter/X, LinkedIn)** — Add to backlog, each needs spike for their specific quirks
- **Social login via popup (OIDC hybrid flow)** — Would require CSRF token handling changes
- **Linking social account during checkout flow** — Has UX implications, needs design review
- **OAuth2 token refresh for long-lived sessions** — Not needed for MVP; JWT expiry handles this
- **Account merge when same email from different providers** — Complex conflict resolution, deferred

> **Note:** We captured these in Future rather than just forgetting them because they WILL come up in code review or planning meetings. Putting them here ensures they don't get lost and can be prioritized later.

---

## Completion Log

**Completed:** YYYY-MM-DD
**Actual changes:** (filled after implementation)
**Deviated from plan:** (filled after implementation)
**Impact:** (filled after implementation)
