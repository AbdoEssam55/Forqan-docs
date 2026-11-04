# Students list columns and filters — Backend Test Cases

## Metadata
- Project: `alforqan`
- Feature: `students`
- Layer: `backend`
- Routes: `-`
- APIs: `GET /API/V01/Students/List`
- Related code: `Alforqan/Alforqan.Service/Implementation/Students/Services/StudentService.cs`
- Related docs: `_See GET Students/List query params on StudentsController._`
- Automation: `Tests/Alforqan.Core.Tests/Features/Students/StudentPageFilterHelperTests.cs`

## Scope

- `GET /API/V01/Students/List` accepts paging, search, sort, and filter query params via `StudentsPageRequest`.
- Age compare and Quran progress (`progressHifzDirection` + `surahId` + `ayahOp`/`ayah`) filter before pagination; surah id alone matches exact `reachedSurahId`.
- First-halaqa halaqa and teacher filters use earliest enrollment by join date.
- Response `StudentDto` includes `firstHalaqaTeacherName` with `firstHalaqaName`.

## Test scenarios

- Query param contract and authorization unchanged (`ViewStudentsClaim`).
- Filter semantics match student profile fields (`DateOfBirth`, `ReachedSurahId`, `ReachedAyahNumber`).

## Test cases

### Wave 1 — API contract

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-01 | ALFQ-STU-LST-BE-001 | Verify GET list returns teacher name and reached surah name on items | Main | Tenant user with ViewStudents; student with first halaqa and reached surah | 1. Call `GET /API/V01/Students/List?pageNumber=1&pageSize=12`.<br>2. Find a known student in `data.items`. | 1. Response succeeds.<br>2. Item includes `firstHalaqaName` and `firstHalaqaTeacherName` when enrolled.<br>3. Item includes `reachedSurahName`, `reachedAyahNumber`, and `dateOfBirth`. | - | `Alforqan/Alforqan.Core/Features/Students/Shared/StudentFirstHalaqaEnricher.cs`, `Alforqan/Alforqan.Api/Controllers/Students/StudentsController.cs` | Seeded student |
| TC-02 | ALFQ-STU-LST-BE-002 | Verify gender and isActive query filters reduce the result set | Main | Mixed active/inactive and genders | 1. Call list with `isActive=false`.<br>2. Call list with `gender=1`. | 1. Every item in step 1 has `isActive` false.<br>2. Every item in step 2 has `gender` male (1). | ALFQ-STU-LST-BE-001 | `Alforqan/Alforqan.Service/Implementation/Students/Services/StudentService.cs` | Mixed students |
| TC-03 | ALFQ-STU-LST-BE-003 | Verify teacherId filter matches first halaqa only | Main | Student enrolled in two halaqat; earliest halaqa has teacher A, later halaqa teacher B | 1. Call list with `teacherId` of teacher B.<br>2. Call list with `teacherId` of teacher A. | 1. Student appears only when filtering by teacher A (first enrollment).<br>2. Filtering by teacher B does not return that student. | ALFQ-STU-LST-BE-001 | `Alforqan/Alforqan.Service/Implementation/Students/Services/StudentService.cs` | Two enrollments |
| TC-04 | ALFQ-STU-LST-BE-004 | Verify age Gte filter uses birth date before paging | Main | Students with known birth dates | 1. Call list with `ageOp=Gte&age=10`.<br>2. Compare `totalCount` to unfiltered list. | 1. All returned students have `dateOfBirth` on or before ten completed years ago (server rule).<br>2. `totalCount` is less than or equal to unfiltered count and excludes young students. | ALFQ-STU-LST-BE-001 | `Alforqan/Alforqan.Service/Implementation/Students/Services/StudentService.cs`, `Alforqan/Alforqan.Service/Implementation/Students/StudentPageFilterHelper.cs` | Ages 8 and 12 |
| TC-05 | ALFQ-STU-LST-BE-005 | Verify progress filter with progressHifzDirection, surahId, and ayah compare | Main | Students with reached surah/ayah at different mushaf positions | 1. Call `surahId={baqaraId}&progressHifzDirection=1&ayahOp=Gte&ayah=10`.<br>2. Call `surahId={nasId}&progressHifzDirection=2&ayahOp=Gte&ayah=2` for from-end progress past An-Nas.<br>3. Call `surahId={baqaraId}` without `ayahOp`. | 1. Step 1 returns students at or beyond Baqara 10 in start-to-end order (including later surahs; same surah needs ayah ≥ 10).<br>2. Step 2 uses from-end semantics (lower surah number = further; An-Nas ayah 1 alone does not satisfy Gte 2).<br>3. Step 3 matches exact `reachedSurahId` only.<br>4. xUnit `StudentPageFilterHelperTests` traits `ALFQ-STU-LST-BE-005` cover helper semantics. | ALFQ-STU-LST-BE-001 | `Alforqan/Alforqan.Service/Implementation/Students/StudentPageFilterHelper.cs`, `Alforqan/Alforqan.Service/Implementation/Students/Services/StudentService.cs`, `Tests/Alforqan.Core.Tests/Features/Students/StudentPageFilterHelperTests.cs` | Baqara + An-Nas |
