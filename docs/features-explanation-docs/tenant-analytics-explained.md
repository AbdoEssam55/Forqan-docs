# Tenant Analytics / Monitoring Feature

## Overview

Per-tenant usage metrics, health, and optional billing data. The design is **generic**: the platform stores metric **keys** and values (e.g. `"ApiRequests"`, `"ActiveUsers"`, `"StorageMb"`, `"BillingAmount"`); products (or middleware) report and consume them. The platform does not define *how* metrics are computed, only storage and read API.

## Why Tenant Analytics?

- **Usage visibility**: Host admin can see per-tenant usage (e.g. API calls, active users) over time.
- **Health**: Last activity per tenant (e.g. "no activity in 7 days") for support and monitoring.
- **Billing**: Optional billing data can be stored as metrics (e.g. `BillingAmount` per period) and consumed by external billing systems.

## Generic Design

- **Metric key**: A string (e.g. `"ApiRequests"`, `"ErrorCount"`, `"StorageMb"`). No platform enum; products define keys.
- **Time-bound**: Metrics are stored per period (e.g. Day or Month) so host admin can query "usage for tenant X in period Y".
- **Who writes**: Middleware (e.g. request count per tenant per day), product-specific code (e.g. "active users" from tenant DB), or external billing job. Platform provides `ITenantMetricsWriter.RecordAsync` and `RecordLastActivityAsync`.
- **Who reads**: Host admin via GET Tenants/{tenantId}/metrics and GET Tenants/analytics/health-summary.

## Flow

1. **Recording**: When a tenant request is processed, middleware (or product code) can call `ITenantMetricsWriter.RecordLastActivityAsync(tenantId)` to update `Tenant.LastActivityAt`. For numeric metrics (e.g. request count), call `RecordAsync(tenantId, "ApiRequests", "Day", periodStart, value, "count")`.
2. **Host admin**: GET Tenants/{tenantId}/metrics?periodType=Day&from=...&to=... returns list of metrics. GET Tenants/analytics/health-summary returns all tenants with LastActivityAt for health view.

## LastActivityAt

- **Tenant.LastActivityAt**: Updated by middleware when a tenant is resolved (every tenant request). Simplifies "health" view (e.g. "no activity in 7 days").
- **Optional**: If you prefer not to update on every request, you can remove the middleware call and update only from product code or a background job.

## References

- **Technical guide**: [tenant-analytics-guide.md](../technical-docs/tenant-analytics-guide.md) – entity, API, how to record metrics, how to add new metric keys.
- **DB entity**: [tenant-metric-entity.md](../DB%20entities/tenant-metric-entity.md); [tenant-entity.md](../DB%20entities/tenant-entity.md) (LastActivityAt).
- **Roadmap**: [multi-tenant-platform-scope-and-roadmap.md](../../../docs/multi-tenant-platform-scope-and-roadmap.md) section 8.7.
