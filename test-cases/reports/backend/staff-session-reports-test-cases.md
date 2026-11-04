# Staff session reports API — Backend Test Cases

## Metadata

- Project: `alforqan`
- Feature: `reports`
- Layer: `backend`
- Routes: `/API/V01/StaffReports/list`, `/API/V01/StaffReports/summary`, `/API/V01/StaffReports/export`, `/API/V01/StaffReports/filter-options`
- APIs: `POST /API/V01/StaffReports/list`, `POST /API/V01/StaffReports/summary`, `POST /API/V01/StaffReports/export`, `GET /API/V01/StaffReports/filter-options`
- Related code: `Alforqan/Alforqan.Api/Controllers/StaffReports/StaffReportsController.cs`, `Alforqan/Alforqan.Core/Features/StaffReports/Queries/StaffReportQueries.cs`, `Alforqan/Alforqan.Core/Features/StaffReports/Queries/StaffReportFilterOptionsQueries.cs`, `Alforqan/Alforqan.Core/Features/StaffReports/Helpers/StaffReportQueryHelper.cs`, `Alforqan/Alforqan.Core/Features/StaffReports/Helpers/StaffReportTeacherScopeHelper.cs`, `Alforqan/Alforqan.Core/Features/StaffReports/Helpers/StaffReportRequestScopeHelper.cs`, `Alforqan/Alforqan.Core/Features/StaffReports/Helpers/StaffReportTypeHelper.cs`, `Alforqan/Alforqan.Core/Features/StaffReports/Helpers/StaffReportAccessHelper.cs`, `Alforqan/Alforqan.Core/Features/StaffReports/Helpers/StaffReportMistakeCountHelper.cs`, `Alforqan/Alforqan.Core/Features/StaffReports/Helpers/StaffReportCsvHelper.cs`, `Alforqan/Alforqan.Core/Features/StaffReports/Helpers/StaffReportExportColumns.cs`, `Alforqan/Alforqan.Core/Features/StaffReports/Helpers/StaffReportExportDocumentBuilder.cs`, `Alforqan/Alforqan.Core/Features/StaffReports/Helpers/StaffReportRangeHelper.cs`, `Alforqan/Alforqan.Core/Features/StaffReports/Helpers/StaffReportExportLabels.cs`, `Alforqan/Alforqan.Service/Implementation/Reports/StaffReportExportFileGenerator.cs`
- Related docs: `docs/features/staff-session-reports.md`
- Automation: `Tests/Alforqan.Core.Tests/Features/StaffReports/StaffReportTypeHelperTests.cs`, `Tests/Alforqan.Core.Tests/Features/StaffReports/StaffReportMistakeCountHelperTests.cs`, `Tests/Alforqan.Core.Tests/Features/StaffReports/StaffReportDateRangeFilterTests.cs`, `Tests/Alforqan.Core.Tests/Features/StaffReports/StaffReportExportLabelsTests.cs`, `Tests/Alforqan.Core.Tests/Features/StaffReports/StaffReportTeacherScopeHelperTests.cs`

## Scope

- Staff-only session evaluation list (hifz, revision) and decided sard rows.
- SQL paging (`Skip`/`Take`), not in-memory page.
- Teacher scope via halaqa `TeacherId`; admin tenant-wide.
- Sard + `halaqaId` keeps sard via active student enrollment (does not drop sard like family list).
- `sessionId` / `sardSessionId` skip `dateFrom`/`dateTo`; partial evaluated students still list.

## Delivery waves

| Wave | Intent | Keys |
| ---- | ------ | ---- |
| 1 | Hifz list API | `ALFQ-SSR-API-001`–`006` |
| 2 | Revision, sard, summary | `ALFQ-SSR-API-010`–`018` |
| 3 | Export | `ALFQ-SSR-API-020` |
| 4 | Mistake counts | `ALFQ-SSR-API-021`–`022` |
| 5 | Honor-board export formats | `ALFQ-SSR-API-023`–`028` |
| 6 | Late minutes on export attendance | `ALFQ-SSR-API-029` |
| 7 | Teacher-scoped filter options and sard rows | `ALFQ-SSR-API-030`–`032` |

## Test cases

