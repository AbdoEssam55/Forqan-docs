# Tenant Provisioning Feature

## Overview

When a new tenant is created, the platform can optionally call a **provisioning** implementation (e.g. create the tenant’s database, run migrations, update the tenant’s connection string). The platform only defines the **interface** and **when** it is invoked; the actual provisioning logic (create DB, which migrations) is implemented by the product or a separate tool. No default implementation is registered, so existing behavior is unchanged unless you register one.

## Why Provisioning?

- **Automation:** New tenants can get a database and schema without manual steps.
- **Placeholder connection strings:** You can create a tenant with a placeholder connection string; provisioning creates the DB and updates the tenant row with the real connection string.
- **Generic:** The platform provides the hook only; each product or deployment plugs in its own logic (e.g. run product’s tenant migrations).

## Flow

1. Host admin creates a tenant (e.g. POST Host/Tenants with connection string or placeholder).
2. TenantService.AddAsync saves the tenant and assigns default modules from the plan.
3. TenantService resolves **ITenantProvisioningService** (optional). If registered, it calls **ProvisionTenantAsync(tenantId, connectionString)**.
4. The implementation may create the database, run migrations, and update the tenant’s connection string in the Host DB.
5. TenantService returns the response (it does not re-load the tenant after provisioning).

## References

- **Technical guide:** [tenant-provisioning-guide.md](../technical-docs/tenant-provisioning-guide.md) – how to implement the interface, register it, and optionally update the tenant’s connection string.
