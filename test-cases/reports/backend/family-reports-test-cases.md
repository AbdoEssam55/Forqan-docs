# Family reports API — Backend Test Cases

## Metadata

- Project: `alforqan`
- Feature: `reports`
- Layer: `backend`
- Routes: `/API/V01/FamilyReports/children`, `/API/V01/FamilyReports/halaqat`, `/API/V01/FamilyReports/list`
- APIs: `GET /API/V01/FamilyReports/children`, `GET /API/V01/FamilyReports/halaqat`, `POST /API/V01/FamilyReports/list`
- Related code: `Alforqan/Alforqan.Api/Controllers/FamilyReports/FamilyReportsController.cs`, `Alforqan/Alforqan.Core/Features/FamilyReports/Queries/FamilyReportQueries.cs`, `Alforqan/Alforqan.Core/Features/FamilyReports/Helpers/FamilyReportAccessHelper.cs`, `Alforqan/Alforqan.Core/Features/FamilyReports/Helpers/FamilyReportHalaqaFilter.cs`
- Related docs: `docs/features/family-reports.md`
- Automation: `Tests/Alforqan.Core.Tests/Features/FamilyReports/FamilyReportHalaqaFilterTests.cs`, `Tests/Alforqan.Core.Tests/Features/FamilyReports/ParentReportDefaultClaimsTests.cs`

## Scope

- Family-scoped children picker and mixed session/sard list.
- Student self-only; parent linked children only.
- Outcome-recorded sessions and decided sard only.
- Role-scoped active halaqat for the family dropdown; `halaqaId` on list drops sard.

## Delivery waves

| Wave | Intent | Keys |
| ---- | ------ | ---- |
| 1d | Parent vs student report claims | `ALFQ-FAM-API-017`–`018` |
| 2 | List and children API | `ALFQ-FAM-API-001`–`008` |
| 4 | Advanced filters | `ALFQ-FAM-API-009`–`012` |
| 4b | List paging | `ALFQ-FAM-API-013` |
| 4c | Halaqa filter | `ALFQ-FAM-API-014`–`016` |

## Test cases

### Wave 1d — Parent vs student report claims

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-17 | ALFQ-FAM-API-017 | Verify Parent role default claims include ViewReportsClaim and exclude ViewOwnReportsClaim | Main | Tenant API restarted after deploy; parent-only account | 1. Restart the tenant API so startup seeding runs.<br>2. Sign in as a parent-only user (not also Student) and decode the JWT permission claims, or open Roles & Permissions for the Parent role in admin UI.<br>3. Confirm report-related claims. | 1. Parent role defaults (and parent-only user JWT after re-login) include `ViewReportsClaim`.<br>2. Parent role defaults and parent-only user JWT do not include `ViewOwnReportsClaim`. | - | `Alforqan/Alforqan.Shared/HelperClasses/TenantPoliciesHelper.cs`, `Platform/Platform.Infrastructure/Seeding/RoleSeeding.cs` | Parent-only account |
| TC-18 | ALFQ-FAM-API-018 | Verify Student role default claims include ViewOwnReportsClaim and exclude ViewReportsClaim | Main | Tenant API running; student account | 1. Open Roles & Permissions for the Student role in admin UI, or sign in as a student and decode JWT report claims.<br>2. Confirm report-related claims. | 1. Student role defaults include `ViewOwnReportsClaim`.<br>2. Student role defaults do not include `ViewReportsClaim`. | - | `Alforqan/Alforqan.Shared/HelperClasses/TenantPoliciesHelper.cs` | Student account |

