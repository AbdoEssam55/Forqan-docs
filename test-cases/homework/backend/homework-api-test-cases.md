# Homework API — Backend Test Cases

## Metadata
- Project: `alforqan`
- Feature: `homework`
- Layer: `backend`
- APIs: `StudentAssignments` CRUD + List
- Related code: `Alforqan/Alforqan.Service/Implementation/StudentAssignments/Services/StudentAssignmentService.cs`, `Alforqan/Alforqan.Core/Features/StudentAssignments/Helpers/HomeworkMemorizationWindowHelper.cs`, `Alforqan/Alforqan.Service/Implementation/StudentAssignments/Helpers/HomeworkAccessHelper.cs`, `Alforqan/Alforqan.Core/Features/StudentAssignments/Helpers/HomeworkNotificationHelper.cs`
- Related docs: `docs/features/student-assignments.md`, `docs/technical-docs/hifz-direction-progress.md`
- Automation: `Tests/Alforqan.Core.Tests/Features/Homework/HomeworkApiTests.cs`, `Tests/Alforqan.Core.Tests/Features/Homework/QuranHomeworkRangeTests.cs`, `Tests/Alforqan.Core.Tests/Features/Homework/HomeworkMemorizationWindowTests.cs`

## Scope

- Homework CRUD with from/to ranges, one Open per `(student, halaqa, type)`, role scoping, notifications, next-session due date.

## Delivery waves

| Wave | Intent | Keys | Depends on |
| ---- | ------ | ---- | ---------- |
| 1 | Schema + range helpers | `ALFQ-HW-API-001`–`003` | — |
| 2 | CRUD API + scope + notify | `ALFQ-HW-API-004`–`012` | Wave 1 |
| 3 | Evaluation lock + override | `ALFQ-HW-API-013`–`020` | Wave 2 — see `homework-evaluation-lock-test-cases.md` |
| 5 | Memorization hifz window (floor + sard) | `ALFQ-HW-API-021`–`030` | Wave 2 |

## Test cases

### Wave 2 — Homework API

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-04 | ALFQ-HW-API-004 | Verify revision homework accepts multiple non-overlapping ranges | Main | Valid ranges | 1. POST create with AssignmentType Revision and two non-overlapping ranges. | 1. API returns 201.<br>2. Response includes both ranges with SortOrder 0 and 1. | ALFQ-HW-API-001 | `Alforqan/Alforqan.Service/Implementation/StudentAssignments/Services/StudentAssignmentService.cs` | Two surahs |
| TC-05 | ALFQ-HW-API-005 | Verify overlapping revision ranges return 400 | Main | Overlapping ranges on same surah | 1. POST create with overlapping revision ranges. | API returns 400 Bad Request. | ALFQ-HW-API-003 | `Alforqan/Alforqan.Core/Features/StudentAssignments/Helpers/HomeworkRangeHelper.cs` | Overlap |
| TC-06 | ALFQ-HW-API-006 | Verify second Open of same type cancels the first | Main | Existing Open hifz homework | 1. Create Open hifz homework.<br>2. Create another Open hifz for same student+halaqa. | 1. First assignment Status becomes Cancelled.<br>2. Second assignment Status is Open.<br>3. Student list hides the Cancelled row. | ALFQ-HW-API-004 | `Alforqan/Alforqan.Service/Implementation/StudentAssignments/Services/StudentAssignmentService.cs` | Same student/halaqa |
| TC-07 | ALFQ-HW-API-007 | Verify teacher list is scoped to own halaqat only | Main | Two teachers with separate halaqat | 1. Teacher A creates homework in halaqa A.<br>2. Teacher B lists assignments. | Teacher B list does not include Teacher A homework. | - | `Alforqan/Alforqan.Service/Implementation/StudentAssignments/Helpers/HomeworkAccessHelper.cs` | Two teachers |
| TC-08 | ALFQ-HW-API-008 | Verify student list shows only own homework | Main | Student account | 1. List assignments as student. | Only rows where StudentId matches the logged-in student are returned; Cancelled hidden. | - | `Alforqan/Alforqan.Service/Implementation/StudentAssignments/Helpers/HomeworkAccessHelper.cs` | Student login |
| TC-09 | ALFQ-HW-API-009 | Verify parent list shows linked children only | Main | Parent linked to child | 1. List assignments as parent. | Rows for linked children only; Cancelled hidden. | - | `Alforqan/Alforqan.Service/Implementation/StudentAssignments/Helpers/HomeworkAccessHelper.cs` | Parent login |
| TC-10 | ALFQ-HW-API-010 | Verify create notifies student and linked parents | Main | Student with parent link | 1. Teacher creates homework. | 1. Student receives in-app notification type HomeworkAssigned.<br>2. Linked parent receives the same notification.<br>3. ReferenceType is StudentAssignment. | ALFQ-HW-API-004 | `Alforqan/Alforqan.Core/Features/StudentAssignments/Helpers/HomeworkNotificationHelper.cs` | Parent linked |
| TC-11 | ALFQ-HW-API-011 | Verify update Open homework re-notifies on range change | Main | Open homework exists | 1. Edit Open homework ranges. | Student and parents receive updated homework notification. | ALFQ-HW-API-004 | `Alforqan/Alforqan.Core/Features/StudentAssignments/Helpers/HomeworkNotificationHelper.cs` | Open row |
| TC-12 | ALFQ-HW-API-012 | Verify DueDate defaults from next scheduled session | Main | Future session exists for halaqa | 1. Create homework without DueDate or SessionId. | 1. DueDate equals next HalaqaSession ScheduledAt.<br>2. SessionId equals that session Id. | - | `Alforqan/Alforqan.Service/Implementation/StudentAssignments/Helpers/HomeworkNextSessionHelper.cs` | Future session |

