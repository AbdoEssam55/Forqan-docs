# Tenant Resolution Guide

## How to Access Current Tenant in Services

Inject `ITenantContext`:

```csharp
public class SomeService
{
    private readonly ITenantContext _tenantContext;
    
    public SomeService(ITenantContext tenantContext)
    {
        _tenantContext = tenantContext;
    }
    
    public async Task DoSomething()
    {
        if (!_tenantContext.HasTenant)
        {
            throw new InvalidOperationException("Tenant required");
        }
        
        var tenantCode = _tenantContext.TenantCode;
        var pluginName = _tenantContext.PluginName;
    }
}
```

## ITenantContext Properties

- `Tenant? CurrentTenant` - The resolved tenant (null for host endpoints)
- `bool HasTenant` - Convenience property indicating if tenant is resolved
- `bool IsSystemAdmin` - Computed from HttpContext.User claims
- `string? PluginName` - From CurrentTenant.PluginName
- `string? TenantCode` - From CurrentTenant.Code

## How Tenant Resolution Middleware Works

Behavior depends on `TenantSettings:Mode`:

- **`SingleTenant` (default):** Host routes (`/API/V01/Host/*`) return **404**. All other
  requests always resolve `DefaultTenantCode` (e.g. `alforqan`). Host header and
  `X-Tenant-Code` are ignored. See [single-tenant-strategy.md](../../docs/single-tenant-strategy.md).
- **`MultiHost`:** Host routes skip tenant resolution. Tenant routes resolve by
  PrimaryDomain, optional `X-Tenant-Code` header, or default-tenant fallback.

The middleware injects `ITenantRepo` (not `HostDbContext` directly).

```csharp
public async Task InvokeAsync(HttpContext context, ITenantRepo tenantRepo)
{
    var path = context.Request.Path.Value ?? string.Empty;
    var isHostEndpoint = CustomRouter.HostRouting.IsHostEndpointPath(path);

    if (isHostEndpoint)
    {
        if (_tenantSettings.IsSingleTenant)
        {
            context.Response.StatusCode = StatusCodes.Status404NotFound;
            return;
        }

        await _next(context);
        return;
    }

    var tenant = _tenantSettings.IsSingleTenant
        ? await ResolveFixedTenantAsync(tenantRepo)
        : await ResolveMultiHostTenantAsync(context, tenantRepo, host);

    if (tenant == null)
        throw new TenantNotFoundException(...);

    context.Items["CurrentTenant"] = tenant;
    await _next(context);
}
```

## How to Configure HostDbContext

In `Alforqan.Api/Program.cs`:

```csharp
builder.Services.AddDbContext<HostDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString(ConstantsHelper.ConnectionStringConstants.HostDb)));
```

Connection string in `appsettings.json`:
```json
"ConnectionStrings": {
  "HostDb": "data source=.;integrated security=True;initial catalog=Tenants_Host_DB;..."
}
```

## How to Configure AppDbContext (Tenant-Scoped)

In `Alforqan.Api/Program.cs`:

```csharp
builder.Services.AddDbContext<AppDbContext>((sp, options) =>
{
    var tenantContext = sp.GetRequiredService<ITenantContext>();
    
    if (!tenantContext.HasTenant)
    {
        throw new InvalidOperationException(
            "Tenant-scoped DbContext requested but no tenant is resolved. " +
            "Use HostDbContext for host-level operations.");
    }
    
    var connectionString = tenantContext.CurrentTenant!.ConnectionString;
    options.UseSqlServer(connectionString);
});
```

## How to Configure Default Tenant Fallback (Development Only)

In `appsettings.Development.json`:

```json
{
  "TenantSettings": {
    "AllowDefaultTenantFallback": true,
    "DefaultTenantDomain": "localhost"
  }
}
```

In `appsettings.json` (Production):

```json
{
  "TenantSettings": {
    "AllowDefaultTenantFallback": false,
    "DefaultTenantDomain": null
  }
}
```

## How to Enable Header-Based Tenant Resolution (Dev / Single-Domain)

1. In `appsettings.Development.json` (or your environment config), add under `TenantSettings`:
   - `AllowHeaderResolution: true`
   - Optionally `TenantHeaderName: "X-Tenant-Code"` (default if omitted)

2. When domain lookup fails (e.g. host is localhost), the middleware will read the header named by `TenantHeaderName`. If the value is non-empty, it resolves the tenant by code via `GetTenantByCodeWithModulesAsync(code)`.

3. **Sending the header in dev:**
   - **Postman / API client:** Add header `X-Tenant-Code: <tenant-code>` (e.g. `X-Tenant-Code: Alforqan2`).
   - **Frontend proxy:** Configure your dev proxy to add the header for tenant API requests (e.g. from env or current tenant slug).

4. Do not enable `AllowHeaderResolution` in production when using domain-based resolution with multiple domains. Use only in dev or single-domain scenarios.

## How to Use HostDbContext for System Admin Operations

Use `HostDbContext` for tenant management operations:

```csharp
public class TenantManagementService
{
    private readonly ITenantRepo _tenantRepo; // Uses HostDbContext
    
    public TenantManagementService(ITenantRepo tenantRepo)
    {
        _tenantRepo = tenantRepo;
    }
    
    public async Task<Tenant> CreateTenantAsync(CreateTenantDto dto)
    {
        var tenant = new Tenant { /* ... */ };
        return await _tenantRepo.AddAsync(tenant);
    }
}
```

## How to Use AppDbContext for Tenant-Specific Operations

Use `AppDbContext` for tenant-specific data operations:

```csharp
public class StudentService
{
    private readonly IStudentRepo _studentRepo; // Uses AppDbContext
    
    public StudentService(IStudentRepo studentRepo)
    {
        _studentRepo = studentRepo;
    }
    
    public async Task<Student> GetStudentAsync(int id)
    {
        return await _studentRepo.GetByIdAsync(id);
    }
}
```

## How to Check if User is System Admin

```csharp
if (_tenantContext.IsSystemAdmin)
{
    // System admin logic
}
```

## Error Handling

The middleware throws exceptions that are handled by `ErrorHandlerMiddleware`:

- `TenantNotFoundException` → 404 Not Found
- `TenantNotActiveException` → 403 Forbidden

Do not write JSON responses directly in middleware. Always throw exceptions.

## Files to Reference

- Middleware: `Alforqan.Api/Middlewares/TenantResolutionMiddleware.cs`
- Context Interface: `Alforqan.Service/Abstractions/Tenants/ITenantContext.cs`
- Context Implementation: `Alforqan.Service/Implementation/Tenants/TenantContext.cs`
- Settings: `Alforqan.Shared/Settings/TenantSettings.cs`
- Exceptions: 
  - `Alforqan.Core/Features/Tenants/Shared/Exceptions/TenantNotFoundException.cs`
  - `Alforqan.Core/Features/Tenants/Shared/Exceptions/TenantNotActiveException.cs`
- Route Constants: `Alforqan.Data/AppMetaData/CustomRouter.cs`
