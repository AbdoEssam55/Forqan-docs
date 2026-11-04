# From-end hifz progress — Frontend Test Cases

## Metadata
- Project: `alforqan`
- Feature: `students`
- Layer: `frontend`
- Routes: `/{locale}/{tenantSlug}/students/create`, `/{locale}/{tenantSlug}/students/{id}`, `/{locale}/{tenantSlug}/students/{id}/edit`
- APIs: `-`
- Related code: `src/lib/utils/hifz-progress.ts`
- Related docs: `docs/technical/tenant/hifz-direction-progress.md`

## Scope
- Student create/edit juz and surah pickers when hifz direction is from-end.
- Student profile juz progress bar for from-end students.

## Test pages
- `/{tenantSlug}/students/create`
- `/{tenantSlug}/students/{id}`
- `/{tenantSlug}/students/{id}/edit`

## Test cases

### From-end pickers and progress

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-01 | ALFQ-STU-HIFZ-001 | Verify from-end student at juz 30 is not shown as finished Mushaf | Main | Student hifz direction is "From the end"; current juz is 30 | 1. Open the student profile. | 1. The juz hint includes "1 of 30 memorized" / "1 من 30 محفوظاً".<br>2. The progress bar is near empty, not full. | - | `src/lib/utils/hifz-progress.ts`, `src/app/[locale]/(tenant)/[tenantSlug]/students/[id]/student-detail-content.tsx` | From-end; juz 30 |
| TC-02 | ALFQ-STU-HIFZ-002 | Verify from-end create form lists juz 30 first and An-Nas first in that juz | Main | User can create a student | 1. Open Add Student.<br>2. Set hifz direction to "From the end".<br>3. Open the juz dropdown.<br>4. Pick juz 30 and open the surah list. | 1. Juz options start at 30, not 1.<br>2. Surahs in juz 30 start at An-Nas (114), not An-Naba. | - | `src/lib/utils/hifz-progress.ts`, `src/features/alforqan/components/student-form.tsx` | From-end create |
| TC-03 | ALFQ-STU-HIFZ-003 | Verify from-end edit form keeps the same juz and surah order | Main | Existing from-end student | 1. Open student edit.<br>2. Open juz then surah for juz 30. | 1. Juz list starts at 30.<br>2. Surah list in juz 30 starts at An-Nas (114). | ALFQ-STU-HIFZ-002 | `src/lib/utils/hifz-progress.ts`, `src/features/alforqan/components/student-form.tsx` | From-end edit |
