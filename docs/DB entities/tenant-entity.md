# Tenant Entity - Detailed Explanation

## Overview

The **Tenant** entity represents a **customer** of the Alforqan system - each row in the `Tenants` table corresponds to one Qur'an center, masjid, institute, or educational organization using the platform. This is a core entity in the **multi-tenant architecture** and serves as the foundation for tenant isolation and data separation.

### Purpose

- **Multi-tenant isolation**: Each tenant operates independently with its own database and configuration
- **Tenant identification**: Used to resolve tenant context from incoming requests (via domain/subdomain)
- **Subscription management**: Tracks subscription plans, status, and enabled modules
- **Configuration storage**: Stores tenant-specific settings like database connection strings

### Database Context

- **Location**: Host Database (`HostDbContext`)
- **Table Name**: `Tenants`
- **Database Type**: SQL Server
- **Relationships**: One-to-Many with `TenantModules`

---

## Entity Structure

### Class Declaration

```csharp
[Index(nameof(Code), IsUnique = true)]
[Index(nameof(PrimaryDomain), IsUnique = true)]
public class Tenant : FullAuditedEntity
```

### Inheritance

**Inherits from `FullAuditedEntity`**, which provides:

- **`Id`** (Guid) - Primary key
- **`CreatedAt`** (DateTime) - Timestamp when record was created
- **`CreatedBy`** (string?, max 256) - User who created the record
- **`ModifiedAt`** (DateTime?) - Timestamp when record was last modified
- **`ModifiedBy`** (string?, max 256) - User who last modified the record
- **`IsDeleted`** (bool) - Soft delete flag (defaults to false)
- **`DeletedAt`** (DateTime?) - Timestamp when record was soft deleted

These properties are **not explicitly defined** in the `Tenant` class - they come from the base class.

### Indexes

1. **Unique Index on `Code`**: 
   - Ensures no two tenants share the same code
   - Enables fast lookups by code
   - Database constraint: `UNIQUE INDEX` on `Code` column

2. **Unique Index on `PrimaryDomain`**:
   - Ensures no two tenants share the same primary domain
   - Critical for tenant resolution from HTTP requests
   - Database constraint: `UNIQUE INDEX` on `PrimaryDomain` column

---

## Property-by-Property Explanation

### 1. Name

```csharp
[Required, StringLength(100, MinimumLength = 2)]
public string Name { get; set; } = string.Empty;
```

**Purpose**: Human-friendly display name of the tenant organization.

**Database**: `nvarchar(100) NOT NULL`, 2-100 characters.

**Usage Examples**:
- Displayed in admin UI: "Dar Al-Qur'an Al-Noor"
- Used in emails: "Dear Admin of Dar Al-Qur'an Al-Noor"
- Shown in invoices and reports
- Example values: "Dar Al-Qur'an Al-Noor", "Al-Furqan Academy", "Masjid Al-Huda"

**Validation**:
- Required (cannot be null or empty)
- Must be between 2 and 100 characters
- Validation messages are localized (Arabic/English)

---

### 2. Code

```csharp
[Required, StringLength(50, MinimumLength = 2)]
public string Code { get; set; } = string.Empty;
```

**Purpose**: Short, unique identifier for the tenant. **Currently under evaluation** - see "Code Field Analysis" section below.

**Database**: `nvarchar(50) NOT NULL`, 2-50 characters, **UNIQUE**.

**Current Usage**:
- Only used in `HostDbSeeding.cs` for seeding default tenant: `"default"`
- **Not currently used** for tenant resolution (that uses `PrimaryDomain`)

**Potential Use Cases** (if kept):
- URL-friendly slugs: `https://alforqan.com/tenants/ALNOOR`
- Subdomain mapping alternative: `https://alnoor.alforqan.com` (though `PrimaryDomain` already covers this)
- Internal references in logs: `TenantCode=ALNOOR`
- API identifiers that don't expose domain names
- Shorter identifiers than GUIDs for human readability

