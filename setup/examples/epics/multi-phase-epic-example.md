# Epic: Migrate Legacy REST API to GraphQL

**Status:** IN_PROGRESS
**Branch:** epic/graphql-migration
**Created:** 2026-03-01
**One-liner:** Replace the v1 REST API with a GraphQL API enabling flexible querying, real-time subscriptions, and improved developer experience

## Objective

This epic migrates the legacy REST API (`/api/v1/*`) to GraphQL, providing clients with a more flexible, efficient, and introspectable interface. The migration proceeds in four ordered phases to manage risk and allow gradual client migration. When complete:

- All data access flows through GraphQL resolvers with centralized authorization
- Real-time updates are available via GraphQL subscriptions for live dashboards
- The REST API is fully deprecated with a clear sunset timeline
- Client migration is opt-in, with REST serving as fallback during transition
- Performance is improved through batching and query optimization

> **Note:** Why phased? The previous v1→v2 migration attempted a "big bang" cutover and caused a week-long incident. Breaking into phases allows rollback at each stage and gives frontend teams time to migrate without freezing.

## Phases

Each phase is a self-contained plan that branches from and merges back into `epic/graphql-migration`. Phases are ordered — later phases depend on earlier ones being merged.

| # | Plan | Description | Dependencies | Status |
|---|------|-------------|--------------|--------|
| 1 | `graphql-phase-1-schema-design` | Design GraphQL schema covering all v1 entities, define naming conventions, establish patterns | none | COMPLETE |
| 2 | `graphql-phase-2-resolver-implementation` | Implement resolvers for all queries and mutations, integrate with existing services | phase 1 | IN_PROGRESS |
| 3 | `graphql-phase-3-client-migration` | Migrate frontend clients from REST to GraphQL incrementally, establish GraphQL as default | phase 2 | PLANNING |
| 4 | `graphql-phase-4-deprecation-cleanup` | Remove REST endpoints, clean up legacy code, archive v1 documentation | phase 3 | PLANNING |

### Phase 1: Schema Design (COMPLETED)

**Completed:** 2026-03-10

Key decisions made:
- Used "buyer" and "seller" as domain prefixes to avoid naming conflicts (e.g., `BuyerUser`, `SellerOrder`)
- Adopted Relay-style connection pagination for all list queries (required for mobile infinite scroll)
- Defined custom scalars: `DateTime`, `JSON`, `UUID`
- Authorization modeled at schema level using custom directive `@auth(requires: [ADMIN, USER])`

> **Note:** Why Relay connections? The mobile team requires cursor-based pagination for efficient infinite scroll. REST's offset-based pagination causes performance issues at scale (O(n) skip). Relay connections are more work upfront but solve a real problem.

Schema coverage:
- All 23 v1 entities mapped
- 47 queries, 31 mutations identified
- 8 subscriptions proposed for real-time features

### Phase 2: Resolver Implementation (IN PROGRESS)

**Started:** 2026-03-11

Current focus:
- Implementing resolvers for `User`, `Order`, and `Product` types
- DataLoader pattern adopted for N+1 query prevention
- Existing service layer (`UserService`, `OrderService`, `ProductService`) reused, not rewritten

### Phase 3: Client Migration (PLANNING)

**Planned start:** 2026-04-07

Planned activities:
- Create GraphQL client wrapper with caching (Apollo Client 3)
- Migrate web dashboard first (team with most GraphQL experience)
- Mobile app migration deferred to phase 4 (requires additional schema work for offline support)
- Establish feature flags for gradual traffic shifting

### Phase 4: Deprecation Cleanup (PLANNING)

**Planned start:** 2026-05-01

Planned activities:
- REST API marked as deprecated in API gateway (add `Deprecation` header)
- 90-day sunset period with migration guides published
- Remove REST handlers after sunset period
- Archive old controller files to `src/archive/rest-v1/` for reference

## Architecture Impact

### Modules Affected

- **src/api/rest/v1/** — Will be deprecated then removed (phase 4)
- **src/api/graphql/** — New GraphQL layer (phases 1-2)
- **src/services/** — Unchanged, called by GraphQL resolvers
- **src/auth/** — Updated to support GraphQL context extraction
- **src/database/** — New DataLoader utilities added
- **api-gateway/** — Route configuration updated for GraphQL endpoint

### Data Model Changes

- No schema migrations required for phase 1-3
- Phase 4 may introduce read-only computed fields as resolver extensions

> **Note:** Why no database changes? The GraphQL schema maps to existing entities 1:1. The value is in the query layer, not the data layer. Clients get better querying; we get centralized auth and N+1 prevention.

## Risks & Open Questions

- **Performance regression risk**: GraphQL flexibility can cause over-fetching. Mitigation: query complexity analysis and depth limiting in production.
- **Subscription scaling**: Current WebSocket infrastructure designed for low connection counts. Mitigation: spike in phase 2 to validate scaling approach.
- **Mobile offline support**: Apollo Client 3 caching works differently than current REST caching. Mobile team concerned about offline-first behavior. Decision deferred to phase 3 planning.
- **Error handling consistency**: REST returns structured errors (`{code, message, details}`). GraphQL errors are different. Need to decide on unified error format across both during coexistence.
- **Schema versioning**: REST had v1, v2 versioning. GraphQL schema evolves differently. Need GraphQL-specific versioning strategy (minor additions OK, breaking changes = new schema).

> **Note:** The schema versioning question is the most contentious. REST versioning was tried and true; GraphQL's "evolve without versions" is elegant but requires discipline. We've decided to add a `schemaVersion` field to introspection and document breaking change policy separately.

## Exit Criteria

The epic is complete when ALL of the following are true:

- [ ] Phase 1 schema covers 100% of v1 REST functionality (verified by contract test suite)
- [ ] Phase 2 resolvers pass all contract tests (REST vs GraphQL responses match)
- [ ] Phase 2 performance: p95 latency ≤ 150ms for equivalent REST queries (benchmark validated)
- [ ] Phase 3: Web dashboard serves 100% of traffic via GraphQL
- [ ] Phase 3: Mobile app passes GraphQL integration testing
- [ ] Phase 4: REST API receives zero production traffic (verified via metrics)
- [ ] Phase 4: Old REST controller files archived or removed
- [ ] Documentation updated: API docs now GraphQL-first, migration guide published
- [ ] Team trained: All backend engineers can write GraphQL resolvers (pair sessions completed)

> **Note:** The "zero production traffic" criteria is strict deliberately. In the previous migration, a few "forgotten" endpoints were left running and caused confusion. This time, we remove completely.

---

## Progress Log

| Phase | Started | Completed | Notes |
|-------|---------|-----------|-------|
| 1: Schema Design | 2026-03-02 | 2026-03-10 | Schema reviewed by all backend leads, approved 2026-03-09 |
| 2: Resolver Implementation | 2026-03-11 | | In progress. User, Order, Product resolvers done. 18 of 47 queries implemented. |
| 3: Client Migration | | | Not started |
| 4: Deprecation Cleanup | | | Not started |
