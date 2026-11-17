# Daily revision window — Backend Test Cases

## Metadata

- Layer: backend
- Related code: `Alforqan/Alforqan.Core/Features/DailyRevision/Helpers/DailyRevisionWindowHelper.cs`, `DailyRevisionReportBuilder.cs`, `DailyRevisionReviewRules.cs`
- Related docs: `docs/technical-docs/tutor-started-sessions.md`
- Automation: `Tests/Alforqan.Core.Tests/Features/DailyRevision/DailyRevisionWindowHelperTests.cs`, `DailyRevisionReportBuilderTests.cs`

## Scope

Owed days run from the assigning session until the tutor starts the next session (the deadline). Catch-up is allowed before the deadline. Submissions after it are stored as late and never counted.

## Test cases

### Window helper

| ID | Key | Case | Steps | Expected | Related code |
| --- | --- | --- | --- | --- | --- |
| TC-01 | ALFQ-REVDAY-WIN-001 | Verify a closed Sunday–Wednesday window | 1. Assign Sunday.<br>2. Tutor starts on Wednesday. | Monday and Tuesday are owed. | `DailyRevisionWindowHelper.cs` |
| TC-02 | ALFQ-REVDAY-WIN-002 | Verify an open window runs through today | 1. Assign Sunday, no session started.<br>2. Today is Thursday. | Monday to Thursday are owed. | same |
| TC-03 | ALFQ-REVDAY-WIN-003 | Verify starting late keeps every day owed | 1. Tutor starts 10 days after assignment. | 1. All 9 days in between are owed.<br>2. The start day is not owed. | same |
| TC-04 | ALFQ-REVDAY-WIN-004 | Verify a back-to-back empty window | 1. Start the day after assignment.<br>2. Open window where today is the assign day. | Both owed lists are empty. | same |
| TC-05 | ALFQ-REVDAY-WIN-005 | Verify deadline resolution | 1. Open homework with a started target.<br>2. Open homework without a started target.<br>3. Done legacy homework without a started target. | 1. Deadline is `StartedAt`.<br>2. No deadline.<br>3. Planned time, then due date, then assignment time. | same |
| TC-06 | ALFQ-REVDAY-WIN-006 | Verify the Riyadh midnight boundary | 1. Use fixed UTC before and after 21:00 on 15 Jan 2026. | Student today and missing/due status match the Riyadh calendar. | same |
| TC-07 | ALFQ-REVDAY-WIN-007 | Verify the late flag at the deadline | 1. Submit 1 second before `StartedAt`.<br>2. Submit at it.<br>3. Submit after it. | 1. Not late.<br>2. Late.<br>3. Late. | same |

### Report and submit

| ID | Key | Case | Steps | Expected | Related code |
| --- | --- | --- | --- | --- | --- |
| TC-08 | ALFQ-REVDAY-RPT-020 | Verify an open window lists days through today | 1. Build the report with no started target. | 1. Days run up to today.<br>2. `DeadlineAt` is null.<br>3. `IsPastDeadline` is false. | `DailyRevisionReportBuilder.cs` |
| TC-09 | ALFQ-REVDAY-RPT-021 | Verify a missed past day can be caught up | 1. A past owed day has nothing sent.<br>2. Build the report as the student. | 1. The day is Missing.<br>2. `CanSubmit` is true. | same |
| TC-10 | ALFQ-REVDAY-RPT-022 | Verify late parts are shown but not counted | 1. A day with one accepted late part covering the homework. | 1. The day is Missing with `IsLate` true.<br>2. The part has `IsLate` true.<br>3. `IsHomeworkFullyCovered` is false. | same |
| TC-11 | ALFQ-REVDAY-RPT-023 | Verify the report after the tutor starts the session | 1. Build the report after `StartedAt`. | 1. `IsPastDeadline` is true.<br>2. Owed days are still listed and each can be submitted as late. | same |
| TC-12 | ALFQ-REVDAY-REV-009 | Verify past-day submit rules | 1. Past day empty.<br>2. Past day refused or partly accepted.<br>3. Past day accepted. | 1. Allowed (catch-up or late).<br>2. Allowed.<br>3. Not allowed. | `DailyRevisionReviewRules.cs` |
| TC-13 | ALFQ-REVDAY-API-030 | Verify a submit after Start is stored late | 1. The tutor starts the session.<br>2. The student `PUT`s a missed day. | 1. 200.<br>2. The row has `IsLate = 1`.<br>3. The staff revision report still counts the day as not sent. | `DailyRevisionMediators.cs`, `RevisionReportQueryHelper.cs` |
| TC-14 | ALFQ-REVDAY-API-031 | Verify no schedule is required | 1. Open the daily revision of a halaqa with no weekly schedule. | 200, not `HalaqaScheduleRequired`. | `DailyRevisionMediators.cs` |