**Example Values**: `"ALNOOR"`, `"IQRAA"`, `"MASJID123"`, `"default"`

**Validation**:
- Required (cannot be null or empty)
- Must be between 2 and 50 characters
- Must be unique across all tenants (enforced by unique index)

**⚠️ Decision Needed**: See "Code Field Analysis" section for evaluation of whether this field should be removed or repurposed.

---

### 3. PrimaryDomain

```csharp
[Required, StringLength(200)]
public string PrimaryDomain { get; set; } = string.Empty;
```

**Purpose**: The main domain or subdomain used to access this tenant. This is the **primary mechanism for tenant resolution** from incoming HTTP requests.

**Database**: `nvarchar(200) NOT NULL`, up to 200 characters, **UNIQUE**.

**How Tenant Resolution Works**:
```
1. HTTP Request arrives: Host = "alnoor.alforqan.com"
   ↓
2. TenantResolutionMiddleware extracts host header (after UseRouting, before UseAuthentication)
   ↓
3. Check if host endpoint (/API/V01/Tenants) → skip tenant resolution
   ↓
4. Query HostDbContext: SELECT * FROM Tenants WHERE PrimaryDomain = "alnoor.alforqan.com"
   ↓
5. Validate tenant status (must be Active)
   ↓
6. Store tenant in HttpContext.Items["CurrentTenant"]
   ↓
7. AppDbContext uses tenant.ConnectionString per-request
   ↓
8. All queries execute against tenant's isolated database
```

**System Admin vs Tenant Admin**:
- **SystemAdmin**: Manages tenants (Host DB), not restricted to a tenant, uses `SystemAdminRole` + `TenantManagementClaim`
- **Admin**: Manages within a tenant (tenant DB), restricted to their tenant, uses `AdminRole`

**Usage Examples**:
- Subdomain-based: `"alnoor.alforqan.com"`
- Custom domain: `"quran-center-eg.com"`
- Development: `"localhost"` (for default tenant)

**Validation**:
- Required (cannot be null or empty)
- Must be unique across all tenants (enforced by unique index)
- Up to 200 characters (sufficient for FQDN)

**Security Note**: This value is used to route requests, so it should be validated and sanitized when set to prevent tenant hijacking attacks.

---

### 4. ContactName

```csharp
[StringLength(100)]
public string? ContactName { get; set; }
```

**Purpose**: Name of the main contact person for the tenant organization.

**Database**: `nvarchar(100) NULL`, up to 100 characters, **optional**.

**Usage Examples**:
- Displayed in admin UI: "Contact person: Sheikh Ahmed Ali"
- Used in support tickets and communications
- Example values: `"Sheikh Ahmed"`, `"Dr. Fatima"`, `"Admin Office"`

**Validation**:
- Optional (nullable)
- If provided, maximum 100 characters

---

### 5. ContactEmail

```csharp
[StringLength(150)]
public string? ContactEmail { get; set; }
```

**Purpose**: Email address of the tenant's main contact person or administrative email.

**Database**: `nvarchar(150) NULL`, up to 150 characters, **optional**.

**Usage Examples**:
- Send onboarding emails
- Send invoices and billing notifications: "Your subscription will expire in 3 days"
- System alerts and notifications
- Example values: `"admin@alnoor-quran.org"`, `"director@furqanacademy.com"`

**Validation**:
- Optional (nullable)
- If provided, maximum 150 characters
- **Note**: No `[EmailAddress]` attribute currently - consider adding for format validation

---

### 6. ContactPhone

```csharp
[StringLength(30)]
public string? ContactPhone { get; set; }
```

**Purpose**: Phone or WhatsApp number for the tenant organization.

**Database**: `nvarchar(30) NULL`, up to 30 characters, **optional**.

**Usage Examples**:
- Displayed in UI: "For help, contact your center: +201012345678"
- Used for internal support and exports
- Example values: `"+201012345678"`, `"0020501234567"`, `"010-1234-5678"`

