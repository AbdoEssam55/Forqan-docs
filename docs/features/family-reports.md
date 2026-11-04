# Family reports (students and parents)

## Purpose

Mixed feed of **session** reports (hifz + revision with recorded outcomes) and **sard** results (passed/failed only) for students and linked parents.

## Endpoints

Base: `/API/V01/FamilyReports`

| Method | Route | Auth |
|--------|-------|------|
| GET | `children` | `ViewOwnReportsClaim` (student) or `ViewReportsClaim` (parent) |
| GET | `halaqat` | same |
| POST | `list` | same |

### Default role claims

- **Student:** `ViewOwnReportsClaim` only (not `ViewReportsClaim`).
- **Parent:** `ViewReportsClaim` only (not `ViewOwnReportsClaim`). Parents browse linked children's reports; they have no own student profile.
- **Dual Parent + Student:** keep `ViewOwnReportsClaim` on the user for student self-access.
- Startup seeding strips `ViewOwnReportsClaim` from the Parent role and parent-only users. Users must re-login after deploy so JWT matches persisted user claims.
- **Roles & Permissions UI:** `GetUserClaims` / `ModifyUserClaims` exclude `ViewOwnReportsClaim` and `ExportReportsClaim` for parent-only users (and staff `ViewReports*` plus `ExportReportsClaim` for student-only). See `TenantPoliciesHelper.GetExcludedClaimTypesForUserRoles`.

### GET children

Returns `{ id, fullName }[]`:

- **Student:** own profile only.
- **Parent:** linked children via `StudentParents`.

### GET halaqat

Query: `studentId?`

Returns `{ id, name }[]` of **Active** enrollments (`HalaqaStudents.Status == Active`, halaqa not deleted):

- **Student:** own halaqat.
- **Parent:** union of linked children's halaqat.
- `studentId` present: that child only; foreign id → 403.

### POST list

Request body (core):

- `studentId?` — omit for all allowed students; foreign id → 403.
- `dateFrom?`, `dateTo?` — calendar date filter on report date.
- `includeHifz`, `includeRevision`, `includeSard` (default true).
- `pageNumber`, `pageSize` (default 20, max 100). Response includes `totalCount` and `totalPages`.
- `search?` — matches student name, halaqa name, or sard scope (unused by family UI).

Advanced (Wave 4):

- `halaqaId` — session items for that halaqa only. **Sard items have no halaqa and are excluded** when `halaqaId` is set.
- `teacherUserId`, `surahId`, `juzId`, `scoreMin`, `scoreMax`, `sardPassed`, `attendanceStatus`, `sort` (`dateDesc` default).
- `scoreMin` / `scoreMax` compare `ScorePercentage` (0–100), not `TotalScore` points.

### Item kinds

- **session** — at least one evaluation with `OutcomeRecordedAt`; date = session `ScheduledAt`.
- **sard** — status `Passed` or `Failed`; date = `DecisionRecordedAt` ?? `ScheduledAt`.

## Scoping

Uses `FamilyReportAccessHelper` — same student/parent link rules as `GetSessionStudentReportQuery`.

## Related code

- `Alforqan.Api/Controllers/FamilyReports/FamilyReportsController.cs`
- `Alforqan.Core/Features/FamilyReports/Queries/FamilyReportQueries.cs`
