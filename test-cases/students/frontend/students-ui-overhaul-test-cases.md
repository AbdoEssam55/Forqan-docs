# Students UI overhaul — Frontend Test Cases


## Metadata
- Project: `alforqan`
- Feature: `students`
- Layer: `frontend`
- Routes: `/{locale}/{tenantSlug}/students`, `/{locale}/{tenantSlug}/students/{id}`, create/edit
- APIs: -
- Related code: `src/app/[locale]/(tenant)/[tenantSlug]/students/students-list-content.tsx`
- Related docs: `docs/technical/tenant/students-ui-overhaul.md`

## Scope

- Phase 5 emerald / BrandPageShell redesign for students list, profile, create/edit chrome.
- Does not replace deep onboarding/edit functional specs.

## Delivery waves

| Wave | Intent | Keys |
| ---- | ------ | ---- |
| 1 | List grid/filters/bulk | `ALFQ-STU-UI-001`–`003` |
| 2 | Profile + forms chrome | `ALFQ-STU-UI-004`–`006` |

## Test cases

### Wave 1 — List

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-01 | ALFQ-STU-UI-001 | Verify students list grid/list toggle, search, and filters | Main | User can open students list | 1. Open students list.<br>2. Toggle grid and list.<br>3. Search and change status/gender filters. | 1. Grid shows avatar cards; list shows a table.<br>2. Search and filters update visible students without a crash. | - | `src/app/[locale]/(tenant)/[tenantSlug]/students/students-list-content.tsx`, `src/features/alforqan/components/student-card.tsx` | Locale en/ar |
| TC-02 | ALFQ-STU-UI-002 | Verify empty students state and create CTA | Main | No matching students | 1. Filter or use a tenant with no students.<br>2. Read empty panel. | 1. Empty title/description shown.<br>2. Add Student CTA visible when create claim allows. | - | `src/app/[locale]/(tenant)/[tenantSlug]/students/students-list-content.tsx` | Empty list |
| TC-03 | ALFQ-STU-UI-003 | Verify bulk select and deactivate toolbar | Main | User has manage-user claim; at least one active linked student | 1. Select one or more students (grid checkbox or list).<br>2. Use Deactivate selected and confirm. | 1. Bulk toolbar shows selection count.<br>2. Confirmed deactivate succeeds and list refreshes. | - | `src/app/[locale]/(tenant)/[tenantSlug]/students/students-list-content.tsx` | Active students with userId |

### Wave 2 — Profile and forms

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-04 | ALFQ-STU-UI-004 | Verify student profile hero, progress bars, and reports link | Main | Student exists | 1. Open student detail.<br>2. Check hero, juz/sard progress, timeline/reports section. | 1. Hero shows name/code with emerald styling.<br>2. Progress cards render.<br>3. Open reports navigates to reports hub. | - | `src/app/[locale]/(tenant)/[tenantSlug]/students/[id]/student-detail-content.tsx` | Existing student |
| TC-05 | ALFQ-STU-UI-005 | Verify parent link dialog still works on redesigned profile | Main | Parents exist to link | 1. On student detail, open Enroll/Add parent.<br>2. Link a parent. | 1. Dialog opens and saves.<br>2. Linked parent appears in the list. | - | `src/app/[locale]/(tenant)/[tenantSlug]/students/[id]/student-detail-content.tsx` | Parent available |
| TC-06 | ALFQ-STU-UI-006 | Verify create/edit pages use BrandPageShell and success feedback | Main | Create/update student claim | 1. Open Add Student; optional upload profile picture preview.<br>2. Complete create or open Edit and save. | 1. Pages use brand wash background.<br>2. Success toast appears and navigates to student detail. | - | `src/app/[locale]/(tenant)/[tenantSlug]/students/create/student-create-content.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/students/[id]/edit/student-edit-content.tsx` | Valid student data |
