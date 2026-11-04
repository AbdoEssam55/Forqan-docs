# Student Actions — Backend Test Cases (commands)

## Metadata

- Project: `alforqan`
- Feature: `student-actions`
- Layer: `backend`
- Routes: `-`
- APIs: `POST /API/V01/StudentActions/notify`, `POST /API/V01/StudentActions/deactivate`
- Related code: `Alforqan/Alforqan.Core/Features/StudentActions/Commands/StudentActionCommands.cs`
- Related docs: `docs/technical/tenant/student-actions.md`
- Automation: `Tests/Alforqan.Core.Tests/Features/StudentActions/StudentActionNotifyAndDeactivateTests.cs`

## Scope

- Notify creates student in-app notifications; deactivate disables student login.
- TakeStudentActionsClaim on command endpoints.

## Delivery waves

| Wave | Intent | Keys |
| ---- | ------ | ---- |
| 1 | Notify + deactivate | `ALFQ-ACT-BE-010`–`014` |

## Test cases

### Wave 1 — Notify + deactivate

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-01 | ALFQ-ACT-BE-010 | Verify notify creates one unread notification for the student user and not parents | Main | Student with login; parent linked | 1. POST notify with one studentId, title, body.<br>2. Sign in as student; open bell.<br>3. Sign in as parent; open bell. | 1. Student sees one unread notification with the title.<br>2. Parent does not receive this notification. | ALFQ-ACT-BE-001 | `Alforqan/Alforqan.Core/Features/StudentActions/Helpers/StudentActionNotifyHelper.cs` | - |
| TC-02 | ALFQ-ACT-BE-011 | Verify notify skips students with no UserId and returns skippedCount | Main | Student without login user | 1. POST notify including that studentId and a valid student with login. | 1. Response `notifiedCount` counts only students with UserId.<br>2. `skippedStudentIds` includes the student without login. | ALFQ-ACT-BE-001 | `Alforqan/Alforqan.Core/Features/StudentActions/Helpers/StudentActionMatchHelper.cs`, `Alforqan/Alforqan.Core/Features/StudentActions/Commands/StudentActionCommands.cs` | - |
| TC-03 | ALFQ-ACT-BE-012 | Verify deactivate sets AppUser and Student IsActive false and login is blocked | Main | Active student login | 1. POST deactivate with that studentId.<br>2. Load student record.<br>3. Attempt student login. | 1. API succeeds with `deactivatedCount` 1.<br>2. `Student.IsActive` is false.<br>3. Login fails or account inactive message appears. | ALFQ-ACT-BE-001 | `Alforqan/Alforqan.Core/Features/StudentActions/Commands/StudentActionCommands.cs`, `Alforqan/Alforqan.Service/Implementation/Identity/TenantUserActivationService.cs` | - |
| TC-04 | ALFQ-ACT-BE-013 | Verify notify and deactivate require TakeStudentActionsClaim | Main | User with ViewStudentActionsClaim only | 1. POST notify with valid body.<br>2. POST deactivate with valid body. | 1. Both responses are 403 Forbidden. | ALFQ-ACT-BE-005 | `Alforqan/Alforqan.Api/Controllers/StudentActions/StudentActionsController.cs` | - |
| TC-05 | ALFQ-ACT-BE-014 | Verify more than 100 studentIds is rejected | Edge | - | 1. POST notify with 101 distinct studentIds. | 1. Response fails with localized max-students message. | - | `Alforqan/Alforqan.Core/Features/StudentActions/Helpers/StudentActionMatchHelper.cs` | 101 ids |
