# Plan: Fix Memory Leak in WebSocket Connection Handler

**Status:** IN_PROGRESS
**Category:** bugfix
**Branch:** fix/websocket-memory-leak
****Parent:** main
**Created:** 2026-03-28
**One-liner:** Fix unbounded memory growth in WebSocket handler by correcting event listener cleanup and removing stale heartbeat interval references

## Scope

Resolves a memory leak in `src/websocket/connection-handler.ts` where WebSocket connections are not properly cleaned up on disconnect, and heartbeat intervals are not cleared when connections terminate. This causes memory to grow unbounded in long-running server processes.

> **Note:** Why is this categorized as bugfix rather than refactor? The behavior is wrong (memory leak) but the fix should not change external behavior — clients should not notice any difference. If we were restructuring for maintainability while fixing, it would be hybrid.

## Files to Change

- `src/websocket/connection-handler.ts` — Fix cleanup logic, proper removeEventListener calls
- `src/websocket/heartbeat.service.ts` — Clear interval on disconnect, use WeakMap for connection→interval mapping
- `src/websocket/connection-handler.spec.ts` — Add memory leak reproduction test
- `src/websocket/websocket.integration.spec.ts` — Long-running integration test to verify fix

> **Note:** Why add a specific memory leak test? This bug has resurfaced twice. A regression test that explicitly exercises the disconnect path will catch future regressions before they reach production.

## Test Cases

- [ ] Memory does not grow after 100 connect/disconnect cycles (baseline vs. after fix)
- [ ] `removeEventListener` is called for all registered listeners on disconnect
- [ ] `clearInterval` is called for heartbeat on disconnect
- [ ] Heartbeat interval is cleared even if disconnect fires before first heartbeat fires
- [ ] Cleanup happens correctly on unexpected socket close (client crash)
- [ ] Cleanup happens correctly on normal socket close (client logout)
- [ ] No dangling intervals exist after connection pool of 50 concurrent connections disconnects
- [ ] Error events on socket do not prevent cleanup

> **Note:** The "heartbeat before first interval fires" test catches a specific edge case where the heartbeat interval ID is stored but the interval hasn't been created yet, or the cleanup runs before the first tick.

## Implementation Notes

### Root Cause Analysis

After profiling with `--inspect` and heap snapshots, the leak has two sources:

1. **Event listeners not removed**: When `handleConnection()` registers listeners for `message`, `close`, and `error`, these are never removed. Each reconnect from a client creates new listeners, accumulating in memory.

2. **Heartbeat intervals not cleared**: The `heartbeat.service.ts` creates an interval per connection but only clears it if `disconnect()` is called. If the socket closes unexpectedly, the interval keeps running and holds a reference to the connection object via closure.

### Fix Strategy

```typescript
// Connection handler - use once option for listeners
socket.once('close', () => this.handleDisconnect(socket));
socket.once('error', () => this.handleDisconnect(socket));

// Heartbeat - use WeakMap instead of storing interval ID on connection object
// This prevents intervals from holding references that prevent GC
const heartbeatIntervals = new WeakMap<WebSocket, NodeJS.Timeout>();
```

> **Note:** Why `socket.once()` instead of `socket.on()`? If we use `on()`, we MUST call `removeListener()` on disconnect. Using `once()` auto-removes after first fire, but we need the listener to persist until close. The correct pattern is `on()` with explicit `removeListener()` in the disconnect handler. The `once()` was attempted as "simpler" but doesn't work because `once` fires and removes on ANY first event, not just close.

### Why WeakMap for Heartbeat Intervals

Storing the interval ID on the connection object (`connection.heartbeatInterval = setInterval(...)`) creates a strong reference cycle:

```
connection → heartbeatInterval → closure → connection
```

This prevents garbage collection even after the socket is closed. WeakMap allows the interval to be garbage collected when the socket reference is dropped.

### Edge Cases to Handle

- Socket closes before heartbeat interval is created — use a flag or guard in the interval creation
- Multiple rapid connect/disconnect cycles — ensure interval cleanup doesn't throw if already cleared
- Heartbeat service initialization before socket exists — handle gracefully with null checks

## Future (Out of Scope)

- **Add connection limit per user** — Would help with other DoS vectors, separate ticket
- **Add WebSocket monitoring/metrics** — Would help catch future leaks earlier
- **Switch to uWebSockets.js** — Performance consideration, separate from correctness

> **Note:** The uWebSockets.js item came up because the current `ws` library has known patterns that make leaks easier. However, that's a significant architectural change that shouldn't block a correctness fix.

---

## Completion Log

**Completed:** YYYY-MM-DD
**Actual changes:** (filled after implementation)
**Deviated from plan:** (filled after implementation)
**Impact:** (filled after implementation)
