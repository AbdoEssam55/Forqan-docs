# Security Documentation

This folder contains security guidelines for the Alforqan backend platform.

## Contents

- [race-conditions.md](race-conditions.md) - What race conditions are and how to prevent them
- [idempotency-guide.md](idempotency-guide.md) - Implementing idempotency for critical operations
- [concurrency-strategies.md](concurrency-strategies.md) - When to use optimistic vs pessimistic locking
- [security-checklist.md](security-checklist.md) - Pre-deployment security checklist
- [attack-scenarios.md](attack-scenarios.md) - Real-world attack examples and mitigations
- [payment-api-guidelines.md](payment-api-guidelines.md) - Payment endpoint security rules

## Cursor Rules

AI-enforced rules are in `.cursor/rules/security/`:

- `race-conditions.md` - Race condition prevention
- `idempotency.md` - Idempotency implementation
- `concurrency.md` - Concurrency strategy selection
- `owasp-backend.md` - OWASP Top 10 backend rules

## Quick Reference

| Scenario | Solution |
|----------|----------|
| Duplicate user registration | DbUpdateException catch + unique constraint |
| Refresh token reuse | IsUsed flag + pessimistic locking |
| Password reset code reuse | ResetPasswordCodeUsed + pessimistic locking |
| Concurrent entity edits | RowVersion + DbUpdateConcurrencyException |
| Payment duplication | Idempotency keys + [RequiresIdempotency] |
| Duplicate entity creation | Unique constraints + DbUpdateException |
