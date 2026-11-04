# Tenant Session Invalidation Feature

## Overview

The tenant session invalidation feature ensures that when a tenant's status changes to `Suspended` or `Inactive`, all existing JWT tokens for that tenant's users are immediately invalidated. This provides security and compliance - suspended/inactive tenants should not have access, even with valid tokens.

## Problem Statement

In a multi-tenant system, JWT tokens are stateless and remain valid until their expiration time. Without proper session invalidation:

- Users with valid tokens can continue accessing the system even after their tenant is suspended or deactivated
- Security boundaries are violated - suspended tenants should not have access
- Compliance issues arise - inactive tenants should not be able to access their data
- Token refresh can generate new access tokens, bypassing status restrictions

## Architecture and Design Decisions

### JWT Token Validation During Authentication

**Why:** Add tenant status check during JWT token validation using `OnTokenValidated` event handler.

**Benefits:**
- **Immediate Effect**: Token validation happens on every request, so status changes take effect immediately
- **No Token Revocation Needed**: JWT tokens are stateless, so we reject them during validation rather than trying to revoke them
- **Performance**: Minimal overhead - only checks tenant status when token is validated (tenant already resolved by middleware)
- **Security**: Status check happens AFTER token signature validation, ensuring only valid tokens are checked

**Implementation:**
- `OnTokenValidated` event handler in `TenantScheme` JWT configuration
- Uses `ITenantContext` to get current tenant (already resolved by `TenantResolutionMiddleware`)
- Calls `ITenantStatusValidationService.IsTenantActiveAsync()` to check status
- Throws `SecurityTokenException` with localized message if tenant is not active

**Trade-offs:**
- Requires database lookup on every authenticated request (mitigated by tenant already being in HttpContext)
- Status check happens after token signature validation (acceptable - signature validation is fast)

### Refresh Token Revocation on Status Change

**Why:** When tenant status changes to Suspended/Inactive, revoke all refresh tokens for that tenant's users.

**Benefits:**
- **Prevents New Tokens**: Revoking refresh tokens prevents users from obtaining new access tokens
- **Complete Invalidation**: Both access tokens (via validation) and refresh tokens (via revocation) are invalidated
- **Immediate Effect**: Refresh token revocation happens synchronously during tenant update

**Implementation:**
- `TenantService.EditAsync()` checks if status changed from Active to Suspended/Inactive
- Calls `ITenantRefreshTokenRevocationService.RevokeAllTokensForTenantAsync()`
- Service creates `AppDbContext` with tenant's connection string
- Gets all user IDs from tenant DB and revokes their refresh tokens
- Errors are logged but don't fail tenant update (JWT validation will still catch inactive tenants)

**Trade-offs:**
- Requires access to tenant DB (uses connection string from Host DB)
- May take time for tenants with many users (acceptable - happens during admin update)
- Errors don't fail tenant update (acceptable - JWT validation provides backup)

### Frontend Error Detection and Handling

**Why:** Frontend detects tenant status errors and shows appropriate message, redirects to login.

**Benefits:**
- **User Experience**: Users see clear message about tenant status
- **Immediate Feedback**: No retry loops for tenant status errors
- **Security**: Tokens are cleared immediately on tenant status error

**Implementation:**
- HTTP client error interceptor checks for 401 responses
- Detects tenant status errors by checking error message for keywords ("suspended", "deactivated", "inactive", "tenant account")
- Shows localized toast message
- Clears tokens and redirects to login
- Prevents token refresh retry for tenant status errors

**Trade-offs:**
- Relies on error message content (acceptable - backend returns localized messages)
- Hardcoded keywords in frontend (acceptable - backend messages are consistent)

## How It Works

