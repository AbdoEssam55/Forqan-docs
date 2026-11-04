# Daily revision audio — Backend Test Cases

## Metadata

- Layer: backend
- Related code: `Alforqan/Alforqan.Core/Features/DailyRevision/Helpers/DailyRevisionAudioValidationRules.cs`, `Alforqan/Alforqan.Core/Features/DailyRevision/Helpers/DailyRevisionAudioHelper.cs`
- Related docs: `Alforqan-Backend/docs/technical-docs/daily-revision-log.md`
- Automation: `Tests/Alforqan.Core.Tests/Features/DailyRevision/DailyRevisionAudioValidationRulesTests.cs`

## Scope

Optional audio limits on PUT (5 MB, 5 minutes, allowed extensions).

## Test cases

### Validation

| ID | Key | Case | Steps | Expected | Related code |
| --- | --- | --- | --- | --- | --- |
| TC-01 | ALFQ-REVDAY-AUD-001 | Verify submit without file allowed | 1. PUT revision log without audio part (manual or API). | 1. Row saves successfully.<br>2. Audio columns remain null. | `DailyRevisionMediators.cs` |
| TC-02 | ALFQ-REVDAY-AUD-002 | Verify over 5 MB rejected | 1. Run AUD-002 unit test. | Validation fails with too-large key. | `DailyRevisionAudioValidationRulesTests.cs` |
| TC-03 | ALFQ-REVDAY-AUD-003 | Verify non-audio extension rejected | 1. Run AUD-003 unit test. | `.pdf` fails invalid-type key. | same |
| TC-04 | ALFQ-REVDAY-AUD-004 | Verify duration over 300 rejected | 1. Run AUD-004 unit test. | Duration 301 fails too-long key. | same |
