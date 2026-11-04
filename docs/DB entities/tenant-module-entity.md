# TenantModule Entity - Detailed Explanation

## Overview

The **TenantModule** entity represents the **feature/module enablement** for a tenant. It establishes a many-to-one relationship with `Tenant`, allowing each tenant to have multiple modules (features) that can be individually enabled or disabled. This enables **feature gating** and **subscription-based access control** in the multi-tenant Alforqan system.

### Purpose

- **Feature gating**: Control which features/modules are available to each tenant
- **Subscription management**: Enable/disable modules based on subscription plans
- **Module tracking**: Track when modules were enabled or disabled
- **Flexible configuration**: Allow tenants to have different feature sets

### Database Context

- **Location**: Host Database (`HostDbContext`)
- **Table Name**: `TenantModules`
- **Database Type**: SQL Server
- **Relationships**: Many-to-One with `Tenant`

---

## Entity Structure

### Class Declaration

```csharp
[Index(nameof(TenantId), nameof(ModuleKey), IsUnique = true)]
public class TenantModule : AuditedEntity
```

### Inheritance

**Inherits from `AuditedEntity`**, which provides:

- **`Id`** (Guid) - Primary key (named `TenantModuleId` in database)
- **`CreatedAt`** (DateTime) - Timestamp when record was created
- **`CreatedBy`** (Guid?) - User ID who created the record (foreign key to `AppUser`)
- **`ModifiedAt`** (DateTime?) - Timestamp when record was last modified
- **`ModifiedBy`** (Guid?) - User ID who last modified the record (foreign key to `AppUser`)

**Note**: Unlike `Tenant` which inherits from `FullAuditedEntity` (includes soft delete), `TenantModule` inherits from `AuditedEntity` (no soft delete). This means:
- ❌ No `IsDeleted` property
- ❌ No `DeletedAt` property
- ✅ Hard delete only (if a module is removed, the record is deleted)

### Indexes

**Composite Unique Index on `(TenantId, ModuleKey)`**:
- Ensures each tenant can have **only one record per module**
- Prevents duplicate module entries for the same tenant
- Enables fast lookups: "Does tenant X have module Y enabled?"
- Database constraint: `UNIQUE INDEX` on `(TenantId, ModuleKey)` columns

**Example**: A tenant cannot have two `TenantModule` records with `ModuleKey = Reports`. The unique constraint prevents this.

---

## Property-by-Property Explanation

### 1. TenantId

```csharp
[Required]
public Guid TenantId { get; set; }
```

**Purpose**: Foreign key reference to the `Tenant` entity. Identifies which tenant this module configuration belongs to.

**Database**: `uniqueidentifier NOT NULL`, foreign key to `Tenants.Id`.

**Relationship**:
- Many `TenantModule` records can belong to one `Tenant`
- Managed via foreign key constraint: `FK_TenantModules_Tenants_TenantId`

**Usage Examples**:

```csharp
// Find all modules for a tenant
var modules = await _context.TenantModules
    .Where(m => m.TenantId == tenantId)
    .ToListAsync();

// Check if tenant has specific module
var hasReports = await _context.TenantModules
    .AnyAsync(m => m.TenantId == tenantId && 
                   m.ModuleKey == TenantModuleKey.Reports && 
                   m.IsEnabled);
```

**Validation**:
- Required (cannot be null)
- Must reference an existing `Tenant.Id`

---

### 2. ModuleKey

```csharp
[Required, MaxLength(100)]
public TenantModuleKey ModuleKey { get; set; }
```

**Purpose**: Enum identifying which feature/module this record represents. Determines what functionality is being enabled or disabled.

**Type**: `TenantModuleKey` enum with values:
- `Core` - Core functionality (always enabled for all tenants)
- `Reports` - Reporting and analytics features
- `PaymentsBasic` - Basic payment processing (Phase 1)

**Database**: Stored as `nvarchar(100)` (string) via Fluent API configuration, **NOT NULL**.

**⚠️ Note on `MaxLength` Attribute**: The `[MaxLength(100)]` attribute on an enum property **does not meaningfully apply** when the enum is stored as a string. EF Core uses the Fluent API configuration (`.HasConversion<string>()`) which determines the storage type. Since enum names are converted to strings (e.g., "Core", "Reports"), they are typically short. The MaxLength could theoretically limit the string length, but enum names are usually under 50 characters. This attribute can be removed if enum storage type is clearly documented, or kept for database schema documentation purposes.

