# Forms UI overhaul — Frontend Test Cases

## Metadata
- Project: `alforqan`
- Feature: `ui`
- Layer: `frontend`
- Routes: `/{locale}/{tenantSlug}/students/create`
- Related code: `src/lib/brand/form-field.ts`, `src/components/ui/form.tsx`, `src/components/ui/input.tsx`, `src/components/ui/file-upload.tsx`, `src/components/ui/radio-group.tsx`, `src/features/alforqan/components/student-create-wizard-form.tsx`
- Related docs: `docs/technical/tenant/forms-ui-overhaul.md`, `docs/technical/tenant/form-validation-ui.md`

## Scope

- Phase 10.1 form control chrome (inputs, selects, radios, switches, file upload) on student create.
- Phase 10.2 inline error / success / helper / loading feedback on the same form.

## Delivery waves

| Wave | Intent | Keys |
| ---- | ------ | ---- |
| 1 | Field chrome + radios | `ALFQ-FORM-UI-001`–`002` |
| 2 | File upload | `ALFQ-FORM-UI-003`–`004` |
| 3 | Existing pickers still branded | `ALFQ-FORM-UI-005` |
| 4 | Validation feedback | `ALFQ-FORM-UI-006`–`009` |

## Test cases

### Wave 1 — Field chrome + radios

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-01 | ALFQ-FORM-UI-001 | Verify student create text inputs use rounded double-border field chrome | Main | User can create students | 1. Open `/{tenantSlug}/students/create`.<br>2. Look at Username / Email / Password fields. | 1. Fields show a double border and rounded corners (not thin single border).<br>2. Focus shows a primary/emerald ring (not a thick black outline only). | - | `src/components/ui/input.tsx`, `src/lib/brand/form-field.ts`, `src/features/alforqan/components/student-create-wizard-form.tsx` | Create-student permission |
| TC-02 | ALFQ-FORM-UI-002 | Verify gender uses radio buttons Male and Female | Main | On student create step 1 | 1. Find the gender control.<br>2. Select "Male" / "ذكر".<br>3. Select "Female" / "أنثى". | 1. Gender is radio buttons (not a dropdown).<br>2. Only one option stays selected at a time.<br>3. Labels "Male"/"Female" (or Arabic equivalents) are visible. | ALFQ-FORM-UI-001 | `src/components/ui/radio-group.tsx`, `src/features/alforqan/components/student-create-wizard-form.tsx` | - |

### Wave 2 — File upload

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-03 | ALFQ-FORM-UI-003 | Verify profile picture uses FileUpload drop zone with preview | Main | On student create optional step | 1. Go to the optional profile step.<br>2. Choose a small JPG/PNG via the drop zone. | 1. Drop zone shows drag/browse copy (not a bare file input only).<br>2. A circular image preview appears.<br>3. File name and remove control are visible. | ALFQ-FORM-UI-001 | `src/components/ui/file-upload.tsx`, `src/features/alforqan/components/student-create-wizard-form.tsx` | Image under 5 MB |
| TC-04 | ALFQ-FORM-UI-004 | Verify FileUpload remove clears the selected picture | Main | Preview shown from TC-03 | 1. Click the remove control on the selected file. | 1. Preview disappears.<br>2. Drop zone returns to empty state. | ALFQ-FORM-UI-003 | `src/components/ui/file-upload.tsx`, `src/features/alforqan/components/student-create-wizard-form.tsx` | Same file |

### Wave 3 — Pickers

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-05 | ALFQ-FORM-UI-005 | Verify birth date picker trigger matches field chrome | Main | On student create optional step | 1. Open the birth date control.<br>2. Pick a past date. | 1. Trigger uses the same double-border rounded field style as text inputs.<br>2. Calendar popover opens and the chosen date shows on the trigger. | ALFQ-FORM-UI-001 | `src/components/ui/birth-date-picker.tsx`, `src/lib/brand/form-field.ts` | - |

### Wave 4 — Validation feedback

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-06 | ALFQ-FORM-UI-006 | Verify invalid email shows inline error with alert styling | Main | On student create step 1 | 1. Enter an invalid email (e.g. `not-an-email`).<br>2. Tab or blur the field. | 1. An inline error message appears under the field (not only a toast).<br>2. The message uses error (red) styling with an icon.<br>3. The email input border turns red / invalid. | ALFQ-FORM-UI-001 | `src/components/ui/form.tsx`, `src/components/ui/input.tsx`, `src/features/alforqan/components/student-create-wizard-form.tsx` | Invalid email |
| TC-07 | ALFQ-FORM-UI-007 | Verify valid email shows success hint | Main | On student create step 1 | 1. Enter a valid email and blur.<br>2. Read under the field. | 1. Success copy "Looks good" / "يبدو جيداً" appears with a check icon.<br>2. The email input border turns emerald (valid). | ALFQ-FORM-UI-006 | `src/components/ui/form.tsx`, `src/features/alforqan/components/student-create-wizard-form.tsx` | Valid email |
| TC-08 | ALFQ-FORM-UI-008 | Verify helper text uses muted helper styling | Main | On student create optional step | 1. Open the profile picture field.<br>2. Read the description under the upload. | 1. Helper text is visible under the control.<br>2. Helper text is muted (not the same weight/color as the error message). | ALFQ-FORM-UI-001 | `src/components/ui/form.tsx`, `src/features/alforqan/components/student-create-wizard-form.tsx` | - |
| TC-09 | ALFQ-FORM-UI-009 | Verify submit shows form loading status | Main | Valid required fields filled; on last step | 1. Click "Save" / "حفظ".<br>2. Watch the footer while the request is in flight. | 1. A "Saving…" / "جاري الحفظ…" status with spinner appears above the actions.<br>2. Action buttons stay disabled while saving. | ALFQ-FORM-UI-007 | `src/components/ui/form.tsx`, `src/features/alforqan/components/student-create-wizard-form.tsx` | Completable create payload |
