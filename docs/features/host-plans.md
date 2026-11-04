# Host Plans

## Overview

Plans define subscription tiers (e.g. Starter, Standard, Enterprise). Tenants are assigned a plan; plan limits (e.g. max students, max users) are enforced when creating tenant-scoped entities. The Host Plans API provides CRUD and activate/deactivate for plan records. All endpoints require host authentication (`HostScheme`) and appropriate host claims.

## Base route

`/API/V01/Host/Plans`

## Main operations

- **List / get**: `GET List`, `GET {id}`
- **Create**: `POST create`
- **Update**: `PUT {id}`
- **Delete**: `DELETE {id}`
- **Deactivate**: `PATCH {id}/UnActivate`

## Related documentation

- [Plan-based limits explained](../features-explanation-docs/plan-based-limits-explained.md) – how limits are enforced per tenant/plan
- [Plan limits guide](../technical-docs/plan-limits-guide.md) – how to add limit keys and wire checks in handlers

## Controller

`Alforqan.Api/Controllers/Host/Plans/HostPlansController.cs`
