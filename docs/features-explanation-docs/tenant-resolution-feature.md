# Tenant Resolution Feature

## Overview

The tenant resolution feature is responsible for identifying which tenant a request belongs to based on the incoming HTTP request's host header. This is a critical component of the multi-tenant architecture that ensures data isolation and proper routing.

## Problem Statement

In a multi-tenant system, each request must be associated with a specific tenant. Without proper tenant resolution:

- Data from different tenants could be mixed
- Security boundaries would be violated
- System administrators would not be able to manage tenants without being restricted to a specific tenant
- Tenant-specific configurations (like plugins) would not be applied correctly

## Architecture and Design Decisions

### Repository Pattern Instead of Direct DbContext

**Why:** The middleware uses `ITenantRepo` instead of injecting `HostDbContext` directly.

**Benefits:**
- **Separation of Concerns**: Repository encapsulates data access logic
- **Testability**: Easy to mock `ITenantRepo` for unit tests
- **Maintainability**: Changes to data access logic are isolated to repository layer
- **Future Flexibility**: Can add caching, auditing, or other cross-cutting concerns in repository without changing middleware

**Trade-offs:**
- Slight indirection, but the benefits outweigh this minor cost

### Path-Based Host Endpoint Detection

**Why:** Host endpoints (like `/API/V01/Tenants`) are detected by checking the request path, not via headers.

**Security Rationale:**
- **Client Cannot Bypass**: Clients cannot send a header to bypass tenant resolution
- **Server-Side Control**: The server controls which paths are host endpoints
- **Explicit and Clear**: The route constant in `CustomRouter` makes it obvious which endpoints are host-only

**Alternative Considered:**
- Header-based flag (e.g., `isHostTenant` header) was rejected because:
  - Clients could manipulate headers
  - Less secure and harder to audit
  - Path-based detection is more explicit and maintainable

### Domain-Based Resolution as Primary Strategy

**Why:** Tenants are resolved by matching the request's `Host` header to the tenant's `PrimaryDomain`.

**Rationale:**
- **Standard HTTP Practice**: Using the Host header is the standard way to route requests in web applications
- **DNS Integration**: Works seamlessly with DNS configuration
- **User-Friendly**: Each tenant can have their own domain (e.g., `tenant1.alforqan.com`)
- **Scalable**: Can support multiple domains per tenant in the future if needed

**Domain Changes:**
- If a tenant's domain changes, the database must be updated
- This is expected and acceptable - domain changes are infrequent and require administrative action

### Exception-Based Error Handling

**Why:** The middleware throws exceptions (`TenantNotFoundException`, `TenantNotActiveException`) instead of writing JSON responses directly.

**Benefits:**
- **Centralized Error Formatting**: `ErrorHandlerMiddleware` formats all error responses consistently
- **Consistency**: All errors follow the same response structure (`GenericResponse<T>`)
- **Maintainability**: Error response format changes only need to be made in one place
- **Separation of Concerns**: Middleware focuses on resolution logic, not response formatting

**Flow:**
1. Middleware throws `TenantNotFoundException` or `TenantNotActiveException`
2. Exception bubbles up to `ErrorHandlerMiddleware`
3. `ErrorHandlerMiddleware` catches exception and formats standard response
4. Client receives consistent error response

## How It Works

### Request Flow

1. **Request Arrives**: HTTP request comes in with a `Host` header (e.g., `tenant1.alforqan.com`)

2. **Path Check**: Middleware checks if path starts with `CustomRouter.HostRouting.HostEndpointPrefix`
   - If yes: Skip tenant resolution, proceed to next middleware (for system admin operations)
   - If no: Continue to tenant resolution

3. **Domain Lookup**: Middleware calls `ITenantRepo.GetTenantByDomainWithModulesAsync(host)`
   - Repository queries `HostDbContext.Tenants` with `.Include(t => t.Modules)`
   - Filters by `PrimaryDomain == host && !IsDeleted`

4. **Header-Based Resolution** (Optional, when domain lookup fails):
   - If tenant not found and `TenantSettings.AllowHeaderResolution == true`
   - Read the tenant header (e.g. `X-Tenant-Code` from `TenantSettings.TenantHeaderName`)
   - If the value is non-empty, call `ITenantRepo.GetTenantByCodeWithModulesAsync(code)`
   - If a tenant is found, use it (same validation: status Active, then store in context)
   - Use only in dev or single-domain; do not rely on it in production with domain-based resolution

