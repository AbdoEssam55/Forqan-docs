# Halaqa enrollment — Frontend Test Cases

## Metadata
- Project: `alforqan`
- Feature: `halaqat`
- Layer: `frontend`
- Routes: `/{locale}/{tenantSlug}/halaqat/{halaqaId}`
- APIs: `POST .../Halaqat/{halaqaId}/students`, `GET .../Halaqat/{halaqaId}/students`, `GET .../Students`
- Related code: `src/app/[locale]/(tenant)/[tenantSlug]/halaqat/[id]/halaqa-detail-content.tsx`, `src/components/ui/select.tsx`
- Related docs: `docs/technical/tenant/halaqat-guide.md`, `docs/explanation/tenant/halaqat-explained.md`

## Scope

- Enrolling a student into a halaqa from the halaqa detail page, including the "Enroll Student" dialog and student dropdown.

## Test pages

- `/{locale}/{tenantSlug}/halaqat/{halaqaId}`

## Test scenarios

- Open the enroll dialog from halaqa detail.
- Choose a student from the dropdown without the list being hidden behind the modal.
- Save enrollment and see the student on the halaqa.

## Delivery waves

| Wave | Intent | Keys (representative) | Depends on (cross-file) |
| ---- | ------ | --------------------- | ----------------------- |
| 1 | Enroll dialog and student pick | `ALFQ-HAL-ENR-001`–`003` | — |

## Test cases

### Wave 1 — Enroll student

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-01 | ALFQ-HAL-ENR-001 | Verify "Enroll Student" opens a dialog with a student selector | Main | User can manage halaqat; halaqa exists; at least one student is not yet enrolled in this halaqa | 1. Open the halaqa detail page.<br>2. Choose "Enroll Student". | 1. A dialog titled "Enroll Student" (or Arabic equivalent) opens.<br>2. A field labeled "Select a student" is visible with placeholder "Select a student". | - | `src/app/[locale]/(tenant)/[tenantSlug]/halaqat/[id]/halaqa-detail-content.tsx` | Halaqa with open seats; unenrolled student exists |
| TC-02 | ALFQ-HAL-ENR-002 | Verify the student dropdown list appears above the enroll modal and can be used | Main | Enroll dialog is open; at least one available student exists | 1. Open the "Select a student" dropdown.<br>2. Confirm the list of student names is fully visible and clickable on top of the modal (not behind it).<br>3. Choose one student (for example by name and code). | 1. The dropdown list renders in front of the modal, not behind the white dialog panel.<br>2. The chosen student name appears in the selector after picking.<br>3. List items remain clickable (pointer is not blocked by the modal). | ALFQ-HAL-ENR-001 | `src/components/ui/select.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/halaqat/[id]/halaqa-detail-content.tsx` | Student labels like "Menna Student begginer (STU-0002)" |
| TC-03 | ALFQ-HAL-ENR-003 | Verify saving enrollment adds the student to the halaqa | Main | A student is selected in the enroll dialog | 1. Choose "Save". | 1. A success toast appears with wording matching "enroll" success (English or Arabic).<br>2. The dialog closes.<br>3. The student appears in the students list on the halaqa detail page. | ALFQ-HAL-ENR-002 | `src/app/[locale]/(tenant)/[tenantSlug]/halaqat/[id]/halaqa-detail-content.tsx` | Same student chosen in TC-02 |
