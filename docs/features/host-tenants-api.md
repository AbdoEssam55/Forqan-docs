# Host Tenants API

## Overview

The Host Tenants API allows platform administrators to manage tenants: resolve by domain or code, get statistics, enable/disable modules and plugins, manage per-tenant branding and feature flags, view metrics and health, and export tenant data. All endpoints require host authentication (`HostScheme`) and appropriate host claims.

## Base route

`/API/V01/Host/Tenants`

## Main operations

- **Resolve tenant**: `GET by-domain/{domain}`, `GET by-code/{code}`
- **Stats**: `GET stats` (aggregate tenant counts)
- **Modules**: `GET available-modules`; `POST {tenantId}/modules/{moduleKey}/enable`, `.../disable`; `POST {tenantId}/modules/enable-multiple`, `disable-multiple`
- **Plugins**: `GET available-plugin-keys`
- **Branding**: `GET {tenantId}/branding`, `PUT {tenantId}/branding`, `POST {tenantId}/branding/logo`
- **Feature flags**: `GET {tenantId}/feature-flags`, `PUT {tenantId}/feature-flags/{flagKey}`, `DELETE {tenantId}/feature-flags/{flagKey}`
- **Analytics**: `GET {tenantId}/metrics`, `GET analytics/health-summary`
- **Export**: `POST {tenantId}/Export`

## Related documentation

- [Tenant resolution feature](../features-explanation-docs/tenant-resolution-feature.md)
- [Tenant provisioning](../features-explanation-docs/tenant-provisioning-explained.md)
- [Tenant branding](../technical-docs/tenant-branding-guide.md) and [tenant-branding-explained](../features-explanation-docs/tenant-branding-explained.md)
- [Feature flags](../features-explanation-docs/feature-flags-explained.md), [feature-flags-guide](../technical-docs/feature-flags-guide.md)
- [Module gating](../features-explanation-docs/module-gating-on-tenant-endpoints-explained.md), [module-gating-guide](../technical-docs/module-gating-guide.md)
- [Tenant analytics](../features-explanation-docs/tenant-analytics-explained.md), [tenant-analytics-guide](../technical-docs/tenant-analytics-guide.md)
- [Tenant data export](../features-explanation-docs/tenant-data-export-feature.md)

## Controller

`Alforqan.Api/Controllers/Tenants/TenantsController.cs` (host routes under `HostEndpointPrefix`).
