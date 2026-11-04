# Tenant Branding – Technical Guide

## Data Model

- **TenantBranding** (Host DB): TenantId (unique), LogoUrl (optional), PrimaryColor, SecondaryColor, AccentColor (strings, e.g. hex), Theme ("Light" | "Dark" | "System").
- **Relationship:** 1:1 with Tenant; optional (tenant can have no branding).

## Reading Branding (Tenant App)

1. Call **GET /API/V01/TenantConfig** (tenant-scoped).
2. Response includes **Branding** (nullable): `{ logoUrl?, primaryColor, secondaryColor, accentColor, theme }`.
3. If Branding is null, use app defaults or omit theming.
4. Apply values: set CSS variables from colors, set logo src from logoUrl, apply theme (light/dark/system).

## Managing Branding (Host Admin)

1. **PUT** **/API/V01/Host/Tenants/{tenantId}/branding** with body:
   ```json
   {
     "logoUrl": "https://example.com/logo.png",
     "primaryColor": "#1976d2",
     "secondaryColor": "#424242",
     "accentColor": "#ff9800",
     "theme": "System"
   }
   ```
2. Creates a TenantBranding row if none exists, or updates the existing one for that tenant.
3. Requires host auth and **UpdateTenantClaim** (or equivalent host tenant update policy).

## Defaults

- If no branding row exists, TenantConfig returns `branding: null`.
- When creating/updating, omitted or null color/theme values default to: PrimaryColor "#1976d2", SecondaryColor "#424242", AccentColor "#ff9800", Theme "System".

## References

- **Explanation:** [tenant-branding-explained.md](../features-explanation-docs/tenant-branding-explained.md).
- **Entity:** Alforqan.Data/Entities/Tenants/TenantBranding.cs.
- **DTO:** Alforqan.Core/Features/Tenants/Queries/Models/TenantBrandingDto.cs.
