# Payment API Security Guidelines

When implementing payment endpoints in the Alforqan platform, follow these security rules to prevent race conditions, double-spending, and fraud.

## Idempotency (Required)

**All payment endpoints MUST use the `[RequiresIdempotency]` attribute.**

```csharp
[RequiresIdempotency]
[HttpPost("pay")]
public async Task<IActionResult> ProcessPayment([FromBody] PaymentRequest request)
```

The client must send a unique `Idempotency-Key` header (UUID) for each logical payment. Duplicate requests with the same key return the cached response without re-processing.

## Database Transactions

- Use **Serializable** isolation level for financial operations when checking balances
- Wrap all payment operations (balance check, deduction, recording) in a single transaction
- Rollback on any failure; never partial commit

## Pessimistic Locking

- Use `UPDLOCK, ROWLOCK` when reading account balances before deducting
- Prevents concurrent requests from reading stale balance and over-spending

## Never Trust Client Amounts

- Validate payment amounts server-side against product/service pricing
- Re-fetch prices from database; never use client-submitted amounts for calculations

## Audit Trail

- Log all payment attempts (success and failure) with timestamp, user, amount, idempotency key
- Store in immutable audit table for compliance and dispute resolution

## Checklist for New Payment Endpoints

- [ ] `[RequiresIdempotency]` attribute applied
- [ ] Transaction with appropriate isolation level
- [ ] Pessimistic locking for balance/stock checks
- [ ] Server-side amount validation
- [ ] Audit logging implemented
- [ ] Rate limiting considered
- [ ] Unit tests for concurrent request scenarios
