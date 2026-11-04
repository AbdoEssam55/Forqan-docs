# Concurrency Strategies

Choose the right strategy based on the scenario.

## Strategy 1: Optimistic Locking (RowVersion)

**When**: Entities edited by multiple users; collaborative workflows; high read, low write.

**How**:
- Add `[Timestamp] public byte[]? RowVersion { get; set; }` to entity
- EF Core includes it in UPDATE; if another request changed the row, `DbUpdateConcurrencyException` is thrown
- Return 409 Conflict; ask user to refresh and retry

**Pros**: No locks; good performance; handles rare conflicts
**Cons**: User may need to retry; not suitable for high-conflict scenarios

## Strategy 2: Pessimistic Locking (UPDLOCK)

**When**: One-time-use resources (tokens, reset codes); balance checks; critical read-before-write.

**How**:
```csharp
.FromSqlRaw("SELECT * FROM Table WITH (UPDLOCK, ROWLOCK) WHERE Id = {0}", id)
```
- Must run within a transaction
- Second request blocks until first commits

**Pros**: Prevents race entirely; no retries needed
**Cons**: Reduces throughput; risk of deadlocks if not careful

## Strategy 3: Database Constraints

**When**: Natural uniqueness (email, code, username).

**How**:
- Add unique index or constraint
- Catch `DbUpdateException`; check inner SQLException.Number (2601 or 2627)
- Return 409 with user-friendly message

**Pros**: Database enforces; no application logic
**Cons**: Only for uniqueness; doesn't help with lost updates

## Strategy 4: Idempotency Keys

**When**: Payments; mutations without natural unique key; client-retriable operations.

**How**: Client sends `Idempotency-Key` header; server deduplicates.

**Pros**: Handles retries; works across distributed systems
**Cons**: Requires client cooperation; storage overhead

## Decision Matrix

| Scenario | Recommended Strategy |
|----------|---------------------|
| User edits profile | Optimistic (RowVersion) |
| Refresh token | Pessimistic + IsUsed flag |
| Password reset code | Pessimistic + ResetCodeUsed flag |
| Payment | Idempotency + Pessimistic for balance |
| User registration | Database constraint + optional idempotency |
| Create role/plan | Database constraint (unique name) |
