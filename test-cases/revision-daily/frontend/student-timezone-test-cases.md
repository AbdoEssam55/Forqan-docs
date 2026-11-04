# Student timezone — Frontend Test Cases

## Metadata

- Layer: frontend
- Related code: `src/features/alforqan/components/student-form.tsx`, `src/features/alforqan/components/student-create-wizard-form.tsx`, `src/features/alforqan/schemas/student-schema.ts`, `src/features/alforqan/student/student-timezone-options.ts`
- Related docs: `Alforqan-Backend/docs/technical-docs/student-timezone.md`
- Automation: -

## Scope

Student create/edit: required country of residence drives timezone. Nationality is identity only.

## Test cases

### Form

| ID | Key | Case | Type | Steps | Expected | Related code |
| --- | --- | --- | --- | --- | --- | --- |
| TC-01 | ALFQ-REVDAY-TZ-004 | Verify save blocked without country of residence | Main | 1. Open student create or edit.<br>2. Clear "Country of residence".<br>3. Try to continue or save. | 1. Validation shows "Country of residence is required." (or "بلد الإقامة مطلوب.").<br>2. Save does not proceed. | `src/features/alforqan/schemas/student-schema.ts`, `src/features/alforqan/schemas/student-create-wizard-schema.ts`, `src/features/alforqan/components/student-form.tsx`, `src/features/alforqan/components/student-create-wizard-form.tsx` |
| TC-02 | ALFQ-REVDAY-TZ-005 | Verify timezone follows residence not nationality | Main | 1. Open student create or edit.<br>2. Set nationality to Egypt.<br>3. Set country of residence to Afghanistan.<br>4. Save. | 1. Timezone field is hidden (single zone).<br>2. Saved timezone is Afghanistan (Kabul), not Egypt (Cairo). | `src/features/alforqan/student/student-timezone-options.ts`, `src/features/alforqan/components/student-form.tsx`, `src/features/alforqan/components/student-create-wizard-form.tsx` |
| TC-03 | ALFQ-REVDAY-TZ-006 | Verify timezone picker labels are translated | Main | 1. Open student edit in English.<br>2. Set country of residence to United States.<br>3. Open the timezone list. | 1. Options show "US Eastern", "US Central", "US Mountain", "US Pacific".<br>2. List does not show raw keys such as timezoneAmericaNewYork. | `src/features/alforqan/components/student-form.tsx`, `src/features/alforqan/student/student-timezone-options.ts` |
