# Student Actions — Backend Test Cases (match)

## Metadata

- Project: `alforqan`
- Feature: `student-actions`
- Layer: `backend`
- Routes: `-`
- APIs: `POST /API/V01/StudentActions/match`
- Related code: `Alforqan/Alforqan.Core/Features/StudentActions/Helpers/StudentActionMatchHelper.cs`
- Related docs: `docs/technical/tenant/student-actions.md`
- Automation: `Tests/Alforqan.Core.Tests/Features/StudentActions/StudentActionMatchHelperTests.cs`

## Scope

- Match students by attendance status counts in a date range (AND/OR rules).
- `Absent` is unexcused only; `Excused` is separate.
- Endpoint requires `ViewStudentActionsClaim`.

## Delivery waves

| Wave | Intent | Keys |
| ---- | ------ | ---- |
| 1 | Match rules + auth | `ALFQ-ACT-BE-001`–`005` |

## Test cases

### Wave 1 — Match rules + auth

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-01 | ALFQ-ACT-BE-001 | Verify Absent greater than or equal to 3 matches that student and not a student with 2 Absents | Main | Tenant with saved attendance marks in range | 1. POST match with one rule Absent gte 3 and a date range covering marks.<br>2. Note students with exactly 2 unexcused Absent marks in that range. | 1. Students with at least 3 Absent marks appear in the page.<br>2. Students with only 2 Absent marks do not appear. | - | `Alforqan/Alforqan.Core/Features/StudentActions/Queries/MatchStudentActionsQuery.cs`, `Alforqan/Alforqan.Core/Features/StudentActions/Helpers/StudentActionMatchHelper.cs` | Absent >= 3 |
| TC-02 | ALFQ-ACT-BE-002 | Verify Excused marks do not count toward Absent rules | Main | Student A has Excused marks only; Student B has Absent marks | 1. POST match with rule Absent gte 1. | 1. Student A is not returned.<br>2. Student B is returned when they have at least one Absent mark. | ALFQ-ACT-BE-001 | `Alforqan/Alforqan.Core/Features/StudentActions/Helpers/StudentActionMatchHelper.cs` | - |
| TC-03 | ALFQ-ACT-BE-003 | Verify AND requires every rule and OR requires any rule | Main | Students with mixed Absent/Late counts | 1. POST match with combinator AND, rules Absent gte 3 and Late gte 5.<br>2. Repeat with combinator OR and the same rules. | 1. AND returns only students meeting both rules.<br>2. OR returns students meeting at least one rule. | ALFQ-ACT-BE-001 | `Alforqan/Alforqan.Core/Features/StudentActions/Helpers/StudentActionMatchHelper.cs` | - |
| TC-04 | ALFQ-ACT-BE-004 | Verify empty rules or Value 366 is rejected | Edge | - | 1. POST match with `Rules: []`.<br>2. POST match with one rule Value 366. | 1. Response fails with localized "Add at least one attendance rule." / Arabic equivalent.<br>2. Response fails with localized value range message. | - | `Alforqan/Alforqan.Core/Features/StudentActions/Helpers/StudentActionMatchHelper.cs`, `Alforqan/Alforqan.Api/Controllers/StudentActions/StudentActionsController.cs` | 366 |
| TC-05 | ALFQ-ACT-BE-005 | Verify match requires ViewStudentActionsClaim | Main | Tenant user without ViewStudentActionsClaim | 1. POST match with a valid body using that user token. | 1. Response is 403 Forbidden. | - | `Alforqan/Alforqan.Api/Controllers/StudentActions/StudentActionsController.cs`, `Alforqan/Alforqan.Shared/HelperClasses/TenantPoliciesHelper.cs` | - |
