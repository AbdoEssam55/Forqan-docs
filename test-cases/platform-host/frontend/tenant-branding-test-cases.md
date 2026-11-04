# Platform host tenant branding — Frontend Test Cases

## Metadata
- Project: `alforqan`
- Feature: `platform-host`
- Layer: `frontend`
- Routes: `/{locale}/tenants/[id]` (Branding tab)
- APIs: Host tenant branding get/update
- Related code: `src/app/[locale]/(host)/tenants/[id]/tenant-detail-content.tsx`, `src/features/branding/use-apply-branding.ts`
- Related docs: `docs/multi-tenant-platform-scope-and-roadmap.md` (section 8.3 Tenant branding)

## Scope

- Host admin configures per-tenant branding (logo URL, colors, theme) from tenant detail.
- Sets up tenant-side branding visibility cases in `platform-tenant/frontend/tenant-branding-test-cases.md`.

## Test pages

- `/{locale}/tenants/[id]` — tab "Branding"

## Test scenarios

- Host admin opens tenant branding settings.
- Host admin saves custom primary color and theme for a demo tenant.

## Delivery waves

| Wave | Intent | Keys | Depends on (cross-file) |
| ---- | ------ | ---- | ----------------------- |
| 1 | Configure tenant branding | `PLAT-HOST-BRD-001`–`002` | `PLAT-HOST-TEN-003` |

## Test cases

### Wave 1 — Tenant branding on host

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-01 | PLAT-HOST-BRD-001 | Verify the Branding tab shows logo, colors, and theme fields | Main | Signed in as host admin with UpdateTenant claim; demo tenant detail is open | 1. Open tenant detail for the demo tenant.<br>2. Select the "Branding" tab.<br>3. Review the form fields. | 1. The section shows "Tenant Branding".<br>2. Fields include logo URL or upload, primary/secondary/accent colors, and theme options (Light, Dark, or System).<br>3. A "Save Branding" button is visible. | PLAT-HOST-TEN-003 | `src/app/[locale]/(host)/tenants/[id]/tenant-detail-content.tsx` | Demo tenant |
| TC-02 | PLAT-HOST-BRD-002 | Verify saving tenant branding persists primary color and theme | Main | On Branding tab for demo tenant | 1. Set primary color to a distinct value (for example `#e11d48`).<br>2. Set theme to "Light".<br>3. Choose "Save Branding".<br>4. Refresh the page and reopen the Branding tab. | 1. A success toast appears after save.<br>2. After refresh, primary color still shows `#e11d48` (or the chosen value).<br>3. Theme remains "Light". | PLAT-HOST-BRD-001 | `src/app/[locale]/(host)/tenants/[id]/tenant-detail-content.tsx` | Primary: `#e11d48`; Theme: Light |
