# Host API Overview

## Base prefix

All host endpoints use the prefix defined in `CustomRouter.HostRouting.HostEndpointPrefix`: **`API/V01/Host`**.

Example base URL: `https://localhost:7292/API/V01/Host`.

## Authentication

- Use **HostScheme** JWT. Obtain via `POST API/V01/Host/Auth/Login`; refresh via `POST API/V01/Host/Auth/RefreshToken`.
- Send token in `Authorization: Bearer <token>` (or query param `access_token` where supported, e.g. SignalR).

## Controllers and routes

| Controller | Route suffix | Purpose |
|------------|--------------|---------|
| HostAuthController | `/Auth` | Login, RefreshToken, Logout, LogoutFromAllDevices, ResetPassword, ChangePassword, SendResetPasswordCode, CurrentUserInfo |
| TenantsController (host) | `/Tenants` | Tenant CRUD, by-domain, by-code, stats, modules, branding, feature flags, metrics, health-summary, export |
| HostTenantUsersController | `/Tenants/{tenantId}/Users` | Manage that tenant's users (list, create, edit, deactivate, set password, roles, claims) |
| HostUserController | `/Users` | Host user CRUD, EditUser, ChangePassword, SetPassword, UnActivate |
| HostRolesController | `/Roles` | Host roles list, get, create, update, delete; user roles/claims get and modify |
| HostPlansController | `/Plans` | Plans list, get, create, update, delete, UnActivate |
| HostSupportPlansController | `/SupportPlans` | Support plans list, get, create, update, delete, UnActivate |
| HostFeatureFlagsController | `/FeatureFlags` | Flag definitions list, get, create, update, archive; get/update tenants per flag |
| HostBrandingController | `/Branding` | Get, Upsert, UploadLogo (platform branding) |

## Route constants

- **Host routing**: `CustomRouter.HostRouting.HostEndpointPrefix` = `API/V01/Host`.
- **Sub-routes**: `CustomRouter.HostRouting.HostAuthRouting.*`, `HostTenantsRouting.*`, `HostUserRouting.*`, `HostRolesRouting.*`, `HostPlansRouting.*`, `HostSupportPlansRouting.*`, `HostFeatureFlagsRouting.*`, `HostBrandingRouting.*` (in `Platform.Data/AppMetaData/CustomRouter.cs`).

## Tenant resolution

Requests whose path starts with `HostEndpointPrefix` are treated as host requests; `TenantResolutionMiddleware` skips tenant resolution for them. For `/Tenants/{tenantId}/Users/*`, `HostTenantUsersContextMiddleware` sets the current tenant from the route so handlers use that tenant's database.

## References

- Feature docs: [docs/features/](../features/) (host-tenants-api, host-users, host-roles-and-claims, host-plans, host-support-plans, host-feature-flags-definitions, host-branding, host-tenant-users).
- [Host identity system](../features-explanation-docs/host-identity-system.md).
