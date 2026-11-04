# Settings & admin UI overhaul — Frontend Test Cases

## Metadata
- Project: `alforqan`
- Feature: `ui`
- Layer: `frontend`
- Routes: `/{locale}/{tenantSlug}/settings`, `/{locale}/{tenantSlug}/settings/grading`, `/{locale}/{tenantSlug}/settings/sard-plan`, `/{locale}/{tenantSlug}/users`, `/{locale}/{tenantSlug}/users/{id}`
- Related code: `src/components/brand/settings-shell.tsx`, `src/components/brand/settings-section.tsx`, `src/components/brand/settings-actions.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/users/users-list-content.tsx`
- Related docs: `docs/technical/tenant/settings-admin-ui-overhaul.md`

## Scope

- Phase 13 settings sidebar, form sections, save/cancel, switches, tenant users hub, user detail, roles/permissions matrix. Host admin pages are out of scope.

## Delivery waves

| Wave | Intent | Keys |
| ---- | ------ | ---- |
| 1 | Settings chrome | `ALFQ-SET-UI-001`–`004` |
| 2 | Admin users + permissions | `ALFQ-SET-UI-005`–`007` |

## Test cases

### Wave 1 — Settings

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-01 | ALFQ-SET-UI-001 | Verify settings uses a sidebar with Account, Grading, and Sard plan | Main | Tenant admin (or user with grading and sard claims) | 1. Open `/{tenantSlug}/settings`.<br>2. Read the settings sidebar (left on LTR, top on small screens). | 1. A "Settings sections" nav is visible.<br>2. Links labeled "Account", "Grading", and "Sard plan" are visible (or Arabic equivalents).<br>3. "Account" is the active (emerald) item. | - | `src/components/brand/settings-shell.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/settings/tenant-settings-content.tsx` | Tenant admin |
| TC-02 | ALFQ-SET-UI-002 | Verify grading settings use titled form sections | Main | User can view halaqat | 1. From Settings, click "Grading".<br>2. Read the main column. | 1. Content sits in rounded emerald-bordered section cards with titles.<br>2. Weight rows and student overrides live in separate sections. | ALFQ-SET-UI-001 | `src/components/brand/settings-section.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/settings/grading/tenant-grading-settings-content.tsx` | - |
| TC-03 | ALFQ-SET-UI-003 | Verify grading weights show emerald Save and Cancel | Main | User can update halaqa | 1. Open grading settings.<br>2. Change a weight value.<br>3. Look at the weights section footer. | 1. An emerald "Save weights" button is visible.<br>2. An outline Cancel button is visible next to it. | ALFQ-SET-UI-002 | `src/components/brand/settings-actions.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/settings/grading/tenant-grading-settings-content.tsx` | Weight: "0.5" |
| TC-04 | ALFQ-SET-UI-004 | Verify an active toggle uses an emerald/teal track | Main | User can update halaqa | 1. Open grading settings.<br>2. Turn a mistake-type "Active" switch on. | 1. The on state uses an emerald/teal fill (not violet/blue).<br>2. The thumb sits at the end of the track. | ALFQ-SET-UI-002 | `src/components/ui/switch.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/settings/grading/tenant-grading-settings-content.tsx` | - |

### Wave 2 — Admin

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-05 | ALFQ-SET-UI-005 | Verify the institute users hub shows role cards | Main | User can view tenant users | 1. Open `/{tenantSlug}/users` with no role query.<br>2. Read the hub grid. | 1. Cards for students, teachers, parents, admin, supervisor, and no-role are visible.<br>2. Each card has an emerald icon tile and a count. | - | `src/app/[locale]/(tenant)/[tenantSlug]/users/users-list-content.tsx`, `src/lib/brand/settings.ts` | - |
| TC-06 | ALFQ-SET-UI-006 | Verify user detail deactivate uses ConfirmDialog, not a browser alert | Main | User can deactivate an active tenant user | 1. Open an active user at `/{tenantSlug}/users/{id}`.<br>2. Click "Deactivate".<br>3. Read the dialog. | 1. An in-app confirm dialog opens (not SweetAlert2 / a browser confirm).<br>2. Confirm and Cancel are visible.<br>3. Emerald tab chrome is used for Information / Roles / Password. | - | `src/app/[locale]/(tenant)/[tenantSlug]/users/[id]/tenant-user-detail-content.tsx`, `src/components/brand/confirm-dialog.tsx` | At least one active user |
| TC-07 | ALFQ-SET-UI-007 | Verify roles and permissions use emerald matrix cards and a combined save | Main | User can modify user roles and claims | 1. Open a tenant user detail Roles tab.<br>2. Tick a role so claims change.<br>3. Look at the save control. | 1. Roles and permissions sit in emerald-bordered cards.<br>2. Selected role/claim rows use an emerald highlight.<br>3. A single emerald Save button appears for combined roles and permissions. | ALFQ-SET-UI-006 | `src/components/users/user-roles-and-claims-tab.tsx`, `src/components/users/user-roles-manager.tsx`, `src/components/users/user-claims-manager.tsx` | User with roles tab |
