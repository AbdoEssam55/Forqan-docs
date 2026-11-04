# Session date admin — Backend Test Cases

## Metadata

- Layer: backend
- Related code: `Alforqan/Alforqan.Core/Features/HalaqaSessions/Helpers/HalaqaSessionDatePolicyHelper.cs`, `Alforqan/Alforqan.Core/Features/HalaqaSessions/Helpers/HalaqaSessionScheduledAtPolicyApplication.cs`
- Related docs: `Alforqan-Backend/docs/features/halaqat.md`
- Automation: `Tests/Alforqan.Core.Tests/Features/HalaqaSessions/HalaqaSessionDatePolicyHelperTests.cs`

## Scope

Admin-only session `ScheduledAt` changes and teacher weekday rules.

## Test cases

### Policy helper

| ID | Key | Case | Steps | Expected | Related code |
| --- | --- | --- | --- | --- | --- |
| TC-01 | ALFQ-REVDAY-DATE-001 | Verify scheduled-at change detection | 1. Compare UTC instants more than one second apart. | `ScheduledAtChanged` is true. | `HalaqaSessionDatePolicyHelper.cs` |
| TC-02 | ALFQ-REVDAY-DATE-002 | Verify unchanged within tolerance | 1. Compare instants within one second. | `ScheduledAtChanged` is false. | same |
| TC-03 | ALFQ-REVDAY-DATE-003 | Verify schedule weekday match | 1. Use Monday session instant in `Asia/Riyadh` with Monday schedule row. | `IsOnScheduleWeekday` is true. | same |
| TC-04 | ALFQ-REVDAY-DATE-004 | Verify non-schedule weekday rejected | 1. Use Tuesday instant with Monday-only schedule. | `IsOnScheduleWeekday` is false. | same |
| TC-05 | ALFQ-REVDAY-DATE-005 | Verify admin role bypass flag | 1. Pass Admin role name to `IsAdminRole`. | Returns true. | same |