### Wave 1 — Hifz list API

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-01 | ALFQ-SSR-API-001 | Verify parent or student JWT receives 403 on StaffReports list | Main | Parent or student JWT without staff report claims | 1. `POST /StaffReports/list` with `type` `hifz`. | 1. HTTP 403.<br>2. No report items returned. | - | `Alforqan/Alforqan.Api/Controllers/StaffReports/StaffReportsController.cs` | Parent or student account |
| TC-02 | ALFQ-SSR-API-002 | Verify teacher list returns only rows for own halaqat | Main | Teacher JWT with `ViewMemorizationReportsClaim`; teacher leads halaqa A; student reports exist in halaqa A and B | 1. `POST /StaffReports/list` with `type` `hifz` and default paging. | 1. Response succeeds.<br>2. Every row's halaqa is one the teacher leads.<br>3. No rows from other teachers' halaqat. | - | `Alforqan/Alforqan.Core/Features/StaffReports/Helpers/StaffReportAccessHelper.cs`, `Alforqan/Alforqan.Core/Features/StaffReports/Helpers/StaffReportQueryHelper.cs` | Teacher + two halaqat |
| TC-03 | ALFQ-SSR-API-003 | Verify invalid report type is rejected | Edge | Admin JWT with memorization claim | 1. `POST /StaffReports/list` with `type` `mixed`. | 1. HTTP 400 or failed response with invalid type message.<br>2. No items returned. | - | `Alforqan/Alforqan.Core/Features/StaffReports/Helpers/StaffReportTypeHelper.cs` | Admin JWT |
| TC-04 | ALFQ-SSR-API-004 | Verify date range and score percent filters apply on hifz list | Main | Admin with hifz outcomes on known dates and scores | 1. `POST /StaffReports/list` with `type` `hifz`, narrow `dateFrom`/`dateTo`, `scoreMin` 80. | 1. Every item date is inside the range.<br>2. Every item `scorePercentage` is at least 80. | ALFQ-SSR-API-001 | `Alforqan/Alforqan.Core/Features/StaffReports/Helpers/StaffReportQueryHelper.cs` | Known dates and scores |
| TC-05 | ALFQ-SSR-API-005 | Verify list paging uses SQL page size cap 100 | Main | Admin with more than 10 hifz rows in range | 1. `POST /StaffReports/list` with `pageNumber` 1, `pageSize` 10.<br>2. Repeat with `pageNumber` 2. | 1. First response `items` length is at most 10.<br>2. `totalCount` exceeds 10.<br>3. Second page ids do not duplicate the first page. | ALFQ-SSR-API-001 | `Alforqan/Alforqan.Core/Features/StaffReports/Queries/StaffReportQueries.cs` | 11+ hifz rows |
| TC-06 | ALFQ-SSR-API-006 | Verify sessionId ignores date range and returns each recorded student | Main | Session scheduled last month; two students with `OutcomeRecordedAt`; other students in the same session not evaluated; this-month date range would exclude the session | 1. `POST /StaffReports/list` with `type` `hifz`, `sessionId` for that session, and `dateFrom`/`dateTo` for this month. | 1. Response succeeds.<br>2. Both evaluated students are in `items`.<br>3. Unevaluated students are absent.<br>4. Rows are not dropped because the session date is outside the month range. | ALFQ-SSR-API-004 | `Alforqan/Alforqan.Core/Features/StaffReports/Helpers/StaffReportQueryHelper.cs` | Partial session outside period |

