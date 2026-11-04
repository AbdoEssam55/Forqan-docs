# Tenant Branding Feature

## Overview

Tenant branding allows each tenant to have its own visual identity: logo URL, primary/secondary/accent colors, and theme (light/dark/system). The platform stores branding in the Host DB and exposes it to the tenant app via **TenantConfig** (GET /API/V01/TenantConfig). The frontend can apply these values (e.g. CSS variables, logo, theme) so each tenant’s UI is recognizable and consistent with their brand.

## Why Tenant Branding?

- **White-label:** Each tenant (institute, shop, clinic) can present the app under their own logo and colors.
- **Consistency:** One config source (TenantConfig) drives theme and branding across the tenant app.
- **Generic:** No product-specific branding fields; any product (Qur'an, e-commerce, clinical) can consume the same DTO for theming.

## Data and Flow

- **TenantBranding** (Host DB): 1:1 with Tenant. Fields: LogoUrl (optional), PrimaryColor, SecondaryColor, AccentColor (e.g. hex strings), Theme ("Light" | "Dark" | "System").
- **Read:** GET TenantConfig includes an optional **Branding** object (TenantBrandingDto). If the tenant has no branding row, Branding is null.
- **Write:** Host admin only. PUT Host/Tenants/{tenantId}/branding with TenantBrandingDto creates or updates the tenant’s branding row.

## References

- **Technical guide:** [tenant-branding-guide.md](../technical-docs/tenant-branding-guide.md) – how to use branding in the frontend, how to manage it from the host API.
- **Tenant entity:** [tenant-entity.md](../DB entities/tenant-entity.md).