**Usage Examples**:

```csharp
// Check if Reports module is enabled
if (tenant.Modules.Any(m => m.ModuleKey == TenantModuleKey.Reports && m.IsEnabled))
{
    // Show reports menu item
    // Allow access to reports endpoints
}

// Enable PaymentsBasic module
tenant.Modules.Add(new TenantModule
{
    TenantId = tenant.Id,
    ModuleKey = TenantModuleKey.PaymentsBasic,
    IsEnabled = true,
    EnabledOn = DateTime.UtcNow
});
```

**Validation**:
- Required (cannot be null)
- Must be a valid `TenantModuleKey` enum value
- Combined with `TenantId`, must be unique (enforced by composite unique index)

**Future Module Keys** (Phase 2+):
- `PaymentsAdvanced` - Advanced payment features
- `Payroll` - Teacher payroll management
- `PointsSystem` - Student points/rewards
- `Analytics` - Advanced analytics
- `CustomBranding` - Custom tenant branding

---

### 3. IsEnabled

```csharp
[Required]
public bool IsEnabled { get; set; }
```

**Purpose**: Boolean flag indicating whether this module is currently **enabled** (true) or **disabled** (false) for the tenant.

**Database**: `bit NOT NULL`, defaults to `false` (if not specified).

**Usage Examples**:

```csharp
// Feature gating in middleware or service
public bool IsModuleEnabled(Tenant tenant, TenantModuleKey moduleKey)
{
    return tenant.Modules
        .Any(m => m.ModuleKey == moduleKey && m.IsEnabled);
}

// Enable a module
var module = tenant.Modules.FirstOrDefault(m => m.ModuleKey == TenantModuleKey.Reports);
if (module != null)
{
    module.IsEnabled = true;
    module.EnabledOn = DateTime.UtcNow;
    module.DisabledOn = null;
}

// Disable a module
module.IsEnabled = false;
module.DisabledOn = DateTime.UtcNow;
```

**Business Rules**:
- When `IsEnabled = true`, the tenant can access the module's features
- When `IsEnabled = false`, the tenant cannot access the module (should return 403 Forbidden or hide UI)
- Changing `IsEnabled` should update `EnabledOn` or `DisabledOn` timestamps accordingly

**Validation**:
- Required (cannot be null)
- Must be `true` or `false`

---

### 4. EnabledOn

```csharp
public DateTime? EnabledOn { get; set; }
```

**Purpose**: Timestamp recording **when** this module was first enabled (or re-enabled) for the tenant. Used for auditing and tracking module activation history.

**Database**: `datetime2 NULL`, **optional**.

**Usage Examples**:

```csharp
// Track when module was enabled
if (module.IsEnabled && module.EnabledOn == null)
{
    module.EnabledOn = DateTime.UtcNow;
}

// Show activation date in UI
if (module.EnabledOn.HasValue)
{
    Console.WriteLine($"Reports module enabled on: {module.EnabledOn.Value:yyyy-MM-dd}");
}

// Calculate days since activation
var daysEnabled = (DateTime.UtcNow - module.EnabledOn.Value).TotalDays;
```

**Business Rules**:
- Should be set to `DateTime.UtcNow` when `IsEnabled` changes from `false` to `true`
- Should be cleared (set to `null`) when module is disabled (optional - some prefer to keep history)
- Used for auditing: "When did tenant X get access to Reports?"

**Validation**:
- Optional (nullable)
- Should be in UTC format (use `DateTime.UtcNow`)

---

### 5. DisabledOn

```csharp
public DateTime? DisabledOn { get; set; }
```

**Purpose**: Timestamp recording **when** this module was disabled for the tenant. Used for auditing and tracking module deactivation history.

**Database**: `datetime2 NULL`, **optional**.

**Usage Examples**:

```csharp
// Track when module was disabled
if (!module.IsEnabled && module.DisabledOn == null)
{
    module.DisabledOn = DateTime.UtcNow;
}

// Show deactivation date in UI
if (module.DisabledOn.HasValue)
{
    Console.WriteLine($"Reports module disabled on: {module.DisabledOn.Value:yyyy-MM-dd}");
}

// Check if module was recently disabled
if (module.DisabledOn.HasValue && 
    (DateTime.UtcNow - module.DisabledOn.Value).TotalDays < 30)
{
    // Show "Reactivate Reports Module" option
}
```

