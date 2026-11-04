# Student timezone — Backend Test Cases

## Metadata

- Layer: backend
- Related code: `Alforqan/Alforqan.Core/Features/Students/Helpers/StudentTimeZoneHelper.cs`, `Alforqan/Alforqan.Core/Features/Students/Helpers/StudentNationalityTimeZoneRules.cs`
- Related docs: `Alforqan-Backend/docs/technical-docs/student-timezone.md`
- Automation: `Tests/Alforqan.Core.Tests/Features/Students/StudentTimeZoneHelperTests.cs`

## Scope

Residence-country timezone resolution and student-local calendar dates. Nationality must not set the zone.

## Test cases

### Helper

| ID | Key | Case | Type | Steps | Expected | Related code |
| --- | --- | --- | --- | --- | --- | --- |
| TC-01 | ALFQ-REVDAY-TZ-001 | Verify SA residence resolves Asia/Riyadh | Main | 1. Call `TryResolveTimeZoneId` with residence country SA and no explicit zone. | Resolved id is `Asia/Riyadh`. | `Alforqan/Alforqan.Core/Features/Students/Helpers/StudentTimeZoneHelper.cs` |
| TC-02 | ALFQ-REVDAY-TZ-002 | Verify US residence without zone fails | Main | 1. Call with residence country US and null zone. | Returns false. | `Alforqan/Alforqan.Core/Features/Students/Helpers/StudentTimeZoneHelper.cs` |
| TC-03 | ALFQ-REVDAY-TZ-003 | Verify Riyadh midnight boundary | Main | 1. Use fixed UTC `2026-01-15T20:59:00Z` and `21:00:00Z` with `Asia/Riyadh`. | 1. `20:59:00Z` maps to 15 Jan.<br>2. `21:00:00Z` maps to 16 Jan. | `Alforqan/Alforqan.Core/Features/Students/Helpers/StudentTimeZoneHelper.cs` |
| TC-04 | ALFQ-REVDAY-TZ-007 | Verify AF residence resolves Asia/Kabul | Main | 1. Call `TryResolveTimeZoneId` with residence country AF and no explicit zone. | Resolved id is `Asia/Kabul`. | `Alforqan/Alforqan.Core/Features/Students/Helpers/StudentTimeZoneHelper.cs` |
| TC-05 | ALFQ-REVDAY-TZ-008 | Verify TryApply uses residence not nationality | Main | 1. Call `TryApply` with Nationality EG, ResidenceCountry AF, TimeZoneId null. | 1. Returns true.<br>2. `TimeZoneId` is `Asia/Kabul`. | `Alforqan/Alforqan.Core/Features/Students/Helpers/StudentNationalityTimeZoneRules.cs` |
