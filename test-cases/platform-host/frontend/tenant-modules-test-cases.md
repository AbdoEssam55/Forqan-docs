# Platform host tenant modules — Frontend Test Cases

## Metadata
- Project: `alforqan`
- Feature: `platform-host`
- Layer: `frontend`
- Routes: `/{locale}/tenants/[id]` (Modules tab)
- APIs: Host tenant module toggle endpoints
- Related code: `src/app/[locale]/(host)/tenants/[id]/tenant-detail-content.tsx`, `src/components/tenants/module-toggle.tsx`
- Related docs: `docs/multi-tenant-platform-overview.md`, `Backend/docs/features-explanation-docs/tenant-plugins-and-reports-feature.md`

## Scope

- Per-tenant module enable/disable from the host portal (Core, Reports, Payments (Basic)).
- Foundation for tenant-side module gating demos.

## Test pages

- `/{locale}/tenants/[id]` — tab "Modules"

## Test scenarios

- Host admin views module toggles for a tenant.
- Host admin disables and re-enables a module (single toggle).
- Host admin uses bulk select to enable or disable multiple modules.

## Delivery waves

| Wave | Intent | Keys | Depends on (cross-file) |
| ---- | ------ | ---- | ----------------------- |
| 1 | Module tab and single toggle | `PLAT-HOST-MOD-001`–`003` | `PLAT-HOST-TEN-003` |
| 2 | Bulk module actions | `PLAT-HOST-MOD-004` | Wave 1 |

Use a **demo tenant** dedicated to module toggling so tenant-side cases can rely on a known state.

## Test cases

### Wave 1 — Module tab and single toggle

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-01 | PLAT-HOST-MOD-001 | Verify the Modules tab lists Core, Reports, and Payments (Basic) for a tenant | Main | Signed in as host admin with ManageTenantModules claim; demo tenant detail is open | 1. Open tenant detail for the demo tenant.<br>2. Select the "Modules" tab.<br>3. Review the module list. | 1. The section heading shows "Module Management".<br>2. Modules include "Core", "Reports", and "Payments (Basic)" (or equivalent labels from the module catalog).<br>3. Each module shows an enabled/disabled control. | PLAT-HOST-TEN-003 | `src/app/[locale]/(host)/tenants/[id]/tenant-detail-content.tsx`, `src/components/tenants/module-toggle.tsx` | Demo tenant slug for later tenant tests |
| TC-02 | PLAT-HOST-MOD-002 | Verify disabling Reports for a tenant updates the module state | Main | Demo tenant has Reports enabled; host admin on Modules tab | 1. Turn off the Reports module for the demo tenant.<br>2. Confirm the UI shows Reports as disabled.<br>3. Refresh the page and reopen the Modules tab. | 1. After toggle, Reports shows as disabled without errors.<br>2. After refresh, Reports remains disabled.<br>3. Core stays enabled (Core cannot be fully removed from the tenant). | PLAT-HOST-MOD-001 | `src/components/tenants/module-toggle.tsx` | Demo tenant; disable Reports only |
| TC-03 | PLAT-HOST-MOD-003 | Verify re-enabling Reports restores the module for the tenant | Main | Demo tenant has Reports disabled from `PLAT-HOST-MOD-002` | 1. On the Modules tab, turn Reports back on.<br>2. Refresh and confirm state. | 1. Reports shows as enabled after toggle.<br>2. After refresh, Reports remains enabled. | PLAT-HOST-MOD-002 | `src/components/tenants/module-toggle.tsx` | Same demo tenant |

### Wave 2 — Bulk module actions

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-04 | PLAT-HOST-MOD-004 | Verify bulk deselect and enable actions on the Modules tab | Main | Host admin on Modules tab for demo tenant; Reports and Payments (Basic) are enabled | 1. Choose "Deselect All" (or clear all optional modules).<br>2. Apply disable for selected modules if prompted.<br>3. Choose "Select All", then "Enable Multiple" (or equivalent bulk enable). | 1. Optional modules can be disabled in bulk; Core remains on.<br>2. Bulk enable turns optional modules back on.<br>3. A selection count such as "{count} selected" updates as modules are checked. | PLAT-HOST-MOD-001 | `src/app/[locale]/(host)/tenants/[id]/tenant-detail-content.tsx` | Demo tenant |