**Business Rules**:
- Should be set to `DateTime.UtcNow` when `IsEnabled` changes from `true` to `false`
- Should be cleared (set to `null`) when module is re-enabled
- Used for auditing: "When did tenant X lose access to Reports?"

**Validation**:
- Optional (nullable)
- Should be in UTC format (use `DateTime.UtcNow`)

**Note**: Both `EnabledOn` and `DisabledOn` can have values simultaneously if you want to keep full history. Alternatively, you can clear one when setting the other.

---

### 6. Tenant (Navigation Property)

```csharp
[ForeignKey(nameof(TenantId))]
public Tenant? Tenant { get; set; }
```

**Purpose**: Navigation property providing access to the related `Tenant` entity. Enables eager loading and navigation from `TenantModule` to `Tenant`.

**Database**: Not a column - represents the foreign key relationship.

**Relationship**:
- Many `TenantModule` records belong to one `Tenant`
- Managed via `TenantId` foreign key
- Inverse navigation: `Tenant.Modules` (ICollection<TenantModule>)

**Usage Examples**:

```csharp
// Eager load tenant with modules
var modules = await _context.TenantModules
    .Include(m => m.Tenant)
    .Where(m => m.IsEnabled)
    .ToListAsync();

// Access tenant info from module
var tenantName = module.Tenant?.Name;
var tenantDomain = module.Tenant?.PrimaryDomain;

// Navigation from module to tenant
if (module.Tenant != null && module.Tenant.Status == TenantStatus.Active)
{
    // Module belongs to an active tenant
}
```

**Lazy Loading**: If lazy loading is enabled, accessing `module.Tenant` will trigger a database query. Prefer `.Include(m => m.Tenant)` for explicit loading.

---

## Database Mapping

### Table Schema

```sql
CREATE TABLE TenantModules (
    TenantModuleId UNIQUEIDENTIFIER PRIMARY KEY,
    TenantId UNIQUEIDENTIFIER NOT NULL,
    ModuleKey NVARCHAR(100) NOT NULL,
    IsEnabled BIT NOT NULL,
    EnabledOn DATETIME2 NULL,
    DisabledOn DATETIME2 NULL,
    CreatedAt DATETIME2 NOT NULL,
    CreatedBy UNIQUEIDENTIFIER NULL,
    ModifiedAt DATETIME2 NULL,
    ModifiedBy UNIQUEIDENTIFIER NULL,
    
    CONSTRAINT FK_TenantModules_Tenants_TenantId 
        FOREIGN KEY (TenantId) REFERENCES Tenants(Id),
    CONSTRAINT UQ_TenantModules_TenantId_ModuleKey 
        UNIQUE (TenantId, ModuleKey)
);
```

**Note**: The primary key column is named `TenantModuleId` in the database (not `Id`), though the entity property is `Id` (from base class).

### EF Core Configuration

Located in `Alforqan.Infrastructure/Data/HostDbContext.cs`:

```csharp
modelBuilder.Entity<TenantModule>(entity =>
{
    // Enum to string conversion (cannot be done with data annotations)
    entity.Property(m => m.ModuleKey)
          .HasConversion<string>();
});
```

**Note**: 
- The composite unique index is defined via data annotation on the entity class (`[Index]` attribute)
- The foreign key relationship is defined via data annotation (`[ForeignKey]` attribute)
- Enum conversion to string requires Fluent API (cannot be done with data annotations)

---

## Common Use Cases

### 1. Check if Module is Enabled

```csharp
public async Task<bool> IsModuleEnabledAsync(Guid tenantId, TenantModuleKey moduleKey)
{
    return await _hostDbContext.TenantModules
        .AnyAsync(m => m.TenantId == tenantId && 
                      m.ModuleKey == moduleKey && 
                      m.IsEnabled);
}
```

### 2. Enable Module for Tenant

```csharp
public async Task EnableModuleAsync(Guid tenantId, TenantModuleKey moduleKey)
{
    var existing = await _hostDbContext.TenantModules
        .FirstOrDefaultAsync(m => m.TenantId == tenantId && 
                                 m.ModuleKey == moduleKey);
    
    if (existing != null)
    {
        existing.IsEnabled = true;
        existing.EnabledOn = DateTime.UtcNow;
        existing.DisabledOn = null;
    }
    else
    {
        _hostDbContext.TenantModules.Add(new TenantModule
        {
            TenantId = tenantId,
            ModuleKey = moduleKey,
            IsEnabled = true,
            EnabledOn = DateTime.UtcNow
        });
    }
    
    await _hostDbContext.SaveChangesAsync();
}
```

