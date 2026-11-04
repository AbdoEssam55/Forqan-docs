# Per-Tenant Rate Limiting Feature

## Overview

The per-tenant rate limiting feature ensures that rate limits are applied per tenant (by tenant ID) for tenant-scoped endpoints, instead of per IP address. This provides resource isolation, fair usage, and the ability to throttle or block specific tenants without affecting others.

## Problem Statement

With IP-based rate limiting:

- Users from different tenants behind a shared IP (e.g. office NAT) share the same rate limit bucket
- One abusive tenant can exhaust the shared bucket and affect other tenants
- There is no tenant-level resource isolation or fair usage
- It is not possible to throttle or block a specific tenant without affecting others

## Architecture and Design Decisions

### Partition Key Strategy

**Tenant-scoped endpoints** (tenant resolved by `TenantResolutionMiddleware`):

- Use **tenant ID** as the rate limit partition key
- Each tenant gets its own rate limit bucket
- Fallback to IP if tenant is not resolved (defensive programming)

**Host endpoints** (no tenant):

- Keep **IP-based** partitioning
- Examples: HostLogin, HostRefreshToken, TenantModuleToggle

**Public tenant endpoints** (tenant resolved by Host header before authentication):

- Use **tenant-based** partitioning
- Examples: TenantLogin, Register, SendResetPasswordCode, ResetPassword

### IRateLimitPartitionKeyResolver

**Why:** Centralize partition key resolution so policies can choose tenant-based or IP-based behavior without duplicating logic.

**Benefits:**

- Single place to change fallback behavior
- Easy to test and mock
- Policies in `Program.cs` stay declarative

**Implementation:**

- Interface: `GetPartitionKey(HttpContext context, string policyName)` returns a string
- Implementation uses `ITenantContext`: if tenant is resolved, return `CurrentTenant.Id.ToString()`; otherwise return `RemoteIpAddress` or `"unknown"`
- Registered as scoped in `ModuleServiceDependencies`

**Trade-offs:**

- Resolver is invoked once per request per policy that runs (minimal overhead)
- Tenant must be resolved before rate limiter runs (middleware order ensures this)

### Middleware Order

`TenantResolutionMiddleware` runs **before** `UseRateLimiter()`, so the tenant is already in `HttpContext.Items` when the rate limiter computes the partition key. No change to middleware order was required.

## How It Works

### Flow

1. Request hits tenant-scoped endpoint (e.g. tenant login).
2. `TenantResolutionMiddleware` runs and resolves tenant by Host (or header); stores tenant in `HttpContext.Items`.
3. Rate limiter runs; policy delegate is invoked.
4. Policy gets `IRateLimitPartitionKeyResolver` from `HttpContext.RequestServices`.
5. Resolver returns `tenant.Id.ToString()` if tenant is resolved, else IP (or `"unknown"`).
6. ASP.NET Core rate limiter uses that string as the partition key (one bucket per key).
7. Request is allowed or rejected (429) based on that bucket’s limits.

### Policies Updated to Tenant-Based Partitioning

| Policy                 | Endpoint(s)           | Limit (unchanged)     | Partition key   |
|------------------------|------------------------|----------------------|-----------------|
| TenantLogin            | Tenant login           | 10 / 2 min           | TenantId        |
| TenantRefreshToken     | Tenant refresh token   | 10 / 15 min          | TenantId        |
| Register               | Tenant registration    | 3 / 15 min           | TenantId        |
| SendResetPasswordCode  | Send reset code        | 3 / 1 hour           | TenantId        |
| ResetPassword          | Reset password         | 5 / 15 min           | TenantId        |

### Policies Left as IP-Based

| Policy            | Endpoint(s)      | Partition key |
|-------------------|------------------|---------------|
| HostLogin         | Host login       | IP            |
| HostRefreshToken  | Host refresh     | IP            |
| TenantModuleToggle| Host module APIs | User/IP       |

## Security Considerations

1. **Fallback to IP:** If tenant resolution fails, the resolver falls back to IP so rate limiting still applies and cannot be bypassed by missing tenant.
2. **Null handling:** Resolver returns a non-null string (e.g. `"unknown"`) when both tenant and IP are unavailable.
3. **Host endpoints:** Host login and host refresh stay IP-based to prevent brute force without relying on tenant context.
4. **Isolation:** Tenant-based keys ensure one tenant’s abuse does not consume another tenant’s quota.

## Performance Considerations

1. **Service resolution:** `GetRequiredService<IRateLimitPartitionKeyResolver>()` is a single scoped resolve per policy evaluation.
2. **Tenant context:** Tenant is already in `HttpContext.Items`; `ITenantContext.CurrentTenant` does not hit the database.
3. **Partition key:** Computed once per request per policy; ASP.NET Core rate limiter uses it for in-memory bucket lookup.

## Related Files

- `Alforqan.Service/Abstractions/Tenants/IServices/IRateLimitPartitionKeyResolver.cs` – interface
- `Alforqan.Service/Implementation/Tenants/Services/RateLimitPartitionKeyResolver.cs` – implementation
- `Alforqan.Service/ModuleServiceDependencies.cs` – DI registration
- `Alforqan.Api/Program.cs` – rate limiter policy configuration
- `Alforqan.Shared/HelperClasses/ConstantsHelper.cs` – policy names and comments

## Testing Considerations

1. **Unit tests:** `RateLimitPartitionKeyResolver` with mocked `ITenantContext`: tenant present → tenant ID; no tenant → IP; null IP → `"unknown"`.
2. **Integration tests:** Call tenant login (or other tenant-scoped endpoint) from two different tenant contexts and confirm separate rate limit buckets (e.g. exhaust limit for tenant A, tenant B still has quota).
3. **Edge cases:** Request to tenant endpoint without resolved tenant (e.g. misconfigured host) should still be rate limited by IP.

## Future Enhancements

- **Plan-based limits:** Different permit/window per tenant plan (e.g. Basic vs Premium).
- **Dynamic limits:** Adjust limits per tenant from config or database.
- **Metrics:** Count rate limit hits per tenant for monitoring and abuse detection.
