# TenantMetric Entity

## Overview

The **TenantMetric** entity stores per-tenant usage metrics (key, period, value). It is **generic**: any product defines its own metric keys (e.g. `"ApiRequests"`, `"StorageMb"`, `"BillingAmount"`); the platform does not hard-code metric names.

### Purpose

- **Per-tenant usage**: Store numeric metrics per period (e.g. API request count per day, storage per month).
- **Time-bound**: Each row is for a specific period (PeriodType + PeriodStart) so host admin can query usage over time.
- **Billing**: Optional billing data can be stored as metrics (e.g. BillingAmount per month).

### Database Context

- **Location**: Host Database (`HostDbContext`)
- **Table Name**: `TenantMetrics`
- **Relationships**: Many-to-one with `Tenant`

---

## Entity Structure

### Class Declaration

```csharp
[Index(nameof(TenantId), nameof(MetricKey), nameof(PeriodType), nameof(PeriodStart), IsUnique = true)]
public class TenantMetric : AuditedEntity
```

### Inheritance

**Inherits from `AuditedEntity`**, which provides:

- **`Id`** (Guid) - Primary key
- **`CreatedAt`**, **`CreatedBy`**, **`ModifiedAt`**, **`ModifiedBy`**

### Indexes

- **Unique index on `(TenantId, MetricKey, PeriodType, PeriodStart)`**: One row per tenant per metric per period (enables upsert).

---

## Properties

| Property      | Type     | Description                                                          |
|---------------|----------|----------------------------------------------------------------------|
| **TenantId**  | Guid     | FK to Tenant. Required.                                             |
| **MetricKey** | string   | Metric identifier (e.g. "ApiRequests", "StorageMb"). Max 100. Required. |
| **PeriodType**| string   | "Day" or "Month". Required.                                          |
| **PeriodStart** | DateTime | Start of the period (e.g. midnight for day, first of month). Required. |
| **Value**     | decimal  | Numeric value.                                                       |
| **Unit**      | string?  | Optional unit (e.g. "count", "Mb"). Max 20.                          |

---

## References

- **Explanation**: [tenant-analytics-explained.md](../features-explanation-docs/tenant-analytics-explained.md)
- **Technical guide**: [tenant-analytics-guide.md](../technical-docs/tenant-analytics-guide.md)
- **Entity**: `Alforqan.Data/Entities/Tenants/TenantMetric.cs`
- **Tenant**: [tenant-entity.md](tenant-entity.md) – Tenant has `LastActivityAt` for health; metrics are linked by TenantId.
