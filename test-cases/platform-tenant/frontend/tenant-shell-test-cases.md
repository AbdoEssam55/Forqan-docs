# Platform tenant shell — Frontend Test Cases

## Metadata
- Project: `alforqan`
- Feature: `platform-tenant`
- Layer: `frontend`
- Routes: `/{locale}/{tenantSlug}`, `/{locale}/{tenantSlug}/settings`
- APIs: `GET /API/V01/TenantConfig`
- Related code: `src/app/[locale]/(tenant)/[tenantSlug]/layout.tsx`, `src/features/alforqan/components/tenant-navigation.tsx`, `src/components/layout/tenant-top-chrome.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/dashboard-content.tsx`
- Related docs: `docs/multi-tenant-platform-overview.md`, `Frontend/alforqan/docs/technical/tenant/tenant-navigation-guide.md`

## Scope

- Institute app shell: dashboard landing, sidebar navigation structure, settings entry, language control.
- Platform demo only — not Alforqan business pages (halaqat, students, etc.).

## Test pages

- `/{locale}/{tenantSlug}`
- `/{locale}/{tenantSlug}/settings`

## Test scenarios

- Signed-in user lands on dashboard with institute layout (sidebar + top bar).
- Settings page opens from navigation.
- Language switcher changes locale on a tenant page.

## Delivery waves

| Wave | Intent | Keys | Depends on (cross-file) |
| ---- | ------ | ---- | ----------------------- |
| 1 | Tenant app shell | `PLAT-TEN-SHL-001`–`003` | `PLAT-TEN-AUTH-001` |

## Test cases

### Wave 1 — Institute shell

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-01 | PLAT-TEN-SHL-001 | Verify the signed-in tenant dashboard shows the institute layout | Main | Tenant user signed in to demo tenant | 1. Open `/{locale}/{tenantSlug}` for the demo tenant.<br>2. Review the page layout. | 1. The main heading shows "Dashboard".<br>2. The subtitle shows "Overview of your institute".<br>3. A sidebar lists navigation items including "Dashboard" and "Settings".<br>4. The top bar includes a language control. | PLAT-TEN-AUTH-001 | `src/app/[locale]/(tenant)/[tenantSlug]/dashboard-content.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/layout.tsx` | Demo tenant slug |
| TC-02 | PLAT-TEN-SHL-002 | Verify Settings opens from the institute sidebar | Main | Tenant user signed in | 1. In the sidebar, choose "Settings". | 1. The page heading shows "Settings".<br>2. A "Log Out From All Devices" action is visible (platform security option). | PLAT-TEN-SHL-001 | `src/app/[locale]/(tenant)/[tenantSlug]/settings/tenant-settings-content.tsx` | — |
| TC-03 | PLAT-TEN-SHL-003 | Verify the language switcher changes site locale on a tenant page | Main | Tenant user signed in; page locale is English | 1. On any signed-in tenant page, open the language control in the top bar.<br>2. Switch to Arabic.<br>3. Confirm the URL locale segment and reading direction. | 1. The URL uses the Arabic locale segment (for example `/ar/`).<br>2. Layout direction is right-to-left.<br>3. Navigation labels show Arabic equivalents (for example dashboard label in Arabic). | PLAT-TEN-SHL-001 | `src/components/layout/tenant-top-chrome.tsx` | Locale: en → ar |
