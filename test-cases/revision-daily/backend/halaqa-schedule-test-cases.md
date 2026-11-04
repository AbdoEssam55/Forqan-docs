# Halaqa schedule — Backend Test Cases

## Metadata

- Layer: backend
- Related code: `Alforqan/Alforqan.Core/Features/Halaqat/Helpers/HalaqaScheduleValidationHelper.cs`, `Alforqan/Alforqan.Core/Features/Halaqat/Services/HalaqaScheduleSyncService.cs`, `Alforqan/Alforqan.Data/Entities/Halaqat/HalaqaSchedule.cs`
- Related docs: `Alforqan-Backend/docs/technical-docs/halaqa-schedule.md`
- Automation: `Tests/Alforqan.Core.Tests/Features/Halaqat/HalaqaScheduleValidationHelperTests.cs`

## Scope

Halaqa weekly session days, timezone, and validation on create/update.

## Test cases

### Schedule validation

| ID | Key | Case | Steps | Expected | Related code |
| --- | --- | --- | --- | --- | --- |
| TC-01 | ALFQ-REVDAY-SCH-001 | Verify one valid weekday passes validation | 1. Call `ValidateSchedulePayload` with `Asia/Riyadh` and one row Sunday 16:00–18:00. | Validation returns null (success). | `Alforqan/Alforqan.Core/Features/Halaqat/Helpers/HalaqaScheduleValidationHelper.cs` |
| TC-02 | ALFQ-REVDAY-SCH-002 | Verify zero weekdays fails | 1. Call with empty schedules list. | Returns `HalaqaScheduleRequired`. | same |
| TC-03 | ALFQ-REVDAY-SCH-003 | Verify duplicate weekday fails | 1. Call with two rows for Monday. | Returns `HalaqaScheduleDuplicateDay`. | same |
| TC-04 | ALFQ-REVDAY-SCH-004 | Verify end before start fails | 1. Call with end 16:00 and start 18:00. | Returns `HalaqaScheduleEndBeforeStart`. | same |
| TC-05 | ALFQ-REVDAY-SCH-005 | Verify invalid timezone fails | 1. Call with `Not/A_Real_Zone`. | Returns `HalaqaTimeZoneInvalid`. | same |
