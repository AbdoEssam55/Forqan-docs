# Race Conditions

## What Is a Race Condition?

When multiple requests arrive simultaneously and the outcome depends on timing, a race condition can occur. The "check-then-act" pattern is vulnerable: two requests both check (e.g., "does this email exist?"), both get "no," then both act (create user), resulting in duplicates.

## Why Frontend Solutions Fail

Disabling the submit button until the request completes does NOT prevent race conditions:

- Users can bypass the UI (curl, Postman, API clients)
- Network retries can send duplicate requests
- Malicious users can intentionally exploit the vulnerability

**All protection must be implemented server-side.**

## Common Vulnerable Patterns

1. **Duplicate registration**: Check email exists → Create user (both can pass the check)
2. **Refresh token reuse**: Validate token → Revoke → Issue new (both can validate before revoke)
3. **Reset code reuse**: Validate code → Reset password → Clear code (both can validate)
4. **Double payment**: Check balance → Deduct → Record (both can read same balance)
5. **Lost updates**: User A loads entity → User B loads → User A saves → User B saves (A's changes lost)

## Mitigations

| Pattern | Mitigation |
|---------|------------|
| Duplicate creation | Unique constraints + DbUpdateException handling |
| Token/code reuse | One-time-use flag + pessimistic locking |
| Double payment | Idempotency keys + pessimistic locking |
| Lost updates | RowVersion (optimistic concurrency) |

See [concurrency-strategies.md](concurrency-strategies.md) for implementation details.
