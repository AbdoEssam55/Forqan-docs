# Host Tenant Users

## Overview

Host Tenant Users API allows platform administrators to manage **a specific tenant's users** from the host app. The tenant context is set from the route (`tenantId`). Operations include list, get, create, update, deactivate, set password, and manage roles/claims for that tenant's users. All endpoints require host authentication (`HostScheme`) and appropriate host claims (e.g. ViewTenantUsers, CreateTenantUser, UpdateTenantUser).

## Base route

`/API/V01/Host/Tenants/{tenantId}/Users`

Middleware (`HostTenantUsersContextMiddleware`) resolves the tenant from the route and sets the tenant context so the same tenant user handlers (queries/commands) run with the correct tenant DbContext.

## Main operations

- **Users**: `GET Single/{search}`, `GET {id}`, `POST List`, `POST create`, `PUT EditUser`, `PATCH UnActivate`, `PATCH {userId}/SetPassword`
- **Roles**: `GET Roles/ListWithClaims`, `GET {userId}/Roles`, `PUT Roles`
- **Claims**: `GET {userId}/Claims`, `PUT Claims`

## Related documentation

- [Host tenant-users feature (explanation)](../features-explanation-docs/host-tenant-users-feature.md) – why host can manage tenant users, route/context, flow

## Controller

`Alforqan.Api/Controllers/Host/TenantUsers/HostTenantUsersController.cs`