5. **Default Tenant Fallback** (Development Only):
   - If tenant still not found and `TenantSettings.AllowDefaultTenantFallback == true`
   - And host is `localhost` or matches `DefaultTenantDomain`
   - Try to find tenant by `DefaultTenantDomain` or `DefaultTenantCode`
   - Log fallback usage

6. **Tenant Not Found**:
   - Throw `TenantNotFoundException`
   - `ErrorHandlerMiddleware` returns 404 response

7. **Tenant Status Validation**:
   - Check if `tenant.Status == TenantStatus.Active`
   - If not active: Throw `TenantNotActiveException`
   - `ErrorHandlerMiddleware` returns 403 response

8. **Store Tenant**: Store resolved tenant in `HttpContext.Items["CurrentTenant"]`

9. **Continue Pipeline**: Call `_next(context)` to continue middleware pipeline

### Header-Based Resolution (Optional)

When the Host header does not match any tenant (e.g. single API domain or localhost in dev), the middleware can resolve the tenant by a request header (e.g. `X-Tenant-Code`) if **AllowHeaderResolution** is true.

- **When it runs:** Only when domain lookup returns null and **before** default-tenant fallback.
- **Flow:** Read `TenantSettings.TenantHeaderName` (default `X-Tenant-Code`); if the header value is non-empty, call `GetTenantByCodeWithModulesAsync(code)`; if a tenant is found, validate status and store in context (same as domain resolution).
- **Config:** `TenantSettings.AllowHeaderResolution` (bool, default false), `TenantSettings.TenantHeaderName` (string, default `"X-Tenant-Code"`).
- **Security:** Do not enable in production when using domain-based resolution with multiple domains. Use only in dev or when explicitly configured for single-domain scenarios.

### Tenant Context Access

After tenant resolution, services can access the current tenant via `ITenantContext`:

```csharp
public class SomeService
{
    private readonly ITenantContext _tenantContext;
    
    public SomeService(ITenantContext tenantContext)
    {
        _tenantContext = tenantContext;
    }
    
    public void DoWork()
    {
        if (!_tenantContext.HasTenant)
        {
            throw new InvalidOperationException("Tenant required");
        }
        
        var tenant = _tenantContext.CurrentTenant;
        var pluginName = _tenantContext.PluginName;
    }
}
```

## Integration with Existing Systems

### AppDbContext Configuration

`AppDbContext` is configured per-request using the resolved tenant's connection string:

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

### Error Handler Integration

`ErrorHandlerMiddleware` must be registered **before** `TenantResolutionMiddleware` to catch exceptions:

```csharp
app.UseMiddleware<ErrorHandlerMiddleware>(); // First
app.UseRouting();
app.UseMiddleware<TenantResolutionMiddleware>(); // After routing
```

### System Admin Operations

System administrators can access tenant management endpoints without tenant resolution:

- Endpoints under `/API/V01/Tenants` skip tenant resolution
- `ITenantContext.HasTenant` will be `false` for these requests
- Use `HostDbContext` and `ITenantRepo` for tenant management operations

## Security Considerations

1. **Tenant Isolation**: Each tenant's data is isolated via separate connection strings
2. **Path-Based Detection**: Clients cannot bypass tenant resolution via headers
3. **Status Validation**: Inactive or suspended tenants are blocked
4. **Default Fallback**: Only enabled in development, never in production
5. **System Admin Separation**: System admin operations are clearly separated from tenant operations

## Related Files

- **Middleware**: `Alforqan.Api/Middlewares/TenantResolutionMiddleware.cs`
- **Repository Interface**: `Alforqan.Infrastructure/Abstractions/Tenants/IRepositories/ITenantRepo.cs`
- **Repository Implementation**: `Alforqan.Infrastructure/Implementation/Tenants/Repositories/TenantRepo.cs`
- **Context Interface**: `Alforqan.Service/Abstractions/Tenants/ITenantContext.cs`
- **Context Implementation**: `Alforqan.Service/Implementation/Tenants/TenantContext.cs`
- **Settings**: `Alforqan.Shared/Settings/TenantSettings.cs`
- **Route Constants**: `Alforqan.Data/AppMetaData/CustomRouter.cs`
- **Exceptions**: 
  - `Alforqan.Core/Features/Tenants/Shared/Exceptions/TenantNotFoundException.cs`
  - `Alforqan.Core/Features/Tenants/Shared/Exceptions/TenantNotActiveException.cs`
- **Error Handler**: `Alforqan.Api/Middlewares/ErrorHandlerMiddleware.cs`