### Wave 2 — Revision, sard, summary

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-10 | ALFQ-SSR-API-010 | Verify revision type returns revision evaluations not hifz | Main | Admin with both hifz and revision outcomes for same student | 1. `POST /StaffReports/list` with `type` `revision`. | 1. Response succeeds.<br>2. Every item `type` is `revision`.<br>3. No memorization-only session rows appear. | ALFQ-SSR-API-001 | `Alforqan/Alforqan.Core/Features/StaffReports/Queries/StaffReportQueries.cs` | Hifz + revision data |
| TC-11 | ALFQ-SSR-API-011 | Verify sard type requires ViewNarrationReportsClaim | Main | Staff JWT with memorization claim only (no `ViewNarrationReportsClaim`) | 1. `POST /StaffReports/list` with `type` `sard`. | 1. HTTP 403.<br>2. No sard items returned. | - | `Alforqan/Alforqan.Api/Controllers/StaffReports/StaffReportsController.cs`, `Alforqan/Alforqan.Core/Features/StaffReports/Helpers/StaffReportTypeHelper.cs` | Memorization-only staff |
| TC-12 | ALFQ-SSR-API-012 | Verify sard list includes only passed or failed sard sessions | Main | Admin with decided and pending sard | 1. `POST /StaffReports/list` with `type` `sard`. | 1. Response succeeds.<br>2. Every item has `sardPassed` true or false.<br>3. Pending sard sessions are absent. | ALFQ-SSR-API-011 | `Alforqan/Alforqan.Core/Features/StaffReports/Helpers/StaffReportQueryHelper.cs` | Pass + pending sard |
| TC-13 | ALFQ-SSR-API-013 | Verify sard list with halaqaId keeps sard via active enrollment | Main | Student actively enrolled in halaqa X with decided sard | 1. `POST /StaffReports/list` with `type` `sard` and `halaqaId` for halaqa X. | 1. Sard rows for that enrolled student remain in the response.<br>2. Sard is not dropped unlike family list with halaqa filter. | ALFQ-SSR-API-012 | `Alforqan/Alforqan.Core/Features/StaffReports/Helpers/StaffReportQueryHelper.cs` | Enrolled student + sard |
| TC-14 | ALFQ-SSR-API-014 | Verify juzId filter applies to session and sard ranges | Main | Admin with reports overlapping and outside a chosen juz | 1. `POST /StaffReports/list` with `type` `revision` and `juzId` set.<br>2. Repeat with `type` `sard` and same `juzId`. | 1. Revision items' surah ranges overlap the juz surah span.<br>2. Sard items' scope surahs overlap the same juz. | ALFQ-SSR-API-010 | `Alforqan/Alforqan.Core/Features/StaffReports/Helpers/StaffReportQueryHelper.cs` | Juz + range data |
| TC-15 | ALFQ-SSR-API-015 | Verify attendanceStatus and sessionPassed filters on session tabs | Main | Admin with mixed attendance and pass/fail sessions | 1. `POST /StaffReports/list` with `type` `hifz`, `attendanceStatus` Absent.<br>2. Repeat with `sessionPassed` false. | 1. First response: every row attendance is Absent.<br>2. Second response: every row session outcome is Failed. | ALFQ-SSR-API-001 | `Alforqan/Alforqan.Core/Features/StaffReports/Helpers/StaffReportQueryHelper.cs` | Mixed attendance |
| TC-16 | ALFQ-SSR-API-016 | Verify teacherUserId filters session evaluator and sard committee | Main | Admin; known evaluator on session and committee teacher on sard | 1. `POST /StaffReports/list` with `type` `hifz` and `teacherUserId`.<br>2. `POST /StaffReports/list` with `type` `sard` and same `teacherUserId`. | 1. Hifz rows match `EvaluatedByUserId`.<br>2. Sard rows include sessions where that user is on the committee. | ALFQ-SSR-API-012 | `Alforqan/Alforqan.Core/Features/StaffReports/Helpers/StaffReportQueryHelper.cs` | Known teacher user ids |
| TC-17 | ALFQ-SSR-API-017 | Verify summary returns aggregates for current filters and type | Main | Admin with filtered hifz data | 1. `POST /StaffReports/summary` with same body as list (no paging). | 1. Response succeeds.<br>2. `totalCount` matches list `totalCount` for the same filters.<br>3. `averageScorePercent`, `passedCount`, and `absentCount` are present for session types. | ALFQ-SSR-API-004 | `Alforqan/Alforqan.Core/Features/StaffReports/Queries/StaffReportQueries.cs` | Filtered hifz set |
| TC-18 | ALFQ-SSR-API-018 | Verify sard summary uses passed and failed counts | Main | Admin with pass and fail sard in range | 1. `POST /StaffReports/summary` with `type` `sard`. | 1. `passedCount` equals pass sard count in range.<br>2. `failedCount` equals fail sard count.<br>3. `absentCount` is zero or not used for sard tab. | ALFQ-SSR-API-012 | `Alforqan/Alforqan.Core/Features/StaffReports/Queries/StaffReportQueries.cs` | Pass + fail sard |

