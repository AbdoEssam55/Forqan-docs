# Real-Time Permissions Updates Feature

## Problem Statement

When an administrator changes a user's roles or claims (e.g. grants or revokes "ModifyUserRolesClaim"), the affected user may still have the app open. Without a mechanism to push updates, the user would continue to see UI and make requests based on their old JWT claims until they refresh the page or log in again. That leads to:

- Confusing UX (buttons or tabs disappear only after reload)
- Risk of the user attempting actions the backend will reject (401/403)

The UI must reflect role/claim changes as soon as they are saved, without requiring a full page reload or re-login.

## Architecture Overview

The solution uses **SignalR** to notify the affected user that their permissions have changed. The frontend then refreshes the JWT (via the refresh-token flow) and updates in-memory permission state so all components using `useClaims` re-render with the new claims.

### Components

- **PermissionsHub** (`/hubs/permissions`): A SignalR hub that accepts connections from both host and tenant users. On connect, the user is added to a group `user-{userId}` (userId from JWT). No application methods are invoked by the client; the hub is used only for server-to-client push.
- **PermissionsNotificationService**: Implements `IPermissionsNotificationService`. When `NotifyPermissionsChangedAsync(userId)` is called, it sends the event `"PermissionsChanged"` to the group `user-{userId}`. SignalR delivers this to all connections in that group (typically one per logged-in session).
- **AuthorizationUserService** (shared): After successfully committing `ModifyUserRolesAsync` or `ModifyUserClaimsAsync`, it calls `permissionsNotificationService.NotifyPermissionsChangedAsync(userId)`. This runs for both host and tenant API paths because the same service is used by host and tenant handlers.

### Flow

1. An admin (host or tenant) changes a user's roles or claims via the API.
2. The shared `AuthorizationUserService` commits the transaction, then calls `NotifyPermissionsChangedAsync(userId)`.
3. `PermissionsNotificationService` sends `"PermissionsChanged"` to the SignalR group `user-{userId}`.
4. The affected user's browser has an active SignalR connection to the hub (established with their host or tenant JWT). That connection is in the group `user-{userId}`.
5. The client receives the `"PermissionsChanged"` event. The frontend **PermissionsProvider** then refreshes the appropriate token (host or tenant) via the refresh-token API and updates stored tokens. It then triggers a permission refresh so all `useClaims` consumers re-render with the new claims.

### Host vs Tenant

- **Same hub, same event.** Both host and tenant users connect to `/hubs/permissions`. The hub authorizes with `HostScheme,TenantScheme` so either JWT type is accepted. The backend does not distinguish who is being notified; it only sends to `user-{userId}`.
- **Backend:** When a **tenant** user's roles/claims are modified (via tenant API), the same `AuthorizationUserService.ModifyUserRolesAsync` / `ModifyUserClaimsAsync` runs and calls `NotifyPermissionsChangedAsync(tenantUserId)`. So the tenant user is notified. No backend change was required for tenant *sending*; only hub auth had to allow tenant *connections*.
- **Frontend:** The app maintains up to two SignalR connections when both tokens exist: one with the host token, one with the tenant token. Each connection is placed in the group for that token's user ID. When the event is received on the host connection, the frontend refreshes the host token; when received on the tenant connection, it refreshes the tenant token and stores it. In both cases it then triggers a permission refresh so `useClaims("host")` or `useClaims("tenant")` re-renders with the new claims.

## Design Decisions

### Why SignalR and not polling?

Polling would require the client to periodically call an endpoint to check for permission changes, and would delay updates by at least the polling interval. SignalR gives immediate push as soon as the backend commits the change.

### Why notify by user ID and not by connection ID?

The backend does not track which connection belongs to which user except at connect time, when the hub adds the connection to `user-{userId}`. Notifying by user ID allows a single service call to reach all of that user's connections (e.g. multiple tabs) without the backend maintaining a user-to-connection map.

### Why refresh the token instead of pushing new claims?

The rest of the app (and the refresh-token API) is built around JWTs. Pushing raw claims would require a separate store and logic to merge with the existing token. Refreshing the token yields a new JWT with the updated claims and keeps a single source of truth (the token) for authorization and UI gating.
