# Modals UI overhaul — Frontend Test Cases

## Metadata
- Project: `alforqan`
- Feature: `ui`
- Layer: `frontend`
- Routes: `/{locale}/{tenantSlug}/students`, `/{locale}/{tenantSlug}/teachers`, `/{locale}/{tenantSlug}/parents`, `/{locale}/{tenantSlug}/halaqat`, `/{locale}/{tenantSlug}/reports/memorization`, `/{locale}/{tenantSlug}/reports/sessions`
- Related code: `src/components/brand/confirm-dialog.tsx`, `src/components/ui/alert-dialog.tsx`, `src/components/ui/dialog.tsx`, `src/components/ui/sheet.tsx`, `src/components/ui/sonner.tsx`
- Related docs: `docs/technical/tenant/modals-ui-overhaul.md`

## Scope

- Phase 9 confirm/info/form dialog chrome, slide-overs, toasts, and SweetAlert replacement on overhauled lists.
- Does not change create/edit page routes (still full pages, not quick-create modals).

## Delivery waves

| Wave | Intent | Keys |
| ---- | ------ | ---- |
| 1 | Single deactivate confirm | `ALFQ-MOD-UI-001`–`002` |
| 2 | Bulk deactivate confirm | `ALFQ-MOD-UI-003` |
| 3 | Delete confirm | `ALFQ-MOD-UI-004`–`005` |
| 4 | Clear-filters confirm + toasts | `ALFQ-MOD-UI-006`–`007` |

## Test cases

### Wave 1 — Single confirm

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-01 | ALFQ-MOD-UI-001 | Verify parent deactivate opens ConfirmDialog not SweetAlert | Main | User can manage parent users; at least one active linked parent | 1. Open parents list.<br>2. Open a parent row menu.<br>3. Click "Deactivate" / "تعطيل". | 1. An in-app confirm dialog appears (not a SweetAlert popup).<br>2. Title matches the deactivate confirm copy.<br>3. Parent name is shown in the description. | - | `src/components/brand/confirm-dialog.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/parents/parents-list-content.tsx` | Active parent with `userId` |
| TC-02 | ALFQ-MOD-UI-002 | Verify Cancel on deactivate confirm does not deactivate | Main | Confirm dialog open from TC-01 | 1. Click "Cancel" / "إلغاء". | 1. Dialog closes.<br>2. Parent stays active.<br>3. No success toast. | ALFQ-MOD-UI-001 | `src/components/brand/confirm-dialog.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/parents/parents-list-content.tsx` | Same parent |

### Wave 2 — Bulk confirm

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-03 | ALFQ-MOD-UI-003 | Verify bulk deactivate uses ConfirmDialog then deactivates on confirm | Main | User has manage-user claim; two or more active linked students | 1. Open students list.<br>2. Select two students.<br>3. Click "Deactivate selected".<br>4. Confirm in the dialog. | 1. Confirm dialog shows the bulk count copy.<br>2. Confirm button is destructive (rose), not emerald.<br>3. After confirm, selected students deactivate and the list refreshes. | - | `src/components/brand/confirm-dialog.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/students/students-list-content.tsx` | Active students with `userId` |

### Wave 3 — Delete confirm

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-04 | ALFQ-MOD-UI-004 | Verify halaqa delete confirm shows the halaqa name | Main | User can delete a halaqa | 1. Open halaqat list.<br>2. Choose Delete on a halaqa.<br>3. Read the dialog. | 1. Confirm dialog title is "Delete" / "حذف".<br>2. Description is the halaqa name.<br>3. Cancel closes without deleting. | - | `src/components/brand/confirm-dialog.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/halaqat/halaqat-list-content.tsx` | Existing halaqa |
| TC-05 | ALFQ-MOD-UI-005 | Verify memorization report delete confirm then removes the row | Main | Staff can delete a memorization report | 1. Open `/reports/memorization`.<br>2. Choose Delete on a report.<br>3. Confirm. | 1. Dialog shows student name and report date.<br>2. After confirm the row is gone and a success toast appears. | - | `src/components/brand/confirm-dialog.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/reports/memorization/memorization-reports-list-content.tsx`, `src/components/ui/sonner.tsx` | Existing report |

### Wave 4 — Filters and toasts

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-06 | ALFQ-MOD-UI-006 | Verify session clear-filters confirm uses emerald confirm not rose | Main | Staff on `/reports/sessions` with filters applied | 1. Click "Show filters".<br>2. Click "Clear filters".<br>3. Read the dialog, then confirm. | 1. Confirm dialog title is "Clear all filters?" / "مسح كل التصفية؟".<br>2. Confirm button is emerald (not rose).<br>3. After confirm, filters reset to this month. | ALFQ-SSR-UI-015 | `src/components/brand/confirm-dialog.tsx`, `src/features/alforqan/reports/staff-session-reports-page.tsx` | Filtered sessions |
| TC-07 | ALFQ-MOD-UI-007 | Verify success and error toasts use emerald and rose chrome | Main | Any overhauled list that can toast | 1. Complete a successful deactivate or delete.<br>2. Trigger a failed API action if possible. | 1. Success toast uses emerald border/icon.<br>2. Error toast uses rose border/icon when an error occurs. | ALFQ-MOD-UI-001 | `src/components/ui/sonner.tsx` | Success path |
