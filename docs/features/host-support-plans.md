# Host Support Plans

## Overview

Support Plans define support tiers (e.g. Basic, Premium) that can be offered alongside subscription plans. The Host Support Plans API provides CRUD and activate/deactivate for support plan records. All endpoints require host authentication (`HostScheme`) and appropriate host claims.

## Base route

`/API/V01/Host/SupportPlans`

## Main operations

- **List / get**: `GET List`, `GET {id}`
- **Create**: `POST create`
- **Update**: `PUT {id}`
- **Delete**: `DELETE {id}`
- **Deactivate**: `PATCH {id}/UnActivate`

Support plans can include rules (e.g. response time, channels). See DTOs and entity in Platform.Data (SupportPlan, SupportPlanRule).

## Related documentation

- [Host support-plans feature (explanation)](../features-explanation-docs/host-support-plans-feature.md) – purpose, entity, and how it ties to tenants/plans if applicable

## Controller

`Alforqan.Api/Controllers/Host/SupportPlans/HostSupportPlansController.cs`
