# Staff session reports (admin / teacher)

## Purpose

Tenant-wide **session evaluation** and **decided sard** reports for staff with `ViewMemorizationReportsClaim` (hifz + revision tabs) or `ViewNarrationReportsClaim` (sard tab). Parents and students use `FamilyReports` instead.

## Endpoints

Base: `/API/V01/StaffReports`

| Method | Route | Auth |
|--------|-------|------|
| POST | `list` | `type=hifz|revision` → `ViewMemorizationReportsClaim`; `type=sard` → `ViewNarrationReportsClaim` |
| POST | `summary` | same |
| POST | `export` | view claim for `type` **and** `ExportReportsClaim` (csv / pdf / xlsx, max 5000 rows) |

### POST list

Request body:

- `type` — `hifz`, `revision`, or `sard` (required)
- `dateFrom`, `dateTo` — session `ScheduledAt` or sard decision date. **Ignored** when `sessionId` (hifz/revision) or `sardSessionId` (sard) is set; the selected meeting is the period. Partial sessions are listed: each student with `OutcomeRecordedAt` is a row; you do not need every student evaluated.
- `pageNumber`, `pageSize` (max 100)
- `search` — student name, halaqa name, or sard scope
- `halaqaId`, `studentId`, `teacherUserId`
- `sessionId` — hifz/revision tabs only (`HalaqaSession` meeting)
- `sardSessionId` — sard tab only (`SardSession`)
- `surahId`, `juzId`
- `scoreMin`, `scoreMax` — percent 0–100
- `sardPassed` — sard tab only
- `attendanceStatus`, `sessionPassed` — session tabs only
- `sort` — `dateDesc` (default), `dateAsc`, `scoreDesc`, `scoreAsc`

**Scope**

- **Admin** role: all tenant rows matching filters.
- **Teacher** (non-admin with teacher profile): only rows for halaqat where `Halaqa.TeacherId` matches the teacher (session tabs). Sard: sessions where the teacher is on the committee.
- **Sard + halaqa filter:** keeps sard rows for students actively enrolled in that halaqa (unlike family list).

### Item fields (session tabs)

`type`, student, date, halaqa, **session title** (`sessionTitle`), **teacher** (halaqa assigned teacher via `Halaqa.TeacherId`), Quran range, attendance, score %, mistake counts (`memorizationMistakeCount`, `tajweedMistakeCount`, `tashkeelMistakeCount`, `totalMistakeCount`), session outcome, `sessionId`.

`teacherUserId` filter matches the halaqa teacher’s user id (same as Teachers list dropdown).

Counts exclude `MistakeType.Other`. Zero when the evaluation has no mistakes.

### POST export

Same filters as summary, plus:

- `format` — `csv` (default), `pdf`, `xlsx`
- `style` — `formal` (default) or `playful` (drawn medal badges on PDF; no emoji)
- `columns` — allowlist: `rank`, `date`, `student`, `halaqa`, `session`, `teacher`, `range`, `attendance`, `score`, `mistakes`, `outcome`. Empty → PDF defaults (rank, student, range, score, outcome) or sheet defaults (date, student, halaqa, range, score, outcome). Sard drops `attendance`.
- `sort` — same as list
- Locale from `Accept-Language` (Arabic unless the first tag is `en`)

PDF is an honor-board (tenant logo when `logoUrl` is http(s), KPIs, top-3 podium, every filtered row). Table rows are ordered by rank (highest score at the top). Arabic PDF uses RTL layout and **Cairo** (Regular body, Bold titles). Playful style draws colored rank badges (not emoji — Cairo has no medal glyphs). Excel is right-to-left with Cairo when installed. CSV uses localized headers, UTF-8 BOM, RTL embedding, and reversed column order.

Range cells use locale surah **names** (`الفاتحة: ١ — البقرة: ٥`), not `1:1 → 2:5`.

### Item fields (sard tab)

`type`, student, date, halaqa (enrollment), teacher, scope label, score %, mistake counts (same fields; from the sard evaluation), pass/fail, `sardSessionId`.

## Related code

- `Alforqan.Api/Controllers/StaffReports/StaffReportsController.cs`
- `Alforqan.Core/Features/StaffReports/Queries/StaffReportQueries.cs`
- `Alforqan.Core/Features/StaffReports/Helpers/StaffReportQueryHelper.cs`
- `Alforqan.Core/Features/StaffReports/Helpers/StaffReportMistakeCountHelper.cs`
- `Alforqan.Core/Features/StaffReports/Helpers/StaffReportCsvHelper.cs`
- `Alforqan.Core/Features/StaffReports/Helpers/StaffReportExportColumns.cs`
- `Alforqan.Core/Features/StaffReports/Helpers/StaffReportExportDocumentBuilder.cs`
- `Alforqan.Core/Features/StaffReports/Helpers/StaffReportRangeHelper.cs`
- `Alforqan.Service/Implementation/Reports/StaffReportExportFileGenerator.cs`
