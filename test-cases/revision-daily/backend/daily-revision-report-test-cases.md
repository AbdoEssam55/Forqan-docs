# Daily revision report GET — Backend Test Cases

## Metadata

- Layer: backend
- Related code: `Alforqan/Alforqan.Core/Features/DailyRevision/Helpers/DailyRevisionReportBuilder.cs`, `Alforqan/Alforqan.Core/Features/DailyRevision/Helpers/DailyRevisionAccessHelper.cs`
- Related docs: `Alforqan-Backend/docs/technical-docs/daily-revision-log.md`
- Automation: `Tests/Alforqan.Core.Tests/Features/DailyRevision/DailyRevisionReportBuilderTests.cs`, `Tests/Alforqan.Core.Tests/Features/DailyRevision/DailyRevisionAccessHelperTests.cs`

## Scope

GET report day statuses and read authorization.

## Test cases

### Report builder

| ID | Key | Case | Steps | Expected | Related code |
| --- | --- | --- | --- | --- | --- |
| TC-01 | ALFQ-REVDAY-RPT-001 | Verify missing and upcoming statuses | 1. Run RPT-001 unit test with fixed UTC and window dates. | 1. Past required day without row is `Missing`.<br>2. Future required day is `Upcoming`. | `DailyRevisionReportBuilderTests.cs` |
| TC-02 | ALFQ-REVDAY-RPT-002 | Verify delivered day payload | 1. Run RPT-002 unit test with one log row. | 1. Day status is delivered.<br>2. Range and listener fields are set.<br>3. `HasAudio` is false when no file path. | same |
| TC-03 | ALFQ-REVDAY-RPT-003 | Verify another student cannot read | 1. Run access helper test. | `CanRead` false when student id does not match assignment. | `DailyRevisionAccessHelperTests.cs` |
