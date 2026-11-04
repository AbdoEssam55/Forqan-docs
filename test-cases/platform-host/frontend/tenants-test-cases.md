# Platform host tenants — Frontend Test Cases

## Metadata
- Project: `alforqan`
- Feature: `platform-host`
- Layer: `frontend`
- Routes: `/{locale}/tenants`, `/{locale}/tenants/create`, `/{locale}/tenants/[id]`
- APIs: Host tenant CRUD under `/API/V01/Host/Tenants`
- Related code: `src/app/[locale]/(host)/tenants/tenants-list-content.tsx`, `src/app/[locale]/(host)/tenants/create/tenant-create-content.tsx`, `src/app/[locale]/(host)/tenants/[id]/tenant-detail-content.tsx`, `src/features/tenants/schemas/tenant-schema.ts`, `src/features/alforqan/components/tenant-live-updates-listener.tsx`, `src/hooks/use-tenant-live-updates.ts`, `Backend/Alforqan/Alforqan.Api/Services/HostEntityNotificationService.cs`
- Related docs: `docs/multi-tenant-platform-overview.md`, `Backend/docs/DB entities/tenant-entity.md`

## Scope

- Listing, filtering, viewing, and creating tenants from the host portal.
- Tenant detail Information tab (status, plan, contact) — core multi-tenant admin workflow.

## Test pages

- `/{locale}/tenants`
- `/{locale}/tenants/create`
- `/{locale}/tenants/[id]`

## Test scenarios

- Host admin browses and filters the tenant list.
- Host admin opens tenant detail and sees status and plan.
- Host admin starts the create-tenant wizard (demo of onboarding flow).

## Delivery waves

| Wave | Intent | Keys | Depends on (cross-file) |
| ---- | ------ | ---- | ----------------------- |
| 1 | Tenant list and detail | `PLAT-HOST-TEN-001`–`003` | `PLAT-HOST-AUTH-001` |
| 2 | Create tenant wizard | `PLAT-HOST-TEN-004` | Wave 1 |

## Test cases

### Wave 1 — Tenant list and detail

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-01 | PLAT-HOST-TEN-001 | Verify the host tenants list shows tenants with status and plan columns | Main | Signed in as host admin with ViewTenants claim; at least two tenants exist | 1. Open `/{locale}/tenants` from the sidebar "Tenants" link.<br>2. Review the table columns and at least one data row. | 1. The heading shows "Tenants Management".<br>2. The table includes columns for Name, Code, Primary Domain, Plan, Support Plan, Status, and Modules.<br>3. Each row shows a status label such as "Active", "Suspended", or "Inactive". | PLAT-HOST-AUTH-001 | `src/app/[locale]/(host)/tenants/tenants-list-content.tsx` | Demo tenants in host DB |
| TC-02 | PLAT-HOST-TEN-002 | Verify tenant list filters by status and plan | Main | Same as `PLAT-HOST-TEN-001` | 1. On the tenants list, open the status filter and choose "Active".<br>2. Clear or widen the filter, then open the plan filter and choose a plan that exists (for example "Standard"). | 1. After the status filter, only rows with status "Active" remain (or an empty state if none match).<br>2. After the plan filter, only rows on the chosen plan remain. | PLAT-HOST-TEN-001 | `src/app/[locale]/(host)/tenants/tenants-list-content.tsx` | At least one Active tenant |
| TC-03 | PLAT-HOST-TEN-003 | Verify opening a tenant shows the Information tab with core fields | Main | Signed in as host admin; a known demo tenant exists | 1. From the tenants list, choose "View" on a demo tenant row.<br>2. Confirm the "Information" tab is selected.<br>3. Review the basic and contact sections. | 1. The page heading area shows "Tenant Details".<br>2. The "Information" tab is visible alongside tabs such as "Modules" and "Branding".<br>3. Basic fields show tenant name, code, primary domain, plan, and status. | PLAT-HOST-TEN-001 | `src/app/[locale]/(host)/tenants/[id]/tenant-detail-content.tsx` | Known demo tenant id |

### Wave 2 — Create tenant wizard

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-04 | PLAT-HOST-TEN-004 | Verify the create-tenant wizard shows the three setup steps | Main | Signed in as host admin with permission to create tenants | 1. From the tenants list, choose "Create New Tenant".<br>2. Review step labels on the wizard.<br>3. Fill required fields on "Basic Information" (name, code, primary domain) and choose "Next" through each step without submitting. | 1. The heading shows "Create New Tenant".<br>2. Wizard steps are labeled "Basic Information", "Contact Information", and "Configuration".<br>3. "Previous" and "Next" move between steps; the final step shows "Create Tenant". | PLAT-HOST-TEN-001 | `src/app/[locale]/(host)/tenants/create/tenant-create-content.tsx`, `src/features/tenants/components/tenant-form.tsx` | Do not submit in demo unless using a throwaway code |
| TC-05 | PLAT-HOST-TEN-005 | Verify localhost is accepted as primary domain on create and edit | Main | Signed in as host admin; a tenant with primary domain "localhost" exists or can be created | 1. Open tenant edit (or create wizard step 1) and set primary domain to "localhost".<br>2. Change only the tenant name and save (edit) or advance past step 1 (create). | 1. No validation message "Please enter a valid domain" / "يرجى إدخال نطاق صحيح" appears for "localhost".<br>2. Edit save succeeds with a success message (no HTTP 500).<br>3. Create wizard allows "Next" on step 1 with primary domain "localhost". | PLAT-HOST-TEN-001 | `src/features/tenants/schemas/tenant-schema.ts`, `src/features/tenants/components/tenant-form.tsx`, `Backend/Alforqan/Alforqan.Api/Middlewares/TenantResolutionMiddleware.cs` | Tenant id with domain localhost |
| TC-06 | PLAT-HOST-TEN-006 | Verify tenant app shows reload dialog when host updates the institute | Main | Host admin in one browser; tenant user logged into the same institute in another | 1. In the tenant app (`/{locale}/{tenantSlug}/...`), open the dashboard and stay on the page.<br>2. In the host portal, open that tenant's detail and change the tenant name (or branding/module) and save.<br>3. Return to the tenant app tab without refreshing manually. | 1. The host save succeeds with no reload dialog on the host tenant detail page.<br>2. The tenant app shows a blocking dialog titled "Institute settings updated" / "تم تحديث إعدادات المعهد".<br>3. Clicking "Reload page" shows the updated settings. | PLAT-HOST-TEN-003 | `src/features/alforqan/components/tenant-live-updates-listener.tsx`, `src/hooks/use-tenant-live-updates.ts`, `Backend/Alforqan/Alforqan.Api/Services/HostEntityNotificationService.cs` | Host + tenant browser sessions |
