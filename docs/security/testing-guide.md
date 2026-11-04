# Security Testing Guide

## Unit Tests

Run: `dotnet test Tests/Alforqan.SecurityTests/`

### Race Condition Tests

- `RaceConditionPreventionTests`: Verifies RefreshToken.IsUsed excludes from IsActive, CryptoHelper determinism
- Add tests for DbUpdateException handling in create handlers
- Add tests for IdempotencyService Get/Store flow

## Integration Tests (Concurrent Requests)

To simulate 10+ concurrent requests:

```csharp
var tasks = Enumerable.Range(0, 10)
    .Select(_ => httpClient.PostAsync("/api/v01/authentication/register", content))
    .ToList();
var results = await Task.WhenAll(tasks);
// Verify: only one 201, rest 409 or cached 200
```

Use `[Collection("Integration")]` and a shared test database. Ensure tenant context is set.

## Load Tests (K6)

Create `tests/load/race-condition.js`:

```javascript
import http from 'k6/http';
import { check } from 'k6';

export const options = {
  vus: 100,
  duration: '30s',
};

export default function () {
  const payload = JSON.stringify({
    email: 'concurrent@test.com',
    username: 'concurrent',
    password: 'Test123!',
    confirmPassword: 'Test123!',
    firstName: 'Test',
    lastName: 'User',
  });
  const res = http.post('http://localhost:5000/api/v01/authentication/register',
    payload, { headers: { 'Content-Type': 'application/json' } });
  check(res, { 'status 201 or 409': (r) => r.status === 201 || r.status === 409 });
}
```

Run: `k6 run tests/load/race-condition.js`

## Checklist

- [ ] Unit tests pass
- [ ] No duplicate users created in concurrent registration test
- [ ] Refresh token reuse returns 401
- [ ] Idempotency key returns cached response on duplicate
- [ ] Load test completes without data corruption
