# Students list columns and filters — Frontend Test Cases

## Metadata
- Project: `alforqan`
- Feature: `students`
- Layer: `frontend`
- Routes: `/{locale}/{tenantSlug}/students`
- APIs: `GET /API/V01/Students/List`
- Related code: `src/app/[locale]/(tenant)/[tenantSlug]/students/students-list-content.tsx`
- Related docs: `docs/technical/tenant/students-ui-overhaul.md`
- Automation: `-`

## Scope

- Students list table and grid show main halaqa teacher, age (from birth date), last surah, and last ayah.
- Filters (status, gender, stage, halaqa, teacher, age compare, Quran progress with hifz direction + surah + optional ayah compare) call the server so paging stays correct across the full dataset.

## Test pages

- `/{locale}/{tenantSlug}/students` (list + filter panel)

## Test scenarios

- List view columns and card fields match student profile progress fields.
- Server-backed filters change results and total page count, not only the current page slice.
- Compare operators and localized filter labels render in English and Arabic.

## Test cases

### Wave 1 — Columns

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-01 | ALFQ-STU-LST-001 | Verify list view shows teacher, age, last surah, and last ayah columns | Main | User can view students; at least one student has birth date, reached surah/ayah, and a first halaqa with a teacher | 1. Open the students list.<br>2. Switch to list (table) view.<br>3. Read the row for a known student. | 1. Columns include localized headers for teacher, age, last surah, and last ayah.<br>2. Teacher shows the first halaqa teacher name or "—".<br>3. Age shows completed years or "—".<br>4. Last surah shows a surah name (not a raw id).<br>5. Last ayah shows a number or "—". | - | `src/app/[locale]/(tenant)/[tenantSlug]/students/students-list-content.tsx` | Student with DOB + reached surah/ayah + halaqa |
| TC-02 | ALFQ-STU-LST-002 | Verify grid cards show teacher, age, last surah, and last ayah under the halaqa line | Main | Same as TC-01 | 1. Open the students list in grid view.<br>2. Read a card for a known student. | 1. Card shows halaqa name when enrolled.<br>2. Card shows teacher, age, last surah, and last ayah lines when data exists.<br>3. Labels match the list column translations (en/ar). | ALFQ-STU-LST-001 | `src/features/alforqan/components/student-card.tsx` | Same student as TC-01 |

### Wave 2 — Server filters

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-03 | ALFQ-STU-LST-003 | Verify teacher filter returns only students whose first halaqa uses that teacher | Main | Two students with different first-halaqa teachers; more than one page of students optional | 1. Open filters and pick a teacher from the teacher dropdown.<br>2. Note total count or last page.<br>3. Open page 2 if available.<br>4. Confirm every visible row/card matches the teacher. | 1. Network request includes `teacherId` query param.<br>2. No visible student shows a different first-halaqa teacher.<br>3. Page 2 still respects the filter (not only page 1). | ALFQ-STU-LST-001 | `src/app/[locale]/(tenant)/[tenantSlug]/students/students-list-content.tsx`, `src/lib/api/tenant-client.ts` | Distinct teachers |
| TC-04 | ALFQ-STU-LST-004 | Verify age "At least" filter uses server paging | Main | Students with known ages (e.g. one under 10, one 12+) | 1. Set age filter operator to "At least" / "أكبر أو يساوي".<br>2. Enter value `10`.<br>3. Read visible ages in list or cards. | 1. Request includes `ageOp=Gte` and `age=10`.<br>2. Every student with a shown age is 10 or older.<br>3. Students clearly under 10 are not listed. | ALFQ-STU-LST-001 | `src/app/[locale]/(tenant)/[tenantSlug]/students/students-list-content.tsx`, `src/lib/api/tenant-client.ts` | Mixed ages |
| TC-05 | ALFQ-STU-LST-005 | Verify Quran progress filter with direction, surah, and ayah compare | Main | Students with different reached surah/ayah; at least one from-end student past An-Nas | 1. Under "Quran progress", choose "From the beginning of the Mushaf".<br>2. Pick Al-Baqara; set operator "At least" with ayah `10`.<br>3. Note results; switch direction to "From the end of the Mushaf", surah An-Nas, ayah `1`, operator "At least". | 1. Step 2 request includes `surahId`, `progressHifzDirection=1`, `ayahOp=Gte`, `ayah=10`.<br>2. Step 2 lists students at or beyond Baqara 10 in start-to-end order (later surahs included).<br>3. Step 3 request includes `progressHifzDirection=2` and lists from-end students at or beyond An-Nas 1 (lower surah numbers included). | ALFQ-STU-LST-001 | `src/app/[locale]/(tenant)/[tenantSlug]/students/students-list-content.tsx`, `src/lib/api/tenant-client.ts` | Start vs end progress |
| TC-07 | ALFQ-STU-LST-007 | Verify surah-only progress filter does not send ayah compare | Main | Students on different reached surahs | 1. Under "Quran progress", pick a surah only (leave ayah operator on "All").<br>2. Inspect the list network request. | 1. Request includes `surahId` only (no `ayahOp` / `ayah`).<br>2. Only students whose reached surah matches that surah appear.<br>3. Students on other surahs are excluded even if ayah would match a progress compare. | ALFQ-STU-LST-001 | `src/app/[locale]/(tenant)/[tenantSlug]/students/students-list-content.tsx`, `src/lib/api/tenant-client.ts` | Distinct reached surahs |
| TC-06 | ALFQ-STU-LST-006 | Verify status and gender filters hit the API (not client-only) | Regression | Active and inactive students; male and female | 1. Set status to inactive only.<br>2. Reset; set gender to female.<br>3. Watch the list request query string. | 1. Inactive filter sends `isActive=false` and hides active students on all pages.<br>2. Gender filter sends `gender=2` and only female students appear.<br>3. Changing filter resets to page 1. | ALFQ-STU-LST-001 | `src/app/[locale]/(tenant)/[tenantSlug]/students/students-list-content.tsx` | Mixed status/gender |