### Flow Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│ 1. Tenant Status Changed (Admin updates tenant status)         │
└────────────────────┬──────────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│ 2. TenantService.EditAsync()                                    │
│    - Detects status change to Suspended/Inactive               │
│    - Calls ITenantRefreshTokenRevocationService                 │
└────────────────────┬──────────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│ 3. Revoke All Refresh Tokens                                    │
│    - Get tenant connection string                               │
│    - Create AppDbContext for tenant DB                          │
│    - Get all user IDs                                           │
│    - Revoke refresh tokens for each user                        │
└────────────────────┬──────────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│ 4. User Makes Request with Valid JWT Token                      │
└────────────────────┬──────────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│ 5. TenantResolutionMiddleware                                   │
│    - Resolves tenant by domain                                  │
│    - Stores tenant in HttpContext                               │
└────────────────────┬──────────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│ 6. JWT Token Validation (OnTokenValidated)                      │
│    - Validates token signature                                  │
│    - Gets tenant from ITenantContext                            │
│    - Checks tenant status via ITenantStatusValidationService    │
└────────────────────┬──────────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│ 7. Status Check Result                                          │
│    ┌──────────────────┐         ┌──────────────────┐           │
│    │ Status = Active  │         │ Status != Active │           │
│    └────────┬─────────┘         └────────┬─────────┘           │
│             │                            │                      │
│             │                            ▼                      │
│             │              ┌──────────────────────────────┐   │
│             │              │ Throw SecurityTokenException  │   │
│             │              │ (401 Unauthorized)            │   │
│             │              └────────────┬───────────────────┘   │
│             │                           │                      │
│             ▼                           ▼                      │
│    ┌──────────────────┐      ┌──────────────────────────┐     │
│    │ Request Proceeds │      │ Frontend Detects Error    │     │
│    └──────────────────┘      │ - Shows toast message    │     │
│                               │ - Clears tokens          │     │
│                               │ - Redirects to login     │     │
│                               └──────────────────────────┘     │
└─────────────────────────────────────────────────────────────────┘
```

### Sequence Diagram

```
Admin                    TenantService          RevocationService      TenantDB
  │                            │                        │                 │
  │──Update Tenant Status─────>│                        │                 │
  │                            │                        │                 │
  │                            │──Revoke Tokens────────>│                 │
  │                            │                        │                 │
  │                            │                        │──Get Users─────>│
  │                            │                        │<──User IDs──────│
  │                            │                        │                 │
  │                            │                        │──Revoke Tokens─>│
  │                            │                        │<──Success───────│
  │                            │<──Success──────────────│                 │
  │<──Success──────────────────│                        │                 │
  │                            │                        │                 │
User                    HttpClient          JWT Validation      TenantStatusService    HostDB
  │                            │                        │                        │
  │──API Request──────────────>│                        │                        │
  │                            │                        │                        │
  │                            │──Validate Token───────>│                        │
  │                            │                        │                        │
  │                            │                        │──Check Status─────────>│
  │                            │                        │<──Status: Suspended────│
  │                            │                        │                        │
  │                            │<──SecurityTokenException───────────────────────│
  │                            │                        │                        │
  │<──401 Unauthorized─────────│                        │                        │
  │                            │                        │                        │
  │──Detect Tenant Error───────│                        │                        │
  │──Show Toast────────────────│                        │                        │
  │──Clear Tokens──────────────│                        │                        │
  │──Redirect to Login─────────│                        │                        │