**Validation**:
- Optional (nullable)
- If provided, maximum 30 characters
- **Note**: No format validation currently - consider adding `[Phone]` attribute if standardized format is needed

---

### 7. PlanName

```csharp
[Required, MaxLength(100)]
public TenantPlanName PlanName { get; set; } = TenantPlanName.Starter;
```

**Purpose**: Enum describing which subscription plan this tenant is on. Used to determine feature availability and limits.

**Type**: `TenantPlanName` enum with values:
- `Starter` - Basic plan with limited features
- `Standard` - Mid-tier plan
- `Enterprise` - Full-featured plan

**Database**: Stored as `nvarchar(100)` (string) via Fluent API configuration, defaults to `"Starter"`.

**⚠️ Note on `MaxLength` Attribute**: The `[MaxLength(100)]` attribute on an enum property **does not apply** when the enum is stored as a string. EF Core uses the Fluent API configuration (`.HasConversion<string>()`) which determines the storage type. However, since enums are converted to strings (e.g., "Starter"), the MaxLength could theoretically limit the string length, but enum names are typically short. This attribute can be removed if enum storage type is clearly documented.

**Usage Examples**:

```csharp
// Check plan limits
if (tenant.PlanName == TenantPlanName.Starter && halaqatCount >= 3)
{
    // Prevent creating more halaqat or show upgrade message
    return BadRequest("Upgrade to Standard plan to create more halaqat");
}

// Feature gating
if (tenant.PlanName >= TenantPlanName.Standard)
{
    // Allow advanced reporting features
}
```

**Default Value**: `TenantPlanName.Starter` - New tenants start on the basic plan.

**Validation**:
- Required (cannot be null)
- Must be a valid `TenantPlanName` enum value

---

### 8. Status

```csharp
[Required, MaxLength(50)]
public TenantStatus Status { get; set; } = TenantStatus.Active;
```

**Purpose**: Enum indicating the current operational state of the tenant. Used to control access and system availability.

**Type**: `TenantStatus` enum with values:
- `Inactive` (0) - Tenant account is not active
- `Active` (1) - Tenant is operational (default)
- `Suspended` (2) - Tenant access is temporarily suspended

**Database**: Stored as `nvarchar(50)` (string) via Fluent API configuration, defaults to `"Active"`.

**⚠️ Note on `MaxLength` Attribute**: Same as `PlanName` - the `[MaxLength(50)]` attribute doesn't meaningfully apply to enum-to-string conversion. Can be removed if storage type is documented.

**Usage Examples**:

```csharp
// Middleware check
if (tenant.Status != TenantStatus.Active)
{
    return Forbid("Tenant is not active. Please contact support.");
}

// UI banner
if (tenant.Status == TenantStatus.Suspended)
{
    // Show: "Your account is suspended. Please contact support."
}
```

**Default Value**: `TenantStatus.Active` - New tenants are active by default.

**Validation**:
- Required (cannot be null)
- Must be a valid `TenantStatus` enum value

---

### 9. ConnectionString

```csharp
[Required, StringLength(500)]
public string ConnectionString { get; set; } = string.Empty;
```

**Purpose**: The **database connection string** for this tenant's dedicated database. This is the key to multi-tenant data isolation.

**Database**: `nvarchar(500) NOT NULL`, up to 500 characters.

**How It Works**:

```
1. HTTP Request arrives for tenant "alnoor.alforqan.com"
   ↓
2. TenantResolutionMiddleware extracts host header
   ↓
3. Query HostDbContext: SELECT * FROM Tenants WHERE PrimaryDomain = host
   ↓
4. Extract connection string: var connString = tenant.ConnectionString
   ↓
5. Configure AppDbContext per-request:
   options.UseSqlServer(connString)
   ↓
6. All queries now execute against tenant's isolated database
```