### Wave 2 — List and children API

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-01 | ALFQ-FAM-API-001 | Verify student children endpoint returns only the signed-in student | Main | Student JWT with `ViewOwnReportsClaim` | 1. `GET /FamilyReports/children`. | 1. Response succeeds.<br>2. Exactly one child row matches the student profile id and name. | - | `Alforqan/Alforqan.Core/Features/FamilyReports/Queries/FamilyReportQueries.cs` | Student account |
| TC-02 | ALFQ-FAM-API-002 | Verify parent children endpoint returns linked children only | Main | Parent JWT linked to at least one student | 1. `GET /FamilyReports/children`. | 1. Response succeeds.<br>2. Every returned id is a linked child.<br>3. Unlinked students are not listed. | - | `Alforqan/Alforqan.Core/Features/FamilyReports/Helpers/FamilyReportAccessHelper.cs` | Parent + linked child |
| TC-03 | ALFQ-FAM-API-003 | Verify student list returns only own session and sard items | Main | Student with at least one outcome-recorded session and one decided sard | 1. `POST /FamilyReports/list` with default flags. | 1. Response succeeds.<br>2. Every item has `studentId` equal to the student id.<br>3. At least one `kind` is `session` or `sard` when data exists. | ALFQ-FAM-API-001 | `Alforqan/Alforqan.Core/Features/FamilyReports/Queries/FamilyReportQueries.cs` | Seeded session + sard |
| TC-04 | ALFQ-FAM-API-004 | Verify parent list with omitted studentId includes all linked children | Main | Parent with two linked children who both have reports | 1. `POST /FamilyReports/list`. | 1. Response includes items for more than one `studentId` when both children have data.<br>2. No foreign student ids appear. | ALFQ-FAM-API-002 | `Alforqan/Alforqan.Core/Features/FamilyReports/Queries/FamilyReportQueries.cs` | Two linked children |
| TC-05 | ALFQ-FAM-API-005 | Verify foreign studentId returns forbidden | Edge | Parent JWT | 1. `POST /FamilyReports/list` with `studentId` of an unlinked student. | 1. HTTP 403.<br>2. No report items for that student. | ALFQ-FAM-API-002 | `Alforqan/Alforqan.Api/Controllers/FamilyReports/FamilyReportsController.cs` | Unlinked student id |
| TC-06 | ALFQ-FAM-API-006 | Verify undecided sard sessions are excluded | Main | Student with a sard session not in Passed/Failed status | 1. `POST /FamilyReports/list` with `includeSard` true. | 1. No list item has `kind` `sard` for the undecided session id. | ALFQ-FAM-API-003 | `Alforqan/Alforqan.Core/Features/FamilyReports/Queries/FamilyReportQueries.cs` | Pending sard session |
| TC-07 | ALFQ-FAM-API-007 | Verify session without recorded outcome is excluded | Main | Student with submitted but not outcome-recorded evaluation | 1. `POST /FamilyReports/list` with hifz/revision included. | 1. No session item exists for that session until outcome is recorded. | ALFQ-FAM-API-003 | `Alforqan/Alforqan.Core/Features/FamilyReports/Queries/FamilyReportQueries.cs` | Draft evaluation |
| TC-08 | ALFQ-FAM-API-008 | Verify type flags filter hifz revision and sard rows | Main | Student with session (hifz and/or revision with outcome) and at least one decided sard | 1. `POST /FamilyReports/list` with `includeSard` false (hifz and revision true).<br>2. `POST /FamilyReports/list` with `includeHifz` false, `includeRevision` false, `includeSard` true. | 1. First response has no items with `kind` `sard`.<br>2. Second response has no items with `kind` `session`. | ALFQ-FAM-API-003 | `Alforqan/Alforqan.Core/Features/FamilyReports/Queries/FamilyReportQueries.cs` | Mixed data |

