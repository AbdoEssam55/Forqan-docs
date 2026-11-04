# Attack Scenarios and Mitigations

Real-world scenarios that exploit race conditions and how we prevent them.

## Scenario 1: Duplicate User Registration

**Attack**: Attacker sends 100 concurrent registration requests with the same email. Without protection, multiple accounts could be created.

**Mitigation**:
- ASP.NET Identity enforces unique NormalizedEmail/NormalizedUserName
- Database unique constraint causes second insert to fail with DbUpdateException
- RegisterUserBuilder catches exception and returns "Email already exists"
- Optional: Idempotency-Key for additional deduplication

## Scenario 2: Refresh Token Reuse (Token Theft)

**Attack**: Attacker steals a refresh token. Legitimate user and attacker both use it. Without protection, both could get new tokens.

**Mitigation**:
- Refresh token loaded with `UPDLOCK, ROWLOCK` (pessimistic lock)
- `IsUsed` flag set atomically in same transaction
- Second request blocks until first completes; then sees IsUsed=true and is rejected
- Returns generic "Invalid or revoked token" to prevent enumeration

## Scenario 3: Password Reset Code Reuse

**Attack**: User requests password reset. Attacker intercepts the code. Both use it simultaneously to change password.

**Mitigation**:
- User loaded with `UPDLOCK` (pessimistic lock)
- `ResetPasswordCodeUsed` checked before validation
- Set to true atomically when reset succeeds
- Second request blocked; first completes; second sees code already used

## Scenario 4: Double Payment

**Attack**: User clicks "Pay" multiple times quickly, or network retries send duplicate requests. Without protection, user could be charged twice.

**Mitigation** (when payments are implemented):
- `[RequiresIdempotency]` on payment endpoint
- Client sends Idempotency-Key per logical payment
- First request processes; second returns cached success without re-charging
- Pessimistic lock on balance check to prevent over-spending

## Scenario 5: Lost Update (Concurrent Edit)

**Attack**: User A and User B both edit the same student record. A saves first, then B saves. A's changes are lost.

**Mitigation**:
- Student entity has RowVersion
- When B saves, EF detects RowVersion mismatch (A already changed it)
- DbUpdateConcurrencyException thrown
- Return 409: "Record was modified by another user. Please refresh and try again."
- User B refreshes, sees A's changes, edits again if needed