### Wave 5 — Memorization hifz window

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-21 | ALFQ-HW-API-021 | Verify create memorization past pending sard returns gate error | Main | Student with pending sard checkpoint | 1. POST create memorization homework with To past the pending boundary. | 1. API returns 400.<br>2. Message is `SARD_GATE_MEMORIZATION_RANGE_BLOCKED`.<br>3. `errorsBag.checkpointLabel` names the pending checkpoint. | ALFQ-HW-API-004 | `Alforqan/Alforqan.Core/Features/StudentAssignments/Helpers/HomeworkMemorizationWindowHelper.cs`, `Alforqan/Alforqan.Core/Features/StudentAssignments/Commands/Handlers/AddStudentAssignmentCommandHandler.cs` | Mid-Qur'an student |
| TC-22 | ALFQ-HW-API-022 | Verify create memorization ending at pending boundary succeeds | Main | Student with pending sard | 1. POST create with range ending exactly at the pending boundary. | API returns 201 and stores the range. | ALFQ-HW-API-021 | `Alforqan/Alforqan.Core/Features/StudentAssignments/Helpers/HomeworkMemorizationWindowHelper.cs` | At-boundary range |
| TC-23 | ALFQ-HW-API-023 | Verify create memorization From before last reached returns floor error | Main | Student reached surah 2 ayah 50 | 1. POST create with From surah 1. | 1. API returns 400.<br>2. Message matches localized `HomeworkRangeBeforeReached`. | ALFQ-HW-API-004 | `Alforqan/Alforqan.Core/Features/StudentAssignments/Helpers/HomeworkMemorizationWindowHelper.cs` | Reached 2:50 |
| TC-24 | ALFQ-HW-API-024 | Verify revision create ignores memorization floor and sard window | Main | Same student as TC-21 | 1. POST create revision homework with range before last reached or past sard. | API returns 201 (revision is not gated by memorization window). | ALFQ-HW-API-021 | `Alforqan/Alforqan.Service/Implementation/StudentAssignments/Services/StudentAssignmentService.cs` | Revision ranges |
| TC-25 | ALFQ-HW-API-025 | Verify update Open memorization uses same floor and sard errors | Main | Open memorization homework | 1. PUT update with From before reached.<br>2. PUT update with To past sard. | 1. First call returns `HomeworkRangeBeforeReached`.<br>2. Second call returns `SARD_GATE_MEMORIZATION_RANGE_BLOCKED`. | ALFQ-HW-API-023 | `Alforqan/Alforqan.Core/Features/StudentAssignments/Commands/Handlers/EditStudentAssignmentCommandHandler.cs` | Open row |
| TC-26 | ALFQ-HW-API-026 | Verify confirm-replace past sard does not cancel Open homework | Main | Locked eval with Open homework | 1. Upsert memorization draft with different range past sard and `ConfirmReplaceHomework` true. | 1. Upsert returns gate error.<br>2. Existing Open homework remains Open (not Cancelled). | ALFQ-HW-API-013 | `Alforqan/Alforqan.Core/Features/HalaqaSessions/Commands/SessionEvaluationMediators.cs` | Locked eval |
| TC-27 | ALFQ-HW-API-027 | Verify from-end memorization window toward Fatiha | Main | From-end student | 1. Create range toward Fatiha inside pending zone.<br>2. Create From at An-Nas when floor is past Nas.<br>3. Create past pending toward Fatiha. | 1. Inside-zone range succeeds.<br>2. From before floor is rejected.<br>3. Past pending returns sard gate error. | ALFQ-SARD-BE-HIFZ-002 | `Alforqan/Alforqan.Core/Features/StudentAssignments/Helpers/HomeworkMemorizationWindowHelper.cs` | From-end student |
| TC-28 | ALFQ-HW-API-028 | Verify no progress floor is 1:1 from-start and 114:1 from-end | Main | New student with no reached surah | 1. Read floor for from-start student.<br>2. Read floor for from-end student. | 1. From-start floor is Al-Fatiha 1:1.<br>2. From-end floor is An-Nas 114:1. | - | `Alforqan/Alforqan.Core/Features/StudentAssignments/Helpers/HomeworkMemorizationWindowHelper.cs` | New students |
| TC-29 | ALFQ-HW-API-029 | Verify start cap crosses surah boundary at last reached ayah | Main | From-start student reached surah 1 ayah 7 | 1. Assert `GetStartCap` is surah 2 ayah 1.<br>2. POST create memorization From 2:1 To 2:3.<br>3. POST create memorization From 2:2 To 2:5. | 1. Start cap is Al-Baqarah 2:1 after Al-Fatiha 1:7.<br>2. Create with From 2:1 succeeds.<br>3. Create with From 2:2 returns `HomeworkRangeBeforeReached`. | ALFQ-HW-API-004 | `Alforqan/Alforqan.Core/Features/StudentAssignments/Helpers/HomeworkMemorizationWindowHelper.cs`, `Alforqan/Alforqan.Core/Features/Sard/Helpers/SardCheckpointHelper.cs` | Reached 1:7 |
| TC-30 | ALFQ-HW-API-030 | Verify from-end start cap after last ayah is ayah 1 of the previous surah | Main | From-end student reached Al-Humaza 104:9 | 1. Assert `GetStartCap` is 103:1.<br>2. POST create memorization From 103:1 To 103:1.<br>3. POST create memorization From 103:2 To 103:2. | 1. Start cap is Al-Asr 103:1 (not Humaza 104:8).<br>2. Create with From 103:1 succeeds.<br>3. Create with From 103:2 returns `HomeworkRangeBeforeReached`. | ALFQ-HW-API-004 | `Alforqan/Alforqan.Core/Features/StudentAssignments/Helpers/HomeworkMemorizationWindowHelper.cs`, `Alforqan/Alforqan.Core/Features/Sard/Helpers/SardCheckpointHelper.cs` | Reached 104:9 |