### Wave 3 — Export

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-20 | ALFQ-SSR-API-020 | Verify export returns a file capped at 5000 rows and requires ExportReportsClaim | Main | Admin with memorization claim and `ExportReportsClaim`; parent JWT; staff JWT without export claim | 1. `POST /StaffReports/export` with `type` `hifz` as admin.<br>2. Repeat as parent JWT.<br>3. Repeat as staff JWT that can view hifz but lacks `ExportReportsClaim`. | 1. Admin receives a file download with a header row.<br>2. Row count is at most 5000.<br>3. Parent receives HTTP 403.<br>4. Staff without `ExportReportsClaim` receives HTTP 403. | ALFQ-SSR-API-001 | `Alforqan/Alforqan.Api/Controllers/StaffReports/StaffReportsController.cs` | Admin + parent JWT |

### Wave 4 — Mistake counts

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-21 | ALFQ-SSR-API-021 | Verify list items include mistake counts by type excluding Other | Main | Admin with a hifz evaluation that has حفظ, تجويد, تشكيل, and Other mistakes | 1. `POST /StaffReports/list` with `type` `hifz` for that student/date. | 1. Item has `memorizationMistakeCount`, `tajweedMistakeCount`, and `tashkeelMistakeCount`.<br>2. Counts match the evaluation mistake rows of those three types.<br>3. `MistakeType.Other` rows are not included in any count.<br>4. `totalMistakeCount` equals the sum of the three type counts. | ALFQ-SSR-API-001 | `Alforqan/Alforqan.Core/Features/StaffReports/Helpers/StaffReportMistakeCountHelper.cs`, `Alforqan/Alforqan.Core/Features/StaffReports/Queries/StaffReportQueries.cs` | Evaluation with mixed types |
| TC-22 | ALFQ-SSR-API-022 | Verify export CSV includes a mistakes column when selected | Main | Admin with memorization claim, `ExportReportsClaim`, and at least one hifz row | 1. `POST /StaffReports/export` with `type` `hifz`, `format` `csv`, `columns` including `mistakes`, `Accept-Language: en`. | 1. CSV header includes "Mistakes".<br>2. Each data row has a mistakes cell with the total and Hifz/Tajweed/Tashkeel split. | ALFQ-SSR-API-020 | `Alforqan/Alforqan.Core/Features/StaffReports/Helpers/StaffReportCsvHelper.cs`, `Alforqan/Alforqan.Core/Features/StaffReports/Helpers/StaffReportExportDocumentBuilder.cs` | Exportable hifz rows |

### Wave 5 — Honor-board export formats

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-23 | ALFQ-SSR-API-023 | Verify export column allowlist drops unknown ids and sard attendance | Main | - | 1. Call column sanitize with type `sard` and columns `student, attendance, nope, score, student`. | 1. Result is `student` then `score` only.<br>2. `attendance` is absent. | - | `Alforqan/Alforqan.Core/Features/StaffReports/Helpers/StaffReportExportColumns.cs` | - |
| TC-24 | ALFQ-SSR-API-024 | Verify pdf format returns a PDF byte stream | Main | Admin with `ExportReportsClaim` and at least one hifz row | 1. `POST /StaffReports/export` with `type` `hifz` and `format` `pdf`. | 1. HTTP 200 file download.<br>2. Body starts with `%PDF`.<br>3. Content-Type is `application/pdf`.<br>4. Embedded Arabic font is Cairo, not Noto Naskh.<br>5. File does not use emoji medals (🥇⭐). | ALFQ-SSR-API-020 | `Alforqan/Alforqan.Service/Implementation/Reports/StaffReportExportFileGenerator.cs` | Exportable hifz rows |
| TC-25 | ALFQ-SSR-API-025 | Verify xlsx format returns an Excel package | Main | Admin with `ExportReportsClaim` and at least one hifz row | 1. `POST /StaffReports/export` with `type` `hifz` and `format` `xlsx`. | 1. HTTP 200 file download.<br>2. Body is a zip/xlsx package (starts with `PK`). | ALFQ-SSR-API-020 | `Alforqan/Alforqan.Service/Implementation/Reports/StaffReportExportFileGenerator.cs` | Exportable hifz rows |
| TC-26 | ALFQ-SSR-API-026 | Verify Arabic CSV headers, Indic digits, and RTL embedding | Main | Admin with `ExportReportsClaim` | 1. `POST /StaffReports/export` with `format` `csv`, `columns` `student,score`, `Accept-Language: ar`. | 1. Header includes "الطالب" and "الدرجة %".<br>2. Score uses Arabic-Indic digits when numeric.<br>3. Cells are RTL (Unicode RLE marks; columns ordered right-to-left). | ALFQ-SSR-API-022 | `Alforqan/Alforqan.Core/Features/StaffReports/Helpers/StaffReportExportLabels.cs`, `Alforqan/Alforqan.Core/Features/StaffReports/Helpers/StaffReportCsvHelper.cs` | Hifz row with score |
| TC-27 | ALFQ-SSR-API-027 | Verify rank by score puts the highest score first | Main | Two hifz rows with scores 40 and 99 | 1. Build ranks for those two rows.<br>2. Build the export document with columns rank, student, score. | 1. Score 99 is rank 1.<br>2. Score 40 is rank 2.<br>3. First table row is rank 1 (99%).<br>4. Second table row is rank 2 (40%).<br>5. Rank and outcome cells have no emoji. | - | `Alforqan/Alforqan.Core/Features/StaffReports/Helpers/StaffReportExportDocumentBuilder.cs` | Two scored rows |
| TC-28 | ALFQ-SSR-API-028 | Verify export range cells use surah names not ids | Main | Hifz row from Al-Fatiha ayah 1 to Al-Baqarah ayah 5 | 1. Export CSV/PDF with `columns` including `range` and `Accept-Language: ar`. | 1. Range cell includes "الفاتحة" and "البقرة".<br>2. Range cell does not use `1:1` / `2:5` id form. | ALFQ-SSR-API-026 | `Alforqan/Alforqan.Core/Features/StaffReports/Helpers/StaffReportRangeHelper.cs`, `Alforqan/Alforqan.Core/Features/StaffReports/Helpers/StaffReportExportDocumentBuilder.cs` | Hifz row with surah ids |

