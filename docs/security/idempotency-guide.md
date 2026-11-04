# Idempotency Implementation Guide

## Overview

Idempotency ensures that performing the same operation multiple times has the same effect as performing it once. Critical for payments, registrations, and any operation that cannot be safely retried.

## Client Flow

1. User clicks "Pay" (or triggers a critical action)
2. Client generates `Guid.NewGuid()` as idempotency key
3. Client sends request with header: `Idempotency-Key: <uuid>`
4. If request fails (network, timeout), client retries with the SAME key
5. Server returns cached response for duplicate keys

## Server Implementation

### Endpoints Requiring Idempotency

Apply `[RequiresIdempotency]` attribute:

```csharp
[RequiresIdempotency]
[HttpPost("pay")]
public async Task<IActionResult> ProcessPayment([FromBody] PaymentRequest request)
```

### How It Works

1. `IdempotencyMiddleware` runs for POST/PUT/PATCH with `Idempotency-Key` header
2. Computes request hash (method + path + body)
3. Checks `IdempotencyRecords` table for existing (key, hash, tenant)
4. If found: returns cached response immediately
5. If not: executes request, on success stores (key, hash, response) for 24h

### Storage

- Table: `IdempotencyRecords` in HostDb
- Expiry: 24 hours
- Unique index on `IdempotencyKey`
- Request hash ensures same key + different body = different operation

## When to Use

- Payment processing (required)
- User registration (optional; database constraints also help)
- Any mutation with significant side effects
- Operations that may be retried due to network issues

## When Not to Use

- GET requests (idempotent by definition)
- Read operations
- Operations with natural idempotency (e.g., "set status to X")
