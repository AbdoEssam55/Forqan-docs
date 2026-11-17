# Tutor-started sessions — Backend Test Cases

## Metadata
- Project: `alforqan`
- Feature: `halaqat`
- Layer: `backend`
- APIs: `POST .../HalaqaSessions/halaqa/{halaqaId}/start-now`, `PUT .../HalaqaSessions/{id}/start`, `PUT .../HalaqaSessions/{id}/link`, `GET .../HalaqaSessions/mine/halaqat`, `GET .../HalaqaSessions/live`, `GET .../HalaqaSessions/mine`
- Related code: `Alforqan/Alforqan.Core/Features/HalaqaSessions/Helpers/HalaqaSessionStartPolicy.cs`, `HalaqaSessionStartEffects.cs`, `HalaqaSessionNowPlanner.cs`, `Alforqan/Alforqan.Api/BackgroundServices/SessionWindowBackgroundService.cs`
- Related docs: `docs/technical-docs/tutor-started-sessions.md`
- Automation: `Tests/Alforqan.Core.Tests/Features/HalaqaSessions/TeacherSessionListHelperTests.cs`, `HalaqaSessionNowPlannerTests.cs`

## Scope

Tutors start sessions on demand. The weekly schedule only sends reminders. Supersedes `session-start-window-test-cases.md`.

## Test cases

| ID | Key | Case | Steps | Expected | Related code |
| --- | --- | --- | --- | --- | --- |
| TC-01 | ALFQ-SESWIN-022 | Verify not-started rows can start at any time | 1. Check `CanStart` for each status. | Scheduled and LinkReady can start. InProgress, Completed and Cancelled cannot. | `HalaqaSessionStartPolicy.cs` |
| TC-02 | ALFQ-SESWIN-023 | Verify live and not-started sessions never drop off the board | 1. Session started or planned 60 days ago. | InProgress, Scheduled and LinkReady are included. | `TeacherSessionListHelper.cs` |
| TC-03 | ALFQ-SESWIN-024 | Verify finished sessions show for 8 days | 1. Completed 7 days ago.<br>2. Completed 9 days ago. | 1. Included.<br>2. Not included. | `TeacherSessionListHelper.cs` |
| TC-04 | ALFQ-SESWIN-025 | Verify start-now reuses only a recent live session | 1. Live session started 1 hour ago.<br>2. Live session started 2 days ago. | 1. Reused.<br>2. A new session is created. | `HalaqaSessionStartPolicy.cs` |
| TC-05 | ALFQ-SESWIN-026 | Verify the link can be set until the session ends | 1. Check `CanSetLink` per status. | True for Scheduled, LinkReady and InProgress. False for Completed. | `HalaqaSessionStartPolicy.cs` |
| TC-06 | ALFQ-SESNOW-001 | Verify duration comes from today's slot | 1. Monday slot 17:00–18:30 Riyadh.<br>2. Start on Monday and on Tuesday. | 1. 90 minutes on Monday.<br>2. 60 minutes on Tuesday. | `HalaqaSessionNowPlanner.cs` |
| TC-07 | ALFQ-SESNOW-002 | Verify the reminder span | 1. Early 15 minutes.<br>2. Check 13:40, 13:45, 15:30 and 15:31 UTC on Monday. | Due only from 13:45 to 15:30. | `HalaqaSessionNowPlanner.cs` |
| TC-08 | ALFQ-SESNOW-003 | Verify the reminder fires once per local day and not while live | 1. Already reminded today.<br>2. Session live.<br>3. Last reminded a week ago. | 1. No reminder.<br>2. No reminder.<br>3. Reminder. | `SessionWindowNotifyHelper.cs` |
| TC-09 | ALFQ-SESNOW-004 | Verify the next scheduled start keeps today's slot until it ends | 1. During Monday's slot.<br>2. After it. | 1. Today's start.<br>2. Next Monday's start. | `HalaqaSessionNowPlanner.cs` |
| TC-10 | ALFQ-SESNOW-005 | Verify Start attaches open homework | 1. Homework without a target.<br>2. Homework targeting a not-started session.<br>3. Homework targeting an earlier started session. | 1. Attached.<br>2. Attached.<br>3. Not attached. | `HalaqaSessionStartEffects.cs` |
| TC-11 | ALFQ-SESNOW-010 | Verify start-now as the halaqa tutor | 1. Teacher calls `start-now` with no link.<br>2. Check the session, homework and notifications. | 1. 201 with an InProgress session.<br>2. Last link reused.<br>3. Open homework `SessionId` is the new session.<br>4. Students and parents get notification type 11. | `HalaqaSessionMutations.cs` |
| TC-12 | ALFQ-SESNOW-011 | Verify start-now by another teacher | 1. A teacher of a different halaqa calls `start-now`. | 403. | `HalaqaSessionAccessHelper.cs` |
| TC-13 | ALFQ-SESNOW-012 | Verify a student sees only their halaqa live | 1. The tutor starts halaqa A.<br>2. A student of A and a student of B call `live`. | 1. The A student gets one item with the link.<br>2. The B student gets an empty list. | `GetTeacherHalaqaSessionsQuery.cs` |
| TC-14 | ALFQ-SESNOW-013 | Verify the migration cleanup | 1. Seed a system Scheduled row with open homework targeting it.<br>2. Apply the migration. | 1. Row soft-deleted.<br>2. Homework `SessionId` and `DueDate` are null.<br>3. Rows with attendance are kept. | `20261117120000_TutorStartedSessionsAndLateRevision.cs` |
