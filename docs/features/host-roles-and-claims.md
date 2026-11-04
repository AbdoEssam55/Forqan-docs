# Host Roles & Claims

## Overview

Host Roles and Claims are used to authorize platform administrators. Roles (e.g. SystemAdmin) and claims (e.g. ViewTenants, UpdateTenant) are stored in the Host database. The API allows CRUD for roles and assignment of roles/claims to host users. All endpoints require host authentication (`HostScheme`) and appropriate host claims.

## Base route

`/API/V01/Host/Roles`

## Main operations

- **Roles**: `GET List`, `GET ListWithClaims`, `GET {id}`, `POST CreateRole`, `PATCH UpdateRole`, `DELETE {id}`
- **User roles**: `GET Users/{id}/Roles`, `PUT Users/Roles`
- **User claims**: `GET Users/{id}/Claims`, `PUT Users/Claims`

Protected system roles (e.g. SystemAdmin) cannot be deleted or renamed via API.

## Related documentation

- [Shared authorization services](../features-explanation-docs/shared-authorization-services-feature.md) – how host and tenant share authorization handlers
- [Protected system roles](../features-explanation-docs/protected-system-roles-feature.md) and [protected-system-roles-guide](../technical-docs/protected-system-roles-guide.md)
- [Host identity system](../features-explanation-docs/host-identity-system.md)
- [Authorization services guide](../technical-docs/authorization-services-guide.md)

## Controller

`Alforqan.Api/Controllers/Host/Roles/HostRolesController.cs`
