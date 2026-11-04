# Platform tenant module gating — Frontend Test Cases

## Metadata
- Project: `alforqan`
- Feature: `platform-tenant`
- Layer: `frontend`
- Routes: `/{locale}/{tenantSlug}`, `/{locale}/{tenantSlug}/reports/...`, `/{locale}/{tenantSlug}/payments`
- APIs: `GET /API/V01/TenantConfig` (modules list)
- Related code: `src/features/alforqan/config/tenant-nav.ts`, `src/features/alforqan/components/tenant-navigation.tsx`, `src/features/tenant/components/module-gate.tsx`, `src/features/alforqan/tenant-config-provider.tsx`
- Related docs: `docs/multi-tenant-platform-overview.md`, `Frontend/alforqan/docs/technical/api/tenant-config-module-gate-guide.md`

## Scope

- TenantConfig-driven UI: navigation hides module-gated items (Reports, Payments).
- Direct URL access to a disabled module shows the module-not-enabled message.
- Does not test Alforqan report or payment business flows — only platform gating.

## Test pages

- `/{locale}/{tenantSlug}` (sidebar navigation)
- `/{locale}/{tenantSlug}/payments` (module-gated route)
- `/{locale}/{tenantSlug}/reports/memorization` (module-gated route)

## Test scenarios

- With Reports enabled, "Reports" appears in nav; with Reports disabled from host, it disappears.
- With Payments (Basic) disabled, "Payments" is hidden and `/payments` is blocked.
- Blocked route shows "This feature is not enabled for your institute." with "Go to dashboard".

## Delivery waves

| Wave | Intent | Keys | Depends on (cross-file) |
| ---- | ------ | ---- | ----------------------- |
| 1 | Reports module nav gating | `PLAT-TEN-MOD-001`–`002` | `PLAT-TEN-AUTH-001`, `PLAT-HOST-MOD-002` |
| 2 | Payments module route gating | `PLAT-TEN-MOD-003`–`004` | `PLAT-TEN-AUTH-001`, `PLAT-HOST-MOD-001` |

**Demo setup:** Use the same demo tenant as host module cases. Tenant user must have claims to view Reports and Payments when modules are enabled.

## Test cases

### Wave 1 — Reports module gating

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-01 | PLAT-TEN-MOD-001 | Verify Reports appears in tenant navigation when the Reports module is enabled | Main | Demo tenant has Reports enabled (`PLAT-HOST-MOD-003` or default); tenant user signed in with permission to view reports | 1. Sign in to the demo tenant.<br>2. Open the institute sidebar navigation.<br>3. Look for a "Reports" item. | 1. "Reports" is visible in the sidebar.<br>2. Choosing "Reports" opens a reports section (sub-routes may vary by product). | PLAT-TEN-AUTH-001, PLAT-HOST-MOD-003 | `src/features/alforqan/components/tenant-navigation.tsx`, `src/features/alforqan/config/tenant-nav.ts` | Demo tenant with Reports on |
| TC-02 | PLAT-TEN-MOD-002 | Verify Reports is hidden from navigation when the module is disabled on host | Main | Host admin disabled Reports for demo tenant (`PLAT-HOST-MOD-002`); tenant user signed in | 1. Sign in to the demo tenant (or refresh after host change).<br>2. Open the institute sidebar.<br>3. Search the nav list for "Reports". | 1. "Reports" is not shown in the sidebar.<br>2. Core items such as "Dashboard" and "Settings" remain visible. | PLAT-TEN-AUTH-001, PLAT-HOST-MOD-002 | `src/features/alforqan/components/tenant-navigation.tsx`, `src/features/alforqan/tenant-config-provider.tsx` | Same demo tenant; Reports off |

### Wave 2 — Payments module route gating

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-03 | PLAT-TEN-MOD-003 | Verify Payments is hidden when Payments (Basic) is disabled for the tenant | Main | Host disabled Payments (Basic) for demo tenant; tenant user signed in with payments view claim | 1. Sign in to the demo tenant.<br>2. Check the sidebar for "Payments". | 1. "Payments" is not listed in the sidebar. | PLAT-TEN-AUTH-001, PLAT-HOST-MOD-001 | `src/features/alforqan/config/tenant-nav.ts` | Demo tenant; PaymentsBasic off |
| TC-04 | PLAT-TEN-MOD-004 | Verify direct navigation to payments shows the module-not-enabled message | Main | Payments (Basic) disabled for demo tenant; tenant user signed in | 1. While signed in, open `/{locale}/{tenantSlug}/payments` directly in the address bar. | 1. The page shows "This feature is not enabled for your institute."<br>2. A "Go to dashboard" link is visible.<br>3. Choosing "Go to dashboard" opens the tenant dashboard. | PLAT-TEN-MOD-003 | `src/features/tenant/components/module-gate.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/payments/layout.tsx` | Demo tenant slug |
