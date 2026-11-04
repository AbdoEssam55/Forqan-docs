# Session Invalidation on Role/Claim Change Feature

## Overview

When an administrator saves changes to a user's roles or claims, that user's **refresh tokens** are revoked so that their next token refresh fails. The user must sign in again and receives a new JWT with the updated claims. This applies to both **host users** and **tenant (Alforqan) users**.

## Problem Statement

Claims are carried in the JWT. If an admin changes a user's roles or claims while that user is offline or has the app open in another tab:

- The user's existing JWT still contains the old claims until it expires or is refreshed.
- If the user only refreshes the token (without re-login), the refresh flow issues a new JWT—but the backend does not re-fetch claims from the database on refresh; it uses the same identity. So the user could keep using an outdated JWT until expiry.
- Real-time update via SignalR works only when the user is connected; if they closed the app while the admin made changes, they never receive the event.

Revoking all refresh tokens for that user when roles/claims are modified ensures that the next refresh fails and the user must log in again, at which point they receive a new JWT with the correct claims.

## Architecture and Design Decisions

### Where Modifications Happen

- **Tenant users** (whether modified from the tenant app or from the host app when editing a tenant user): Handled by `AuthorizationCommandHandler` (tenant). It uses the tenant `AlforqanDbContext`; the same handler and tenant DB are used when the host calls the tenant user API (via `HostTenantUsersContextMiddleware`).
- **Host users**: Handled by `HostAuthorizationCommandHandler`; uses the host `HostIdentityDbContext`.

No change was made to the shared `AuthorizationUserService` or to `NotifyPermissionsChangedAsync`. Revocation is done in the **command handlers** after a successful modify, where the correct DB and scope (tenant vs host) are already available.

### Why Revoke in Handlers Instead of in the Service

The service is shared and does not know whether the current context is host or tenant. The handlers have access to the correct `IRefreshTokenRepo` (tenant) or `IHostRefreshTokenRepo` (host) for the DB where the user's refresh tokens are stored. Revoking in the handlers keeps the service independent of refresh-token storage and avoids introducing a second abstraction (e.g. "revoke tokens for user in current context") that would duplicate the handler’s context knowledge.

### Best-Effort Revocation

Revocation is wrapped in try/catch. If it fails (e.g. database timeout), the failure is logged and the role/claim update response is still returned as success. The existing SignalR `NotifyPermissionsChangedAsync` remains; revocation adds a guarantee for the offline case. Not failing the API response avoids blocking admins from saving roles/claims when the revocation store is temporarily unavailable.

## How It Works

### Tenant Path

1. Admin modifies a tenant user's roles or claims (via tenant app or host app).
2. `AuthorizationCommandHandler` handles `ModifyUserRolesCommand` or `ModifyUserClaimsCommand`, calls `AuthorizationUserService.ModifyUserRolesAsync` or `ModifyUserClaimsAsync` with the tenant `UserManager` and `DbContext`.
3. On success, the handler calls `refreshTokenRepo.RevokeAllAsync(request.UserId!, cancellationToken)` (tenant-scoped `IRefreshTokenRepo`).
4. The handler returns the service response. Any revocation failure is logged and does not change the response.

### Host Path

1. Admin modifies a host user's roles or claims.
2. `HostAuthorizationCommandHandler` handles `ModifyHostUserRolesCommand` or `ModifyHostUserClaimsCommand`, calls the same shared service with the host `UserManager` and `HostIdentityDbContext`.
3. On success, the handler calls `hostRefreshTokenRepo.RevokeAllAsync(request.UserId!, cancellationToken)`.
4. The handler returns the service response. Any revocation failure is logged and does not change the response.

### Interaction with SignalR

`AuthorizationUserService` still calls `permissionsNotificationService.NotifyPermissionsChangedAsync(userId)` after committing. So:

- **User online:** They receive the SignalR event, refresh their token, and the UI updates.
- **User offline:** Their refresh tokens are revoked; when they return and the app tries to refresh, the refresh fails and they are redirected to login, where they receive a new JWT with the updated claims.

## Related Files

### Backend

- `Alforqan.Core/Features/Identity/Tenant/Authorization/Commands/Handlers/AuthorizationCommandHandler.cs` — Injects `IRefreshTokenRepo`; revokes after successful `ModifyUserRolesAsync` and `ModifyUserClaimsAsync`.
- `Alforqan.Core/Features/Identity/Host/Authorization/Commands/Handlers/HostAuthorizationCommandHandler.cs` — Injects `IHostRefreshTokenRepo`; revokes after successful host role/claims modify.
- `Platform.Infrastructure/Abstractions/Identity/RefreshTokens/IRefreshTokenRepo.cs` — Defines `RevokeAllAsync(string userId, ...)`.
- `Platform.Infrastructure/Abstractions/Identity/RefreshTokens/IHostRefreshTokenRepo.cs` — Extends `IRefreshTokenRepo` for host DB.

### Related Documentation

- [Real-Time Permissions Updates Feature](real-time-permissions-updates-feature.md) — SignalR notification when roles/claims change.
- [Tenant Session Invalidation Feature](tenant-session-invalidation-feature.md) — Revocation when tenant status becomes Suspended/Inactive (all users in the tenant).

## Summary

When an admin saves a user's roles or claims (host or tenant), that user's refresh tokens are revoked in the same DB the handler uses (tenant DB for tenant users, host DB for host users). The next refresh fails and the user must log in again to get an updated JWT. This complements SignalR-based real-time updates and covers the case where the user is offline when their permissions change.
