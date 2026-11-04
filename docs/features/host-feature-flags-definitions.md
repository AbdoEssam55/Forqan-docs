# Host Feature Flags (Definitions)

## Overview

Feature flag **definitions** are created and managed at the host level. Each definition has a key (e.g. "NewCheckout", "BetaDashboard"); tenants are then assigned on/off (and optional rollout percentage) per flag via the Host Tenants API. This API provides CRUD for definitions and bulk assign/update of tenants per flag. All endpoints require host authentication (`HostScheme`) and appropriate host claims.

## Base route

`/API/V01/Host/FeatureFlags`

## Main operations

- **Definitions**: `GET List`, `GET {id}`, `POST Create`, `PUT {id}`, `PATCH {id}/Archive`
- **Tenants per flag**: `GET {id}/Tenants`, `PUT {id}/Tenants` (bulk update)

Per-tenant flag values are managed under Host Tenants: `PUT /API/V01/Host/Tenants/{tenantId}/feature-flags/{flagKey}`.

## Related documentation

- [Feature flags explained](../features-explanation-docs/feature-flags-explained.md) – per-tenant flags, rollout %, TenantConfig
- [Feature flags guide](../technical-docs/feature-flags-guide.md) – entity, API, adding new flag keys

## Controller

`Alforqan.Api/Controllers/Host/FeatureFlags/HostFeatureFlagsController.cs`
