# Security Checklist

Use this checklist before deploying new API features or endpoints.

## New Endpoint Checklist

### POST (Create)
- [ ] Unique constraint on natural key (if applicable)
- [ ] Catch `DbUpdateException` for duplicate key; return 409 with friendly message
- [ ] Idempotency key required? If yes, add `[RequiresIdempotency]`
- [ ] Transaction wraps multi-step operations
- [ ] Authorization (role/claim) enforced
- [ ] Input validation (FluentValidation)
- [ ] Rate limiting considered

### PUT/PATCH (Update)
- [ ] Entity has RowVersion for optimistic concurrency
- [ ] Catch `DbUpdateConcurrencyException`; return 409
- [ ] Authorization enforced
- [ ] Tenant isolation verified
- [ ] Input validation

### Token / Code Operations
- [ ] One-time-use flag (IsUsed, ResetCodeUsed)
- [ ] Pessimistic locking when loading
- [ ] Entire flow in transaction
- [ ] Rate limiting applied

### Payment / Financial
- [ ] `[RequiresIdempotency]` applied
- [ ] Pessimistic locking for balance/stock checks
- [ ] Transaction with appropriate isolation
- [ ] Server-side amount validation
- [ ] Audit logging

## General
- [ ] No secrets in logs
- [ ] CORS restricted to known origins
- [ ] HTTPS enforced
- [ ] Error messages don't leak sensitive info