### Wave 6 — Late minutes

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-29 | ALFQ-SSR-API-029 | Verify export attendance label includes late minutes | Main | - | 1. Call `StaffReportExportLabels.AttendanceLabel` with Late and 15.<br>2. Call it with Late and no minutes.<br>3. Call it with Present and 15. | 1. Late with 15 is "Late (15)".<br>2. Late with no minutes is "Late".<br>3. Present still is "Present". | - | `Alforqan/Alforqan.Core/Features/StaffReports/Helpers/StaffReportExportLabels.cs` | 15 minutes |

### Wave 7 — Teacher scope

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-30 | ALFQ-SSR-API-030 | Verify filter-options returns scoped students for teacher | Main | Teacher JWT with report claims; student A in teacher halaqa; student B only in another teacher halaqa | 1. `GET /StaffReports/filter-options`. | 1. Response succeeds.<br>2. `isTenantWide` is false.<br>3. Student A appears in `students`.<br>4. Student B is absent from `students`. | ALFQ-SSR-API-002 | `Alforqan/Alforqan.Core/Features/StaffReports/Queries/StaffReportFilterOptionsQueries.cs`, `Alforqan/Alforqan.Core/Features/StaffReports/Helpers/StaffReportTeacherScopeHelper.cs` | Two teachers + two students |
| TC-31 | ALFQ-SSR-API-031 | Verify filter-options for admin is tenant-wide with empty scoped lists | Main | Admin JWT with report claims | 1. `GET /StaffReports/filter-options`. | 1. Response succeeds.<br>2. `isTenantWide` is true.<br>3. `students` and `sardSessions` arrays are empty (admin uses global list APIs). | ALFQ-SSR-API-001 | `Alforqan/Alforqan.Api/Controllers/StaffReports/StaffReportsController.cs`, `Alforqan/Alforqan.Core/Features/StaffReports/Queries/StaffReportFilterOptionsQueries.cs` | Admin JWT |
| TC-32 | ALFQ-SSR-API-032 | Verify teacher sard list includes decided sard for own halaqa students | Main | Teacher leads halaqa X; student in X has passed sard; teacher is not on sard committee | 1. `POST /StaffReports/list` with `type` `sard`. | 1. Response succeeds.<br>2. Sard row for that student is present.<br>3. Sard rows for students outside the teacher halaqat are absent. | ALFQ-SSR-API-012 | `Alforqan/Alforqan.Core/Features/StaffReports/Helpers/StaffReportQueryHelper.cs`, `Alforqan/Alforqan.Core/Features/StaffReports/Helpers/StaffReportTeacherScopeHelper.cs` | Pass sard without committee membership |
