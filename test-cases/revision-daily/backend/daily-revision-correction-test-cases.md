# Daily revision correction — Backend Test Cases

## Metadata

- Layer: backend
- Related code: `Alforqan/Alforqan.Core/Features/DailyRevision/Commands/DailyRevisionMediators.cs`, `Alforqan/Alforqan.Core/Features/DailyRevision/Helpers/DailyRevisionAccessHelper.cs`, `Alforqan/Alforqan.Shared/HelperClasses/TenantPoliciesHelper.cs`
- Related docs: `Alforqan-Backend/docs/technical-docs/daily-revision-log.md`
- Automation: `Tests/Alforqan.Core.Tests/Features/DailyRevision/DailyRevisionCorrectionRulesTests.cs`

## Scope

Admin `CorrectDailyRevisionClaim` POST for missing past days only.

## Test cases

### Correction

| ID | Key | Case | Steps | Expected | Related code |
| --- | --- | --- | --- | --- | --- |
| TC-01 | ALFQ-REVDAY-COR-001 | Verify missing day correction sets CorrectedByUserId | 1. Run COR-001 unit test on correction eligibility helper. | 1. Missing past day inside window is eligible.<br>2. Saved row sets `CorrectedByUserId`. | `DailyRevisionCorrectionRulesTests.cs` |
| TC-02 | ALFQ-REVDAY-COR-002 | Verify claim required | 1. Call POST correct without `CorrectDailyRevisionClaim`. | HTTP 403. | `DailyRevisionController.cs` |
| TC-03 | ALFQ-REVDAY-COR-003 | Verify existing on-time row rejected | 1. Run COR-003 unit test. | Delivered day is not eligible for correction. | `DailyRevisionCorrectionRulesTests.cs` |
| TC-04 | ALFQ-REVDAY-COR-004 | Verify overlap before coverage on correct | 1. Run COR-004 using coverage helper. | Overlap before full coverage still rejected. | `DailyRevisionCoverageHelperTests.cs` |