**Implementation**: The `TenantResolutionMiddleware` (located in `Alforqan.Api/Middlewares/TenantResolutionMiddleware.cs`) handles tenant resolution per-request. The resolved tenant is stored in `HttpContext.Items["CurrentTenant"]` and accessed via `ITenantContext` service.

**Example Connection String**:
```
Server=sql999.cloudprovider.com,1433;
Database=Alforqan_Tenant_AlNoor;
User Id=tenant_user;
Password=SuperSecret123;
TrustServerCertificate=True;
```

**🔒 Security Critical**:
- **NEVER expose this to the frontend** - it's for backend consumption only
- **NEVER log connection strings** in plain text
- **Encryption at rest**: Implemented via `ISecretsProtector`; when `EncryptionSettings.EncryptTenantConnectionStrings` is true, connection strings are encrypted before save and decrypted in `TenantResolutionMiddleware` after load. See [connection-string-security-explained.md](../features-explanation-docs/connection-string-security-explained.md) and [connection-string-security-guide.md](../technical-docs/connection-string-security-guide.md).
- Use secure password management practices
- Consider using Azure Key Vault or similar for production (same `ISecretsProtector` abstraction can be extended)

**Validation**:
- Required (cannot be null or empty)
- Maximum 500 characters (typical SQL Server connection strings are ~200-300 chars)

**Future Considerations**:
- Connection pooling per tenant
- Automatic failover for high availability

---

### 10. PluginName

```csharp
[StringLength(100)]
public string? PluginName { get; set; }
```

**Purpose**: Optional identifier for the plugin/customization package to use for this tenant. Used by the plugin system to resolve tenant-specific implementations.

**Database**: `nvarchar(100) NULL`, up to 100 characters, **optional**.

**Usage Examples**:
- Custom payment rules: `"ClientXPaymentPlugin"`
- Custom scoring logic: `"MasjidAlNoorScoring"`
- Custom notifications: `"EnterpriseNotificationPlugin"`
- Example values: `"ClientX"`, `"AlNoorCustom"`, `null` (default behavior)

**How Plugin System Works**:
```
1. Tenant resolved from request
   ↓
2. Check tenant.PluginName
   ↓
3. If PluginName is set, resolve plugin implementation via DI
   ↓
4. Use plugin-specific services instead of default implementations
```

**Validation**:
- Optional (nullable)
- If provided, maximum 100 characters
- Should match plugin project name pattern (e.g., `Alforqan.Plugins.ClientX.*`)

**Related**: See `docs/business/Alforqan-Plugins.md` for plugin system guidelines.

---

### 11. Modules (Navigation Property)

```csharp
public ICollection<TenantModule> Modules { get; set; } = new HashSet<TenantModule>();
```

**Purpose**: One-to-Many relationship with `TenantModule` entities. Represents which features/modules are enabled for this tenant.

**Database**: Foreign key relationship (not a column in `Tenants` table).

**Relationship**:
- One `Tenant` can have many `TenantModule` records
- Each `TenantModule` belongs to one `Tenant`
- Managed via `TenantModule.TenantId` foreign key

**Usage Examples**:

```csharp
// Check if module is enabled
if (tenant.Modules.Any(m => m.ModuleKey == TenantModuleKey.Reports && m.IsEnabled))
{
    // Show reports module in UI
}

// Enable module during onboarding
tenant.Modules.Add(new TenantModule
{
    ModuleKey = TenantModuleKey.PaymentsBasic,
    IsEnabled = true
});
```

**Lazy Loading**: Consider using `.Include(t => t.Modules)` when querying if you need modules data.

---

## Id vs Code vs PrimaryDomain - When to Use Each

Understanding the three identifiers is crucial for correct usage:

