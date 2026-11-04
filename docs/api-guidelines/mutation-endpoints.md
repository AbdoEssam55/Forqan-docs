# Mutation Endpoint Standards

Standards for POST, PUT, PATCH, and DELETE endpoints.

## POST (Create)

### Idempotency
- For payments and critical operations: require `Idempotency-Key` header via `[RequiresIdempotency]`
- For others: optional; consider if operation can be retried

### Uniqueness
- Add unique constraints for natural keys (email, code, name)
- Catch `DbUpdateException`; if inner exception is SQL 2601/2627, return 409 Conflict
- Message: "A record with this value already exists" or similar

### Transactions
- Use transaction for multi-step creates (e.g., create user + send email)
- Rollback on any failure

## PUT/PATCH (Update)

### Optimistic Concurrency
- Entities inheriting `AuditedEntity` have RowVersion
- Catch `DbUpdateConcurrencyException`; return 409 Conflict
- Message: "The record was modified by another user. Please refresh and try again."

### Partial Updates (PATCH)
- Validate only provided fields
- Do not overwrite unprovided fields with defaults
- Consider JSON Patch (RFC 6902) for complex partial updates

## DELETE

### Soft Delete
- Prefer soft delete (IsDeleted = true) for audit trail
- Hard delete only when required (e.g., GDPR right to erasure)
- Authorization: verify user can delete this resource (tenant isolation)

## Response Codes

| Scenario | Code |
|----------|------|
| Success (create) | 201 Created |
| Success (update/delete) | 200 OK |
| Duplicate (unique violation) | 409 Conflict |
| Concurrency conflict | 409 Conflict |
| Validation error | 422 Unprocessable Entity |
| Not found | 404 NotFound |
| Unauthorized | 401 Unauthorized |
| Forbidden | 403 Forbidden |
