# Host Branding

## Overview

Host (platform) branding controls the visual identity of the **platform admin UI**: logo, primary/secondary/accent colors, theme (Light/Dark/System), and optional typography (font family, size, spacing, border radius). A single row is stored in the Host DB; same DTO shape as tenant branding. All endpoints require host authentication (`HostScheme`) and appropriate host claims.

## Base route

`/API/V01/Host/Branding`

## Main operations

- **Get**: `GET` – returns current host branding or defaults when no row exists
- **Upsert**: `PUT` – create or update with body (logoUrl, primaryColor, secondaryColor, accentColor, theme, fontFamilySans, fontFamilyArabic, fontSizeBase, spacingUnit, borderRadius)
- **Upload logo**: `POST UploadLogo` – upload file; returns URL path (e.g. /images/host/logos/...)

Tenant branding is separate: see Tenant Branding (Host Tenants API) for per-tenant branding.

## Related documentation

- [Host branding feature (explanation)](../features-explanation-docs/host-branding-feature.md) – platform vs tenant branding, usage
- [Tenant branding](../technical-docs/tenant-branding-guide.md) – for tenant-side branding and host management of tenant branding

## Controller

`Alforqan.Api/Controllers/Host/Branding/HostBrandingController.cs`
