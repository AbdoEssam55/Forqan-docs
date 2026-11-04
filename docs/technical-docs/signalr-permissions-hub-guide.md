# SignalR Permissions Hub Guide

Short, action-oriented reference for the permissions SignalR hub and notifications.

---

## Hub Endpoint and Auth

1. **Path:** `/hubs/permissions` (e.g. `https://localhost:7292/hubs/permissions`).
2. **Auth:** Hub uses `[Authorize(AuthenticationSchemes = "HostScheme,TenantScheme")]`. Client must send a valid host or tenant JWT (e.g. via `access_token` query parameter; middleware copies it to `Authorization: Bearer`).
3. **Group:** On connect, the hub adds the connection to group `user-{userId}` where `userId` is from the JWT claim `UserId`.

---

## Notifying a User That Their Permissions Changed

1. Inject `IPermissionsNotificationService`.
2. After successfully updating the user's roles or claims, call:
   ```csharp
   await permissionsNotificationService.NotifyPermissionsChangedAsync(userId, cancellationToken);
   ```
3. Use the **string** user id (e.g. `userId.ToString()` for a Guid). The service sends the event `"PermissionsChanged"` to the group `user-{userId}`.

---

## Adding Token Extraction for a New Scheme

If you add another JWT scheme that should be able to connect to the hub:

1. Configure that scheme's `JwtBearerOptions` with `OnMessageReceived` that sets `context.Token` from `context.Request.Query["access_token"]` when `context.Request.Path` starts with `/hubs`.
2. Add the scheme name to `PermissionsHub`: `[Authorize(AuthenticationSchemes = "HostScheme,TenantScheme,YourScheme")]`.

---

## Key Files

- Hub: `Alforqan.Api/Hubs/PermissionsHub.cs`
- Notification service: `Alforqan.Api/Services/PermissionsNotificationService.cs`
- Interface: `Alforqan.Service.Abstractions/Identity/IPermissionsNotificationService.cs`
- Call sites: `AuthorizationUserService` (after ModifyUserRoles and ModifyUserClaims).
- Token extraction for `/hubs`: `SignalRTokenExtractionMiddleware`; `OnMessageReceived` in HostScheme (ModuleSharedDependencies) and TenantScheme (Program.cs Configure).