### 3. Disable Module for Tenant

```csharp
public async Task DisableModuleAsync(Guid tenantId, TenantModuleKey moduleKey)
{
    var module = await _hostDbContext.TenantModules
        .FirstOrDefaultAsync(m => m.TenantId == tenantId && 
                                 m.ModuleKey == moduleKey);
    
    if (module != null)
    {
        module.IsEnabled = false;
        module.DisabledOn = DateTime.UtcNow;
        await _hostDbContext.SaveChangesAsync();
    }
}
```

### 4. Get All Enabled Modules for Tenant

```csharp
public async Task<List<TenantModuleKey>> GetEnabledModulesAsync(Guid tenantId)
{
    return await _hostDbContext.TenantModules
        .Where(m => m.TenantId == tenantId && m.IsEnabled)
        .Select(m => m.ModuleKey)
        .ToListAsync();
}
```

### 5. Feature Gating in Middleware or Authorization

```csharp
// In middleware or authorization handler
public async Task<bool> CheckModuleAccessAsync(
    Guid tenantId, 
    TenantModuleKey requiredModule)
{
    var hasAccess = await _hostDbContext.TenantModules
        .AnyAsync(m => m.TenantId == tenantId && 
                      m.ModuleKey == requiredModule && 
                      m.IsEnabled);
    
    if (!hasAccess)
    {
        throw new ForbiddenException(
            $"Module {requiredModule} is not enabled for this tenant");
    }
    
    return true;
}
```

### 6. Seed Default Modules (from HostDbSeeding.cs)

```csharp
var defaultModules = new[]
{
    new { Key = TenantModuleKey.Core, IsEnabled = true },
    new { Key = TenantModuleKey.Reports, IsEnabled = true },
    new { Key = TenantModuleKey.PaymentsBasic, IsEnabled = true }
};

foreach (var module in defaultModules)
{
    var existing = tenant.Modules.FirstOrDefault(m => m.ModuleKey == module.Key);
    if (existing != null)
    {
        existing.IsEnabled = module.IsEnabled;
        if (module.IsEnabled && existing.EnabledOn == null)
        {
            existing.EnabledOn = DateTime.UtcNow;
        }
    }
    else
    {
        tenant.Modules.Add(new TenantModule
        {
            ModuleKey = module.Key,
            IsEnabled = module.IsEnabled,
            EnabledOn = module.IsEnabled ? DateTime.UtcNow : null
        });
    }
}
```

---

## Relationships

### TenantModule → Tenant (Many-to-One)

```
TenantModule (Many) ──→ (1) Tenant
```

- **Navigation Property**: `TenantModule.Tenant` (Tenant?)
- **Foreign Key**: `TenantModule.TenantId` → `Tenant.Id`
- **Inverse Navigation**: `Tenant.Modules` (ICollection<TenantModule>)
- **Purpose**: Links module configuration to the tenant it belongs to
- **Cascade Delete**: Typically, if a tenant is deleted, its modules should be deleted (hard delete, since no soft delete on TenantModule)

**Example**:

```csharp
// From TenantModule to Tenant
var tenantName = module.Tenant?.Name;

// From Tenant to TenantModules
var enabledModules = tenant.Modules.Where(m => m.IsEnabled).ToList();
```

---

## Business Rules & Validation

### 1. Unique Constraint

- **Rule**: Each tenant can have **only one** `TenantModule` record per `ModuleKey`
- **Enforcement**: Composite unique index on `(TenantId, ModuleKey)`
- **Violation**: Attempting to insert a duplicate will throw a database constraint violation

### 2. Module Enablement Logic

- **When enabling**: Set `IsEnabled = true`, set `EnabledOn = DateTime.UtcNow`, clear `DisabledOn = null`
- **When disabling**: Set `IsEnabled = false`, set `DisabledOn = DateTime.UtcNow`, keep `EnabledOn` for history (optional)

### 3. Core Module

- **Rule**: `TenantModuleKey.Core` should **always** be enabled for all tenants
- **Enforcement**: Enforced in seeding and business logic (not database constraint)
- **Rationale**: Core functionality is required for the system to function

### 4. Plan-Based Module Access