| Identifier | Type | Purpose | Usage | Unique? |
|------------|------|---------|-------|---------|
| **Id** | Guid | Database primary key | Foreign keys, internal references, API IDs | Yes (by definition) |
| **Code** | string (2-50) | Human-readable short identifier | **Currently: Only seeding**<br>**Potential**: URL slugs, logs, API endpoints | Yes (unique index) |
| **PrimaryDomain** | string (max 200) | HTTP request routing | **Tenant resolution from requests**, subdomain mapping | Yes (unique index) |

### Decision Matrix

**Use `Id` when**:
- Creating foreign key relationships
- Storing tenant reference in another table
- API endpoints that accept tenant identifier
- Internal database references

**Use `PrimaryDomain` when**:
- Resolving tenant from HTTP request (most common)
- Tenant resolution middleware
- Routing requests to correct tenant database
- Subdomain-based access control

**Use `Code` when**:
- **Currently**: Only in seeding/default tenant lookup
- **Future potential**: URL-friendly identifiers, logging, shorter human-readable references

**⚠️ Code Field Recommendation**: See "Code Field Analysis" section below.

---

## Database Mapping

### Table Schema

```sql
CREATE TABLE Tenants (
    Id UNIQUEIDENTIFIER PRIMARY KEY,
    Name NVARCHAR(100) NOT NULL,
    Code NVARCHAR(50) NOT NULL,
    PrimaryDomain NVARCHAR(200) NOT NULL,
    ContactName NVARCHAR(100) NULL,
    ContactEmail NVARCHAR(150) NULL,
    ContactPhone NVARCHAR(30) NULL,
    PlanName NVARCHAR(100) NOT NULL,
    Status NVARCHAR(50) NOT NULL,
    ConnectionString NVARCHAR(500) NOT NULL,
    PluginName NVARCHAR(100) NULL,
    CreatedAt DATETIME2 NOT NULL,
    CreatedBy NVARCHAR(256) NULL,
    ModifiedAt DATETIME2 NULL,
    ModifiedBy NVARCHAR(256) NULL,
    IsDeleted BIT NOT NULL DEFAULT 0,
    DeletedAt DATETIME2 NULL,
    
    CONSTRAINT UQ_Tenants_Code UNIQUE (Code),
    CONSTRAINT UQ_Tenants_PrimaryDomain UNIQUE (PrimaryDomain)
);
```

### EF Core Configuration

Located in `Alforqan.Infrastructure/Data/HostDbContext.cs`:

```csharp
modelBuilder.Entity<Tenant>(entity =>
{
    // Enum to string conversions (cannot be done with data annotations)
    entity.Property(t => t.PlanName).HasConversion<string>();
    entity.Property(t => t.Status).HasConversion<string>();

    // Relationship configuration
    entity.HasMany(t => t.Modules)
          .WithOne(m => m.Tenant)
          .HasForeignKey(m => m.TenantId);

    // Soft delete default value
    entity.Property(t => t.IsDeleted).HasDefaultValue(false);
});
```

**Note**: Indexes are defined via data annotations on the entity class (`[Index]` attributes), not in Fluent API.

---

## Common Use Cases

### 1. Tenant Resolution from HTTP Request

```csharp
// Middleware or service
public async Task<Tenant> ResolveTenantAsync(string host)
{
    var tenant = await _hostDbContext.Tenants
        .FirstOrDefaultAsync(t => t.PrimaryDomain == host && !t.IsDeleted);
    
    if (tenant == null)
        throw new TenantNotFoundException($"No tenant found for domain: {host}");
    
    if (tenant.Status != TenantStatus.Active)
        throw new TenantNotActiveException($"Tenant {tenant.Name} is not active");
    
    return tenant;
}
```

### 2. Configure Tenant Database Context

```csharp
public void ConfigureTenantDbContext(DbContextOptionsBuilder options, Tenant tenant)
{
    options.UseSqlServer(tenant.ConnectionString);
    // Additional tenant-specific configuration
}
```

### 3. Check Plan Limits

