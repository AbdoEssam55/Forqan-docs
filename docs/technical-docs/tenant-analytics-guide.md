# Tenant Analytics – Technical Guide

This guide explains the data model, APIs, and how to record or read tenant metrics and health.

## Data Model

- **TenantMetric** (Host DB): `TenantId`, `MetricKey` (string, max 100), `PeriodType` ("Day" or "Month"), `PeriodStart` (DateTime), `Value` (decimal), `Unit` (string?, optional). Unique on `(TenantId, MetricKey, PeriodType, PeriodStart)`.
- **Tenant.LastActivityAt** (DateTime?, optional): Updated when a tenant request is processed (e.g. by middleware). Used for health summary.

## Recording Metrics

### ITenantMetricsWriter

1. **Inject** `ITenantMetricsWriter` in middleware or service.
2. **Record a metric**:  
   `await _tenantMetricsWriter.RecordAsync(tenantId, "ApiRequests", "Day", periodStart, count, "count", cancellationToken);`  
   Upserts the row for that tenant/metric/period (same periodStart overwrites value).
3. **Record last activity**:  
   `await _tenantMetricsWriter.RecordLastActivityAsync(tenantId, cancellationToken);`  
   Sets `Tenant.LastActivityAt = DateTime.UtcNow`.

### Middleware

- **TenantResolutionMiddleware** optionally injects `ITenantMetricsWriter` and calls `RecordLastActivityAsync(tenant.Id)` after resolving the tenant (before calling next). So every tenant request updates LastActivityAt.

### Period boundaries

- **Day**: Use `periodStart = date.Date` (midnight UTC or local).
- **Month**: Use `periodStart = new DateTime(year, month, 1)`.

## Host Admin Read API

### Get tenant metrics

- **GET** **/API/V01/Host/Tenants/{tenantId}/metrics**
- Query: `metricKeys` (list, optional), `periodType` (optional), `fromDate` (optional), `toDate` (optional).
- Returns list of `{ metricKey, periodType, periodStart, value, unit }`.
- Requires host auth and **ViewTenantsClaim**.

### Get tenants health summary

- **GET** **/API/V01/Host/Tenants/analytics/health-summary**
- Returns list of `{ tenantId, code, name, lastActivityAt }`.
- Requires host auth and **ViewTenantsClaim**.

## Adding a New Metric Key

1. **Choose a key**: e.g. `"ApiRequests"`, `"ActiveUsers"`, `"StorageMb"`, `"BillingAmount"`. No platform registration required.
2. **Record**: Call `ITenantMetricsWriter.RecordAsync(tenantId, "YourKey", periodType, periodStart, value, unit)` from middleware, product code, or a background job.
3. **Read**: Host admin uses GET metrics with optional `metricKeys` filter.

## Interface Summary

- **ITenantMetricsWriter.RecordAsync(tenantId, metricKey, periodType, periodStart, value, unit?, cancellationToken)**  
  Upserts the metric row for that tenant/metric/period.
- **ITenantMetricsWriter.RecordLastActivityAsync(tenantId, cancellationToken)**  
  Sets Tenant.LastActivityAt to now.

## References

- **Explanation**: [tenant-analytics-explained.md](../features-explanation-docs/tenant-analytics-explained.md)
- **Entity**: `Alforqan.Data/Entities/Tenants/TenantMetric.cs`; Tenant entity has `LastActivityAt`
- **Repo**: `ITenantRepo.RecordOrUpdateMetricAsync`, `UpdateLastActivityAsync`, `GetMetricsAsync`, `GetTenantsHealthSummaryAsync`
