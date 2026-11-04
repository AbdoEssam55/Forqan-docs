# Host Branding Feature

## Problem statement

The **platform admin UI** (host app) needs its own visual identity: logo, primary/secondary/accent colors, theme (Light/Dark/System), and optional typography. This is separate from tenant branding: tenants have per-tenant branding for their own apps; the host has a single, platform-wide branding applied to the host dashboard and host-only pages.

## Architecture and design

- **Entity**: `HostBranding` (Platform.Data/Entities/Host) – single-row table in Host DB: LogoUrl, PrimaryColor, SecondaryColor, AccentColor, Theme, plus optional FontFamilySans, FontFamilyArabic, FontSizeBase, SpacingUnit, BorderRadius.
- **Storage**: One row per deployment (or default when no row exists). Same DTO shape as tenant branding (`TenantBrandingDto`) for consistency and reuse in the frontend.
- **API**: `/API/V01/Host/Branding` – GET (returns current or defaults), PUT (upsert), POST UploadLogo (returns URL path). All require HostScheme auth.
- **Service**: `IHostBrandingService` / `HostBrandingService` use `IHostBrandingRepo` to get/upsert; logo upload is handled in the controller via `IFileService` (e.g. host logo folder).

## How it works

1. Host admin opens Host Settings (or equivalent) and updates branding (colors, theme, logo). Frontend calls PUT Host/Branding and optionally POST UploadLogo.
2. Host app (e.g. host layout) loads branding on startup or when entering host area: GET Host/Branding, then applies CSS variables and theme so all host UI uses the same identity.
3. Tenant branding is independent: each tenant has TenantBranding rows; tenant app reads branding via TenantConfig. Host branding applies only to the host app.

## Platform vs tenant branding

| Aspect        | Host branding              | Tenant branding                    |
|---------------|----------------------------|-------------------------------------|
| Scope         | Platform admin UI only     | One per tenant; tenant app          |
| Storage       | Host DB, single row        | Host DB, TenantBranding per tenant |
| API           | /API/V01/Host/Branding     | Host/Tenants/{id}/branding + TenantConfig for read |
| Who configures| System admins              | Host admins (per tenant)            |

## Related files

- `Platform.Data/Entities/Host/HostBranding.cs`
- `Alforqan.Api/Controllers/Host/Branding/HostBrandingController.cs`
- `Alforqan.Core/Features/Host/` (GetHostBrandingQuery, UpdateHostBrandingCommand)
- `Alforqan.Service/Implementation/Host/Services/HostBrandingService.cs`
- Feature doc: [host-branding.md](../features/host-branding.md)
