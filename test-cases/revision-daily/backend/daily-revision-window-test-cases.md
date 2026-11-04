# Daily revision window — Backend Test Cases

## Metadata

- Layer: backend
- Related code: `Alforqan/Alforqan.Core/Features/DailyRevision/Helpers/DailyRevisionWindowHelper.cs`
- Related docs: `Alforqan-Backend/docs/technical-docs/daily-revision-log.md`
- Automation: `Tests/Alforqan.Core.Tests/Features/DailyRevision/DailyRevisionWindowHelperTests.cs`

## Scope

Required dates between assigning and next session; student-local status.

## Test cases

### Window helper

| ID | Key | Case | Steps | Expected | Related code |
| --- | --- | --- | --- | --- | --- |
| TC-01 | ALFQ-REVDAY-WIN-001 | Verify Sunday–Wednesday window | 1. Run test with assign Sunday and next Wednesday. | Required dates are Monday and Tuesday only. | `DailyRevisionWindowHelper.cs` |
| TC-02 | ALFQ-REVDAY-WIN-002 | Verify extended next session | 1. Run test with next session moved to Thursday. | Monday, Tuesday, and Wednesday are required. | same |
| TC-03 | ALFQ-REVDAY-WIN-003 | Verify shortened next session | 1. Run test with next session on Tuesday. | 1. Only Monday is required.<br>2. Helper does not delete existing log rows. | same |
| TC-04 | ALFQ-REVDAY-WIN-004 | Verify back-to-back empty window | 1. Run test with consecutive session dates. | Required date list is empty. | same |
| TC-05 | ALFQ-REVDAY-WIN-005 | Verify schedule-only next weekday | 1. Assign Sunday with schedule Sun+Wed and no target session row. | Next session date resolves to Wednesday. | same |
| TC-06 | ALFQ-REVDAY-WIN-006 | Verify Riyadh midnight boundary | 1. Use fixed UTC before and after 21:00 on 15 Jan 2026. | Student today and missing/due status match Riyadh calendar. | same |