- **Rule**: Module availability may be restricted by subscription plan
- **Example**: `PaymentsBasic` might be available on `Starter` plan, but `PaymentsAdvanced` requires `Enterprise` plan
- **Enforcement**: Business logic in service layer, not database constraints

---

## Security Considerations

### 1. Module Access Control

- **Always verify** module is enabled before allowing access to features
- **Check in middleware/authorization**: Don't rely on UI hiding - enforce at API level
- **Return 403 Forbidden** (not 404) when module is disabled - makes it clear the feature exists but is not accessible

### 2. Tenant Isolation

- **Never allow** cross-tenant module queries without proper tenant context
- **Always filter** by `TenantId` when querying modules
- **Validate** that the authenticated user belongs to the tenant before enabling/disabling modules

### 3. Audit Trail

- **Track changes**: `CreatedAt`, `CreatedBy`, `ModifiedAt`, `ModifiedBy` provide audit trail
- **Track enablement**: `EnabledOn` and `DisabledOn` provide module activation history
- **Use for compliance**: Know when and by whom modules were enabled/disabled

### 4. Module Key Validation

- **Validate enum values**: Ensure only valid `TenantModuleKey` values are used
- **Prevent injection**: Don't allow arbitrary string values for `ModuleKey`
- **Type safety**: Using enum prevents invalid module keys

---

## Related Files

- **Entity**: `Alforqan.Data/Entities/Tenants/TenantModule.cs`
- **DbContext**: `Alforqan.Infrastructure/Data/HostDbContext.cs`
- **Enum**: `Alforqan.Data/Enums/Tenants/TenantModuleKey.cs`
- **Seeding**: `Alforqan.Infrastructure/Seeding/HostDbSeeding.cs`
- **Migrations**: `Alforqan.Infrastructure/Migrations/Host/`
- **Base Class**: `Alforqan.Data/Base/Implementations/AuditedEntity.cs`
- **Related Entity**: `Alforqan.Data/Entities/Tenants/Tenant.cs`

---

## Future Enhancements

### Phase 1
- Implement module-based feature gating in middleware
- Add module enablement/disablement API endpoints
- Add module status checks in authorization handlers

### Phase 2+
- **Module Dependencies**: Some modules may require other modules (e.g., `PaymentsAdvanced` requires `PaymentsBasic`)
- **Module Configuration**: Store module-specific settings (e.g., payment gateway credentials for PaymentsBasic)
- **Module Usage Tracking**: Track how often modules are used for analytics
- **Module Trial Periods**: Allow temporary module access for trials
- **Module Expiration**: Automatically disable modules after subscription expires
- **Module Permissions**: Fine-grained permissions within modules (e.g., who can access Reports)

---

## Summary

The `TenantModule` entity enables **flexible feature gating** in the multi-tenant Alforqan system:

- **Represents**: Which features/modules are enabled for each tenant
- **Stores**: Module key, enablement status, activation/deactivation timestamps
- **Enables**: Subscription-based feature access, module management, audit trails
- **Critical for**: Feature gating, access control, subscription management

**Key Takeaways**:
1. Each tenant can have **one record per module** (enforced by unique constraint)
2. `IsEnabled` controls **access to module features**
3. `EnabledOn`/`DisabledOn` provide **audit trail** for module activation
4. Module checks should be **enforced at API level**, not just UI
5. `Core` module should **always be enabled** for all tenants
6. Inherits from `AuditedEntity` (no soft delete - hard delete only)

---

## Quick Reference

### Finding TenantModule
- **By Tenant and Module**: `TenantId` + `ModuleKey` (most common)
- **By Tenant**: All modules for a tenant
- **By Module**: All tenants with a specific module enabled

### Important Constraints
- `TenantId` + `ModuleKey`: **Unique** (composite unique index)
- `ModuleKey`: Required, must be valid enum value
- `IsEnabled`: Required, boolean

### Default Behavior
- New modules: `IsEnabled = false` (unless explicitly set)
- `EnabledOn`/`DisabledOn`: `null` initially
- Timestamps should use `DateTime.UtcNow`

### Common Patterns
```csharp
// Check if enabled
var isEnabled = tenant.Modules.Any(m => m.ModuleKey == key && m.IsEnabled);

// Enable module
module.IsEnabled = true;
module.EnabledOn = DateTime.UtcNow;

// Disable module
module.IsEnabled = false;
module.DisabledOn = DateTime.UtcNow;
```

