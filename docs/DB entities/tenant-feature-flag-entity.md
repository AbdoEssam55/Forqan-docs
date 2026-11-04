# TenantFeatureFlag Entity

## Overview

The **TenantFeatureFlag** entity stores per-tenant feature flags (on/off and optional rollout percentage). It is **generic**: any product defines its own flag keys (e.g. `"NewCheckout"`, `"BetaDashboard"`); the platform does not hard-code flag names.

### Purpose

- **Per-tenant toggles**: Enable or disable a feature for a specific tenant.
- **Rollout %**: Optional 0–100; when set, "enabled" is resolved by a deterministic hash so the same tenant always gets the same result (consistent rollout).
- **In addition to modules**: Modules remain the primary capability switch; feature flags are for gradual rollouts or product-specific toggles.

### Database Context

- **Location**: Host Database (`HostDbContext`)
- **Table Name**: `TenantFeatureFlags`
- **Relationships**: Many-to-one with `Tenant`

---

## Entity Structure

### Class Declaration

```csharp
[Index(nameof(TenantId), nameof(FlagKey), IsUnique = true)]
public class TenantFeatureFlag : AuditedEntity
```

### Inheritance

**Inherits from `AuditedEntity`**, which provides:

- **`Id`** (Guid) - Primary key
- **`CreatedAt`**, **`CreatedBy`**, **`ModifiedAt`**, **`ModifiedBy`**

### Indexes

- **Unique index on `(TenantId, FlagKey)`**: One row per tenant per flag key.

---

## Properties

| Property            | Type   | Description                                                                 |
|---------------------|--------|-----------------------------------------------------------------------------|
| **TenantId**        | Guid   | FK to Tenant. Required.                                                    |
| **FlagKey**         | string | Flag identifier (e.g. "NewCheckout", "BetaReports"). Max 100. Required.   |
| **IsEnabled**       | bool   | Base on/off. When RolloutPercentage is null, this is the resolved value.   |
| **RolloutPercentage** | int? | Optional 0–100. When set, resolved enabled = (stable bucket < this value). |

---

## References

- **Explanation**: [feature-flags-explained.md](../features-explanation-docs/feature-flags-explained.md)
- **Technical guide**: [feature-flags-guide.md](../technical-docs/feature-flags-guide.md)
- **Entity**: `Alforqan.Data/Entities/Tenants/TenantFeatureFlag.cs`
- **Tenant**: [tenant-entity.md](tenant-entity.md) – Tenant has collection `FeatureFlags`.
