# Tenant Config

## Overview

TenantConfig is the main configuration endpoint for the **tenant app**. The frontend calls it once per session (or when entering tenant context) to get tenant name, enabled modules, branding, feature flags, and optionally plugin name. Used for module gating, branding application, and feature-flag checks. The endpoint runs in tenant context (tenant resolved by host header or header); it may allow anonymous or require tenant auth depending on configuration.

## Base route

`/API/V01/TenantConfig`

## Main operations

- **Get config**: `GET` – returns GetTenantConfigResponse (tenant code, name, modules, branding, featureFlags, plugin name, etc.).
- **Greet POC**: `GET greet-poc?name=...` – proof of concept for plugin system (resolves IGreetingService by tenant plugin). Optional.

## Related documentation

- [Tenant resolution](../features-explanation-docs/tenant-resolution-feature.md)
- [Module gating](../features-explanation-docs/module-gating-on-tenant-endpoints-explained.md)
- [Tenant branding](../technical-docs/tenant-branding-guide.md)
- [Feature flags](../features-explanation-docs/feature-flags-explained.md)

## Controller

`Alforqan.Api/Controllers/Tenants/TenantConfigController.cs`