```csharp
public bool CanCreateHalaqa(Tenant tenant, int currentHalaqatCount)
{
    return tenant.PlanName switch
    {
        TenantPlanName.Starter => currentHalaqatCount < 3,
        TenantPlanName.Standard => currentHalaqatCount < 10,
        TenantPlanName.Enterprise => true, // Unlimited
        _ => false
    };
}
```

### 4. Enable/Disable Tenant Modules

```csharp
public async Task EnableModuleAsync(Tenant tenant, TenantModuleKey moduleKey)
{
    var module = tenant.Modules.FirstOrDefault(m => m.ModuleKey == moduleKey);
    if (module == null)
    {
        module = new TenantModule
        {
            TenantId = tenant.Id,
            ModuleKey = moduleKey,
            IsEnabled = true
        };
        tenant.Modules.Add(module);
    }
    else
    {
        module.IsEnabled = true;
    }
    await _hostDbContext.SaveChangesAsync();
}
```

---

## Code Field Analysis

### Current State

The `Code` field is **currently underutilized**:
- ✅ Defined in ERD
- ✅ Has unique constraint
- ✅ Used in `HostDbSeeding.cs` for default tenant: `"default"`
- ❌ **Not used** for tenant resolution (that uses `PrimaryDomain`)
- ❌ **Not used** in any API endpoints
- ❌ **Not used** in logs or references

### Question: Is Code Necessary?

**Arguments for Keeping Code**:
1. **URL-friendly slugs**: `/api/tenants/ALNOOR/reports` is cleaner than `/api/tenants/{guid}`
2. **Human-readable logs**: `TenantCode=ALNOOR` is better than `TenantId=a3f8e9c2-...`
3. **API usability**: External systems might prefer short codes over GUIDs
4. **Future flexibility**: Subdomain changes don't break external references
5. **ERD alignment**: It's part of the designed schema

**Arguments for Removing Code**:
1. **Redundancy**: We have `Id` (for DB) and `PrimaryDomain` (for routing)
2. **No current usage**: Only used in seeding
3. **Maintenance overhead**: Another unique constraint to maintain
4. **Potential confusion**: Three identifiers can be confusing

### Recommendation

**Option 1: Keep Code, Define Use Cases**
- Document specific use cases in requirements
- Implement API endpoints using Code: `/api/tenants/{code}/...`
- Use Code in logs and references
- **Action**: Update documentation and implement Code-based lookups where appropriate

**Option 2: Remove Code**
- Remove from entity
- Remove unique index
- Update seeding to use `Id` or `PrimaryDomain`
- **Action**: Create migration to drop column and index
- **Consideration**: Ensure no external systems depend on it

### Decision Needed

**⚠️ Action Required**: Decide whether to:
1. **Keep Code** and implement specific use cases (URL slugs, logging, API endpoints)
2. **Remove Code** and use only `Id` and `PrimaryDomain`

If keeping Code, consider:
- Adding `GetByCodeAsync()` repository method
- Using Code in API routes
- Using Code in logging
- Documenting Code generation rules (uppercase, alphanumeric, etc.)

---

## Relationships

### Tenant → TenantModules (One-to-Many)

```
Tenant (1) ──→ (Many) TenantModule
```

- **Navigation Property**: `Tenant.Modules` (ICollection<TenantModule>)
- **Foreign Key**: `TenantModule.TenantId`
- **Purpose**: Manages which features/modules are enabled per tenant
- **Cascade Delete**: Typically soft delete, no cascade needed

### Future Relationships (Phase 2+)

Based on ERD and requirements:
- Tenant may link to billing/subscription entities
- Tenant may have custom configuration entities
- Tenant may track usage/analytics

---

## Security Considerations

### ConnectionString Security

1. **Never Log Plain Text**:
   ```csharp
   // ❌ BAD
   _logger.LogInformation($"Tenant connection: {tenant.ConnectionString}");
   
   // ✅ GOOD
   _logger.LogInformation($"Tenant {tenant.Code} database configured");
   ```

