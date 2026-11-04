# Host Tenant Users Feature

## Problem statement

Platform administrators need to manage **tenant users** (Admin, Teacher, Student, Parent, etc.) from the host app without logging into each tenant. Use cases: create a tenant admin during onboarding, reset a tenant user's password, assign roles/claims, or deactivate a user. The host already has the tenant list and tenant context; it needs an API that operates on a specific tenant's user store.

## Architecture and design

- **Route**: `/API/V01/Host/Tenants/{tenantId}/Users` – every request includes the tenant ID in the path. This makes the "current tenant" explicit and avoids relying on headers or body for tenant context.
- **Middleware**: `HostTenantUsersContextMiddleware` runs after routing. It matches paths under Host/Tenants/{tenantId}/Users, loads the tenant by ID from Host DB (via `ITenantRepo`), validates tenant (exists, Active, has connection string), optionally decrypts connection string, and sets `context.Items[CurrentTenant]`. Downstream code (tenant user handlers and DbContext resolution) then use this tenant to resolve the tenant's database.
- **Handlers**: The same tenant user commands/queries used by the tenant API (e.g. GetUsersListQuery, AddAppUserCommand, ModifyUserRolesCommand) are reused. They receive the tenant from the middleware-set context and use the tenant's connection string to create/resolve `AppDbContext` for that tenant. No duplicate handler logic.
- **Authorization**: Host claims (ViewTenantUsers, CreateTenantUser, UpdateTenantUser, UnActivateTenantUser, SetTenantUserPassword, ModifyTenantUserRoles, ModifyTenantUserClaims) gate the endpoints so only authorized host users can manage tenant users.

## Flow

1. Host admin selects a tenant (e.g. from tenant detail page) and opens "Users" for that tenant. Frontend calls `GET /API/V01/Host/Tenants/{tenantId}/Users/List` (or equivalent) with host JWT.
2. Request hits HostTenantUsersContextMiddleware; tenantId is parsed from the path; tenant is loaded and set in HttpContext.
3. MediatR sends the same list query used by tenant API; the handler resolves AppDbContext for the current tenant (from context) and runs the query against that tenant's DB.
4. Response returns that tenant's users. Create, update, deactivate, set password, roles, and claims follow the same pattern: route provides tenantId, middleware sets tenant context, existing tenant user handlers execute against that tenant's DB.

## Why route-based tenant context

- **Security**: Tenant is determined by the path, not by client-supplied headers or body, so clients cannot switch tenant by tampering with payload.
- **Consistency**: One middleware sets context for all Host/Tenants/{tenantId}/Users/* endpoints; handlers stay agnostic of how tenant was resolved.
- **Reuse**: Tenant user CQRS handlers are shared between tenant API and host tenant-users API; only the source of "current tenant" differs (tenant resolution middleware vs host tenant-users middleware).

## Related files

- `Alforqan.Api/Controllers/Host/TenantUsers/HostTenantUsersController.cs`
- `Alforqan.Api/Middlewares/HostTenantUsersContextMiddleware.cs`
- Tenant user handlers in `Alforqan.Core/Features/Users/Tenant/` and Identity tenant authorization handlers
- Feature doc: [host-tenant-users.md](../features/host-tenant-users.md)