```

## Integration with Existing Systems

### Tenant Resolution Middleware

**Integration Point:** `TenantResolutionMiddleware` runs BEFORE authentication, resolving tenant and storing it in `HttpContext.Items`. The JWT validation handler uses `ITenantContext` to access the resolved tenant.

**Why This Works:**
- Tenant is resolved before authentication, so it's available during token validation
- No additional tenant resolution needed in JWT validation handler
- Performance: Tenant is already in memory (HttpContext), no additional DB lookup

### Error Handler Middleware

**Integration Point:** `ErrorHandlerMiddleware` already handles `SecurityTokenException` and returns 401 Unauthorized with localized message.

**Why This Works:**
- No changes needed to error handler
- `SecurityTokenException` is already handled correctly
- Localized messages are already supported

### Frontend HTTP Client

**Integration Point:** `HttpClient` error interceptor detects tenant status errors and handles them appropriately.

**Why This Works:**
- Uses existing error handling infrastructure
- Detects tenant status errors by message content
- Integrates with existing token refresh logic

## Security Considerations

### Token Validation Order

**Critical:** Tenant status check happens AFTER token signature validation. This ensures:
- Only valid tokens are checked (prevents DoS attacks with invalid tokens)
- Signature validation is fast (cryptographic operation)
- Status check is efficient (tenant already in HttpContext)

### Error Messages

**Security:** Error messages don't leak sensitive information:
- Generic message: "Your tenant account has been suspended or deactivated"
- No tenant ID or internal details exposed
- Localized messages for better UX

### Logging

**Security:** Tenant status validation failures are logged for security monitoring:
- Helps identify unauthorized access attempts
- Tracks tenant status changes
- Provides audit trail

### Refresh Token Revocation

**Security:** Refresh token revocation prevents new access tokens:
- Even if access token is still valid, refresh token is revoked
- Prevents token refresh from bypassing status restrictions
- Complete invalidation of all tokens

## Performance Considerations

### Database Load

**Impact:** Tenant status check happens on every authenticated request.

**Mitigation:**
- Tenant is already resolved by middleware (in HttpContext)
- Status check uses `ITenantRepo.GetByIdAsync()` (indexed lookup by ID)
- Consider caching tenant status in memory with short TTL (future enhancement)

### Refresh Token Revocation

**Impact:** Revoking tokens for all users in a tenant may take time for large tenants.

**Mitigation:**
- Happens during admin update (not user-facing)
- Errors don't fail tenant update (JWT validation provides backup)
- Can be optimized with batch operations (future enhancement)

### Frontend Error Detection

**Impact:** Minimal - only checks error message content (string comparison).

**Mitigation:**
- No additional API calls
- No performance impact on normal requests

## Related Files

### Backend

- `Backend/Alforqan.Service/Abstractions/Tenants/IServices/ITenantStatusValidationService.cs` - Interface for tenant status validation
- `Backend/Alforqan.Service/Implementation/Tenants/Services/TenantStatusValidationService.cs` - Implementation of tenant status validation
- `Backend/Alforqan.Service/Abstractions/Tenants/IServices/ITenantRefreshTokenRevocationService.cs` - Interface for refresh token revocation
- `Backend/Alforqan.Service/Implementation/Tenants/Services/TenantRefreshTokenRevocationService.cs` - Implementation of refresh token revocation
- `Backend/Alforqan.Shared/ModuleSharedDependencies.cs` - JWT configuration with `OnTokenValidated` handler
- `Backend/Alforqan.Service/Implementation/Tenants/Services/TenantService.cs` - Tenant update logic with token revocation
- `Backend/Alforqan.Api/Middlewares/ErrorHandlerMiddleware.cs` - Error handling (already supports `SecurityTokenException`)

### Frontend

- `Frontend/alforqan/src/lib/api/http-client.ts` - HTTP client with tenant status error detection
- `Frontend/alforqan/src/messages/en.json` - English localization keys
- `Frontend/alforqan/src/messages/ar.json` - Arabic localization keys

### Documentation

- `docs/multi-tenant-platform-scope-and-roadmap.md` - Roadmap document (section 8.8)

## Testing Considerations

### Unit Tests

- `TenantStatusValidationService`: Test active/inactive/suspended status checks
- `TenantRefreshTokenRevocationService`: Test token revocation for tenant users
- `TenantService.EditAsync`: Test refresh token revocation on status change

### Integration Tests

- JWT validation with inactive tenant should reject token
- Tenant status change should revoke refresh tokens
- Frontend should handle tenant status errors gracefully

## Future Enhancements

### Caching Tenant Status

**Enhancement:** Cache tenant status in memory with short TTL (e.g., 30 seconds) to reduce database load.

**Considerations:**
- Cache invalidation on tenant status change
- TTL should be short enough to catch status changes quickly
- Consider distributed cache for multi-instance deployments

### Batch Token Revocation

**Enhancement:** Optimize refresh token revocation with batch operations for large tenants.

**Considerations:**
- Use SQL batch update instead of per-user revocation
- Handle errors gracefully (some users may fail)
- Log batch operation results

### Token Revocation List (Redis)

**Enhancement:** Maintain a token revocation list in Redis for immediate invalidation without waiting for token expiry.

**Considerations:**
- Requires Redis infrastructure
- Adds complexity but provides immediate invalidation
- Useful for high-security scenarios

## Summary

The tenant session invalidation feature ensures that suspended or inactive tenants cannot access the system, even with valid tokens. It combines JWT token validation (for immediate effect) with refresh token revocation (for complete invalidation) to provide comprehensive security and compliance.
