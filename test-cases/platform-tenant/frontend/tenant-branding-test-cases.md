# Platform tenant branding — Frontend Test Cases

## Metadata
- Project: `alforqan`
- Feature: `platform-tenant`
- Layer: `frontend`
- Routes: `/{locale}/{tenantSlug}`, `/{locale}/{tenantSlug}/auth/login`
- APIs: `GET /API/V01/TenantConfig` (branding payload)
- Related code: `src/features/alforqan/tenant-config-provider.tsx`, `src/features/alforqan/components/tenant-branding-apply.tsx`, `src/features/branding/use-apply-branding.ts`, `src/features/alforqan/components/tenant-navigation.tsx`
- Related docs: `docs/multi-tenant-platform-scope-and-roadmap.md` (section 8.3)

## Scope

- Per-tenant branding applied in the institute app after host configuration.
- Logo in navigation and theme/color tokens from TenantConfig.

## Test pages

- `/{locale}/{tenantSlug}` (signed-in shell)
- `/{locale}/{tenantSlug}/auth/login` (optional logo on login)

## Test scenarios

- After host saves branding, tenant app reflects tenant name and custom primary color.
- Institute header shows configured logo when a logo URL is set.

## Delivery waves

| Wave | Intent | Keys | Depends on (cross-file) |
| ---- | ------ | ---- | ----------------------- |
| 1 | Tenant-visible branding | `PLAT-TEN-BRD-001`–`002` | `PLAT-TEN-AUTH-001`, `PLAT-HOST-BRD-002` |

## Test cases

### Wave 1 — Branding in tenant app

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-01 | PLAT-TEN-BRD-001 | Verify the tenant app shows the institute name from TenantConfig in the sidebar header | Main | Demo tenant branding saved on host; tenant user signed in | 1. Sign in to the demo tenant.<br>2. Look at the sidebar header above the navigation links. | 1. The header shows the tenant display name (for example the institute name configured on host), not a generic placeholder.<br>2. A sublabel such as "Dashboard" appears under the institute name. | PLAT-TEN-AUTH-001, PLAT-HOST-BRD-002 | `src/features/alforqan/components/tenant-navigation.tsx`, `src/features/alforqan/tenant-config-provider.tsx` | Demo tenant name |
| TC-02 | PLAT-TEN-BRD-002 | Verify custom primary color from host branding applies in the tenant UI | Main | Host set primary color to `#e11d48` for demo tenant (`PLAT-HOST-BRD-002`); tenant user signed in | 1. Sign in to the demo tenant.<br>2. Inspect a primary-styled control (for example an active nav item or primary button). | 1. Primary accents reflect the configured color (visually distinct from default purple).<br>2. No "Failed to load institute settings." banner appears. | PLAT-TEN-BRD-001, PLAT-HOST-BRD-002 | `src/features/alforqan/components/tenant-branding-apply.tsx`, `src/features/branding/use-apply-branding.ts` | Primary: `#e11d48` |
