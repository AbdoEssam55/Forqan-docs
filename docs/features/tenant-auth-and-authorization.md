# Tenant Auth and Authorization

## Overview

Tenant authentication and authorization cover login, refresh token, logout, password reset/change, and management of **tenant** roles and claims (Admin, Teacher, Student, Parent). Tenant users are stored in the tenant database; host users are separate (see host-identity-system). The same authentication base builder pattern is used for both host and tenant; tenant uses TenantScheme and tenant DbContext.

## Auth endpoints (tenant)

Typically under `/API/V01/Auth` or `/API/V01/AppUser` (or a dedicated Auth controller): Login, RefreshToken, Logout, LogoutFromAllDevices, SendResetPasswordCode, ResetPassword, ChangePassword. Change password updates credentials only (no notification email). Routes and controller location may follow the auth builder (e.g. tenant auth controller that uses the tenant authentication handlers).

## Authorization (roles and claims)

- **AuthorizationController** (`/API/V01/Authorization` or similar): Tenant roles list, get by ID, create, update, delete; get user roles, modify user roles; get user claims, modify user claims. Uses shared authorization services with tenant UserManager and RoleManager (from tenant DbContext).
- **AppUserController** (`/API/V01/AppUser`): Tenant user CRUD, list, get by ID, create, update, deactivate, change password, set password by admin. Uses tenant DbContext.

All require tenant authentication; role/claim changes may trigger real-time permission updates via SignalR (PermissionsChanged).

## Related documentation

- [Authentication base builder pattern](../features-explanation-docs/authentication-base-builder-pattern-feature.md) – shared auth logic for tenant and host
- [Shared authorization services](../features-explanation-docs/shared-authorization-services-feature.md) – roles and claims handlers
- [Real-time permissions updates](../features-explanation-docs/real-time-permissions-updates-feature.md) – SignalR and token refresh
- [Protected system roles](../features-explanation-docs/protected-system-roles-feature.md) – tenant protected roles (e.g. Admin)
- [Host identity system](../features-explanation-docs/host-identity-system.md) – contrast with host auth

## Controllers

`Alforqan.Api/Controllers/Identity/Authorization/AuthorizationController.cs`  
`Alforqan.Api/Controllers/Identity/AppUsers/AppUserController.cs`  
Auth controller (tenant login/refresh/logout) – see Core/Features/Identity/Tenant/Authentication and API routing.