2. **Encrypt at Rest**: Implemented via `ISecretsProtector`; enable with `EncryptionSettings.EncryptTenantConnectionStrings`. See [connection-string-security-explained.md](../features-explanation-docs/connection-string-security-explained.md) and [connection-string-security-guide.md](../technical-docs/connection-string-security-guide.md).

3. **Secure Storage**: Use Azure Key Vault or similar for production; the same abstraction can be extended for key resolution.

4. **Access Control**: Only admin/system accounts should read connection strings

### Tenant Resolution Security

1. **Validate Host Header**: Ensure host header is not spoofed
2. **Domain Validation**: Validate `PrimaryDomain` against allowed domains
3. **Status Checks**: Always verify tenant is `Active` before allowing access
4. **Rate Limiting**: Implement per-tenant rate limiting

### Authorization

- Tenant resolution should happen early in request pipeline
- Verify user has access to resolved tenant
- Prevent cross-tenant data access

---

## Related Files

- **Entity**: `Alforqan.Data/Entities/Tenants/Tenant.cs`
- **DbContext**: `Alforqan.Infrastructure/Data/HostDbContext.cs`
- **Enums**: 
  - `Alforqan.Data/Enums/Tenants/TenantPlanName.cs`
  - `Alforqan.Data/Enums/Tenants/TenantStatus.cs`
- **Seeding**: `Alforqan.Infrastructure/Seeding/HostDbSeeding.cs`
- **Migrations**: `Alforqan.Infrastructure/Migrations/Host/`
- **Base Class**: `Alforqan.Data/Base/Implementations/FullAuditedEntity.cs`
- **Related Entity**: `Alforqan.Data/Entities/Tenants/TenantModule.cs`

---

## Future Enhancements

### Phase 1
- ✅ **Implemented**: Tenant resolution middleware (`TenantResolutionMiddleware`)
- ✅ **Implemented**: Tenant status checks in middleware
- ✅ **Implemented**: System admin vs tenant admin role distinction
- ✅ **Implemented**: Host endpoint exclusion for tenant management
- ✅ **Implemented**: Default tenant fallback for development
- ⏳ **Pending**: Implement plan-based feature gating

### Phase 2+
- Tenant billing and subscription management
- **Tenant branding:** Per-tenant logo URL, colors, and theme (see [tenant-branding-explained.md](../features-explanation-docs/tenant-branding-explained.md) and [tenant-branding-guide.md](../technical-docs/tenant-branding-guide.md)).
- Tenant-specific configuration settings
- Tenant analytics and usage tracking
- Multi-domain support per tenant
- Tenant backup/restore functionality

---

## Summary

The `Tenant` entity is the **cornerstone of multi-tenant architecture** in Alforqan:

- **Represents**: Each Qur'an center/masjid/institute using the system
- **Stores**: Identity (Name, Code, PrimaryDomain), contact info, subscription details, database connection
- **Enables**: Tenant isolation, subscription management, feature gating
- **Critical for**: Request routing, database configuration, access control

**Key Takeaways**:
1. `PrimaryDomain` is the **primary mechanism** for tenant resolution
2. `ConnectionString` enables **database isolation** per tenant
3. `PlanName` and `Status` control **access and features**
4. `Code` field needs **decision** on purpose or removal
5. All auditing fields come from `FullAuditedEntity` base class
6. Security is **critical** for connection strings and tenant resolution

---

## Quick Reference

### Finding Tenant
- **By Domain**: `PrimaryDomain` (most common)
- **By Code**: `Code` (currently only seeding)
- **By Id**: `Id` (database relationships)

### Important Constraints
- `Code`: Unique, 2-50 characters
- `PrimaryDomain`: Unique, max 200 characters
- `Name`: Required, 2-100 characters
- `ConnectionString`: Required, max 500 characters

### Default Values
- `PlanName`: `TenantPlanName.Starter`
- `Status`: `TenantStatus.Active`
- `IsDeleted`: `false` (from base class)

