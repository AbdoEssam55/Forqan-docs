# Sard session evaluate + delete access — Backend Test Cases

## Metadata
- Project: `alforqan`
- Feature: `sard`
- Layer: `backend`
- Routes: `-`
- APIs: `GET/PUT/POST /api/.../Sard/sessions/{id}/evaluation`, `DELETE /api/.../Sard/sessions/{id}`
- Related code: `Alforqan/Alforqan.Core/Features/Sard/Helpers/SardSessionAccessHelper.cs`, `Alforqan/Alforqan.Core/Features/Sard/Helpers/SardSessionErrorCodes.cs`, `Alforqan/Alforqan.Core/Features/Sard/Commands/SardEvaluationMediators.cs`, `Alforqan/Alforqan.Core/Features/Sard/Commands/SardSessionMediators.cs`, `Alforqan/Alforqan.Api/Controllers/Sard/SardController.cs`
- Related docs: `Alforqan/docs/technical-docs/sard-session-access.md`
- Automation: `Tests/Alforqan.Core.Tests/Features/Sard/SardSessionAccessHelperTests.cs`

## Scope
- Who may evaluate an open sard session.
- When an admin may soft-delete a sard session.

## Test cases

### Evaluate access

| ID | Key | Case | Type | Steps | Expected | Related code |
|----|-----|------|------|-------|----------|--------------|
| TC-01 | ALFQ-SARD-BE-ACCESS-001 | Verify Manage sard can evaluate without a session assignment | Main | 1. Call `SardSessionAccessHelper.CanEvaluate` for Scheduled/In progress with `hasManageSard` true and assignment false.<br>2. Repeat for Passed/Failed with manage true. | 1. Scheduled and In progress return true.<br>2. Passed and Failed return false. | `Alforqan/Alforqan.Core/Features/Sard/Helpers/SardSessionAccessHelper.cs` |
| TC-02 | ALFQ-SARD-BE-ACCESS-002 | Verify only an assigned teacher can evaluate when not Manage sard | Main | 1. Call `CanEvaluate` with committee true, Perform sard true, and assignment false.<br>2. Call `CanEvaluate` with assignment true and manage false. | 1. Committee without assignment returns false.<br>2. Assigned teacher is allowed on an open session. | `Alforqan/Alforqan.Core/Features/Sard/Helpers/SardSessionAccessHelper.cs` |
| TC-04 | ALFQ-SARD-BE-ACCESS-004 | Verify a user with no sard role is denied | Edge | 1. Call `CanEvaluate` with assignment, committee, manage, and perform all false. | The result is false. | `Alforqan/Alforqan.Core/Features/Sard/Helpers/SardSessionAccessHelper.cs` |

### Delete access

| ID | Key | Case | Type | Steps | Expected | Related code |
|----|-----|------|------|-------|----------|--------------|
| TC-03 | ALFQ-SARD-BE-ACCESS-003 | Verify only Scheduled and In progress sessions may be deleted | Main | 1. Call `CanDeleteUnevaluatedSession` for every `SardSessionStatus`. | 1. Scheduled and In progress return true.<br>2. Submitted, Awaiting decision, Passed, and Failed return false. | `Alforqan/Alforqan.Core/Features/Sard/Helpers/SardSessionAccessHelper.cs` |
