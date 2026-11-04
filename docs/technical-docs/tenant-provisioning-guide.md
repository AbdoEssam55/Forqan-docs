# Tenant Provisioning – Technical Guide

## Interface

**ITenantProvisioningService** (Alforqan.Service/Abstractions/Tenants/IServices):

```csharp
Task ProvisionTenantAsync(Guid tenantId, string? connectionStringOrPlaceholder, CancellationToken cancellationToken = default);
```

- **tenantId:** The newly created tenant’s Id.
- **connectionStringOrPlaceholder:** The tenant’s current connection string (may be a placeholder or empty).

Called by **TenantService.AddAsync** after the tenant is saved and default modules are assigned. If no implementation is registered, nothing is called.

## Implementing Provisioning

1. Create a class that implements **ITenantProvisioningService**.
2. In **ProvisionTenantAsync:**  
   - Optionally create the database (e.g. SQL `CREATE DATABASE` or cloud API).  
   - Run the product’s tenant migrations for the new (or placeholder) connection string.  
   - If you generate or change the connection string, update the tenant row in the Host DB (e.g. via ITenantRepo or HostDbContext).
3. Register the implementation in DI (e.g. in Program.cs or a module):

   ```csharp
   services.AddScoped<ITenantProvisioningService, MyTenantProvisioningService>();
   ```

4. Do **not** register a default implementation in the platform if you want to keep current behavior (no automatic DB creation). Register only when you need automated provisioning.

## Example (conceptual)

- Create DB (e.g. `CREATE DATABASE Tenant_<tenantId>`).  
- Build connection string for the new DB.  
- Run EF Core migrations for the tenant schema (e.g. use a separate DbContext or migration runner for tenant DBs).  
- Update the tenant’s ConnectionString in the Host DB (e.g. load tenant by id, set ConnectionString, save; if using encryption, use ISecretsProtector before saving).

## References

- **Explanation:** [tenant-provisioning-explained.md](../features-explanation-docs/tenant-provisioning-explained.md).
- **Interface:** Alforqan.Service/Abstractions/Tenants/IServices/ITenantProvisioningService.cs.
- **Call site:** TenantService.AddAsync (after default modules).
