# Teachers UI overhaul — Frontend Test Cases


## Metadata
- Project: `alforqan`
- Feature: `teachers`
- Layer: `frontend`
- Routes: `/{locale}/{tenantSlug}/teachers`, `/{locale}/{tenantSlug}/teachers/{id}`, create/edit
- APIs: -
- Related code: `src/app/[locale]/(tenant)/[tenantSlug]/teachers/teachers-list-content.tsx`
- Related docs: `docs/technical/tenant/teachers-ui-overhaul.md`

## Scope

- Phase 6 emerald / BrandPageShell redesign for teachers list, profile, create/edit chrome.
- Does not replace deep registration/onboarding functional specs.

## Delivery waves

| Wave | Intent | Keys |
| ---- | ------ | ---- |
| 1 | List grid/filters/bulk | `ALFQ-TCH-UI-001`–`003` |
| 2 | Profile + forms chrome | `ALFQ-TCH-UI-004`–`006` |

## Test cases

### Wave 1 — List

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-01 | ALFQ-TCH-UI-001 | Verify teachers list grid/list toggle, search, and status filter | Main | User can open teachers list | 1. Open teachers list.<br>2. Toggle grid and list.<br>3. Search and change the status filter. | 1. Grid shows avatar cards; list shows a table.<br>2. Search and status filter update visible teachers without a crash. | - | `src/app/[locale]/(tenant)/[tenantSlug]/teachers/teachers-list-content.tsx`, `src/features/alforqan/components/teacher-card.tsx` | Locale en/ar |
| TC-02 | ALFQ-TCH-UI-002 | Verify empty teachers state and create CTA | Main | No matching teachers | 1. Filter or use a tenant with no teachers.<br>2. Read empty panel. | 1. Empty title/description shown.<br>2. Add Teacher CTA visible when create claim allows. | - | `src/app/[locale]/(tenant)/[tenantSlug]/teachers/teachers-list-content.tsx` | Empty list |
| TC-03 | ALFQ-TCH-UI-003 | Verify bulk select and deactivate toolbar | Main | User has manage-user claim; at least one active linked teacher | 1. Select one or more teachers (grid checkbox or list).<br>2. Use Deactivate selected and confirm. | 1. Bulk toolbar shows selection count.<br>2. Confirmed deactivate succeeds and list refreshes. | - | `src/app/[locale]/(tenant)/[tenantSlug]/teachers/teachers-list-content.tsx` | Active teachers with userId |

### Wave 2 — Profile and forms

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-04 | ALFQ-TCH-UI-004 | Verify teacher profile hero, sard stats, and led halaqat | Main | Teacher exists | 1. Open teacher detail.<br>2. Check hero, sard stat cards, halaqat they lead. | 1. Hero shows name/phone with emerald styling.<br>2. Four sard stat cards render.<br>3. Assigned halaqat link to the halaqa detail page (or empty copy shows). | - | `src/app/[locale]/(tenant)/[tenantSlug]/teachers/[id]/teacher-detail-content.tsx` | Existing teacher |
| TC-05 | ALFQ-TCH-UI-005 | Verify roles/permissions tab still works on redesigned profile | Main | Teacher has linked user; tester has roles/claims claims | 1. Open teacher detail.<br>2. Open Roles and permissions tab. | 1. Tab opens without a permission error.<br>2. Existing roles/claims UI is visible. | - | `src/app/[locale]/(tenant)/[tenantSlug]/teachers/[id]/teacher-detail-content.tsx` | Linked teacher user |
| TC-06 | ALFQ-TCH-UI-006 | Verify create/edit pages use BrandPageShell and success feedback | Main | Create/update teacher claim | 1. Open Add Teacher.<br>2. Complete create or open Edit and save. | 1. Pages use brand wash background.<br>2. Success toast appears and navigates to teacher detail. | - | `src/app/[locale]/(tenant)/[tenantSlug]/teachers/create/teacher-create-content.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/teachers/[id]/edit/teacher-edit-content.tsx` | Valid teacher data |