### Wave 4 — Advanced filters

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-09 | ALFQ-FAM-API-009 | Verify date range filters exclude out-of-range items | Main | Student with reports on known dates | 1. `POST /FamilyReports/list` with narrow `dateFrom`/`dateTo`. | 1. Every item date falls inside the range. | ALFQ-FAM-API-003 | `Alforqan/Alforqan.Core/Features/FamilyReports/Queries/FamilyReportQueries.cs` | Known report dates |
| TC-10 | ALFQ-FAM-API-010 | Verify scoreMin and scoreMax filter session and sard scores | Main | Student with mixed scores | 1. `POST /FamilyReports/list` with `scoreMin` 80. | 1. `80` is compared against `ScorePercentage` (0–100), not `TotalScore` points.<br>2. Items without scores or with percent below 80 are excluded.<br>3. Items with percent 80 or above remain even when raw points/`TotalScore` would be below 80. | ALFQ-FAM-API-003 | `Alforqan/Alforqan.Core/Features/FamilyReports/Queries/FamilyReportQueries.cs` | Score data |
| TC-11 | ALFQ-FAM-API-011 | Verify sardPassed filter returns only pass or fail sard rows | Main | Student with at least one passed and one failed sard in range | 1. `POST /FamilyReports/list` with `includeSard` true and `sardPassed` true.<br>2. Repeat with `sardPassed` false. | 1. First response: every `kind` `sard` item has `sardPassed` true.<br>2. Second response: every `kind` `sard` item has `sardPassed` false.<br>3. Session items are unchanged by `sardPassed`. | ALFQ-FAM-API-003 | `Alforqan/Alforqan.Core/Features/FamilyReports/Queries/FamilyReportQueries.cs` | Pass + fail sard |
| TC-12 | ALFQ-FAM-API-012 | Verify sort scoreDesc orders by highest score first | Main | Student with multiple scored items | 1. `POST /FamilyReports/list` with `sort` `scoreDesc`. | 1. First item score is greater than or equal to the next item score when both have scores. | ALFQ-FAM-API-003 | `Alforqan/Alforqan.Core/Features/FamilyReports/Queries/FamilyReportQueries.cs` | Multiple scores |
| TC-13 | ALFQ-FAM-API-013 | Verify list paging returns one page and the full totalCount | Main | Student with more than 10 matching reports | 1. `POST /FamilyReports/list` with `pageNumber` 1 and `pageSize` 10.<br>2. `POST /FamilyReports/list` with `pageNumber` 2 and `pageSize` 10. | 1. First response `items` length is at most 10.<br>2. `totalCount` is greater than 10.<br>3. `totalPages` is `ceil(totalCount / 10)`.<br>4. Second page items do not duplicate the first page ids. | ALFQ-FAM-API-003 | `Alforqan/Alforqan.Core/Features/FamilyReports/Queries/FamilyReportQueries.cs` | 11+ reports |

### Wave 4c — Halaqa filter

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-14 | ALFQ-FAM-API-014 | Verify student halaqat endpoint returns only own active enrollments | Main | Student JWT with at least one Active `HalaqaStudent` and one Inactive or Finished enrollment | 1. `GET /FamilyReports/halaqat`. | 1. Response succeeds.<br>2. Every returned id is an Active enrollment for that student.<br>3. Inactive, Finished, and other students' halaqat are not listed. | ALFQ-FAM-API-001 | `Alforqan/Alforqan.Core/Features/FamilyReports/Queries/FamilyReportQueries.cs` | Student with mixed enrollments |
| TC-15 | ALFQ-FAM-API-015 | Verify parent halaqat endpoint scopes by child and rejects foreign studentId | Main | Parent JWT linked to two children in different Active halaqat | 1. `GET /FamilyReports/halaqat` with no `studentId`.<br>2. `GET /FamilyReports/halaqat?studentId=` one linked child.<br>3. `GET /FamilyReports/halaqat?studentId=` an unlinked student. | 1. First response is the union of both children's Active halaqat (no duplicates for a shared halaqa).<br>2. Second response lists only that child's Active halaqat.<br>3. Third request returns HTTP 403. | ALFQ-FAM-API-002 | `Alforqan/Alforqan.Api/Controllers/FamilyReports/FamilyReportsController.cs`, `Alforqan/Alforqan.Core/Features/FamilyReports/Queries/FamilyReportQueries.cs` | Parent + two children |
| TC-16 | ALFQ-FAM-API-016 | Verify list with halaqaId returns matching session items and no sard rows | Main | Student with session reports in two halaqat and at least one decided sard | 1. `POST /FamilyReports/list` with `halaqaId` of one enrolled halaqa. | 1. Every `kind` `session` item has that `halaqaId`.<br>2. No item has `kind` `sard`.<br>3. Session items from other halaqat are absent. | ALFQ-FAM-API-003 | `Alforqan/Alforqan.Core/Features/FamilyReports/Helpers/FamilyReportHalaqaFilter.cs` | Two halaqat + sard |
