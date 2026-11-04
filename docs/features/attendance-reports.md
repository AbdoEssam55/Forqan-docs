# Attendance reports (admin / teacher)

## Purpose

Staff list of **saved session attendance marks** (Present / Absent / Late / Excused). Not evaluation scores. Unevaluated students still appear if they have an attendance row.

## Endpoints

Base: `/API/V01/AttendanceReports`

| Method | Route | Auth |
|--------|-------|------|
| POST | `list` | `ViewMemorizationReportsClaim` |
| POST | `summary` | same |
| POST | `export` | view claim **and** `ExportReportsClaim` (csv / pdf / xlsx, max 5000 rows) |

Module: Reports.

### POST list

- `dateFrom`, `dateTo` — session `ScheduledAt`. **Ignored** when `sessionId` is set.
- `pageNumber`, `pageSize` (max 100)
- `search` — student name, halaqa name, or session title
- `halaqaId`, `studentId`, `teacherUserId` (halaqa assigned teacher)
- `sessionId` — `HalaqaSession` meeting
- `attendanceStatus` — Present / Absent / Late / Excused
- `sort` — `dateDesc` (default) or `dateAsc`

**Scope**

- Admin: all tenant rows.
- Teacher: halaqat where `Halaqa.TeacherId` matches.

### POST summary

Same filters. Returns `totalCount`, `presentCount`, `absentCount`, `lateCount`, `excusedCount`.

### POST export

Same filters plus `format` (`csv` | `pdf` | `xlsx`) and `columns` (`date`, `student`, `halaqa`, `session`, `teacher`, `attendance`). No honor-board rank or medals.

## Related code

- `Alforqan.Api/Controllers/AttendanceReports/AttendanceReportsController.cs`
- `Alforqan.Core/Features/AttendanceReports/Queries/AttendanceReportQueries.cs`
- `Alforqan.Core/Features/AttendanceReports/Helpers/AttendanceReportQueryHelper.cs`
- `Alforqan.Core/Features/AttendanceReports/Helpers/AttendanceReportExportBuilder.cs`
