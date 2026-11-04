# Daily revision submit — Backend Test Cases

## Metadata

- Layer: backend
- Related code: `Alforqan/Alforqan.Core/Features/DailyRevision/Commands/DailyRevisionMediators.cs`, `Alforqan/Alforqan.Core/Features/DailyRevision/Helpers/DailyRevisionAccessHelper.cs`, `Alforqan/Alforqan.Core/Features/DailyRevision/Helpers/DailyRevisionListenerRules.cs`, `Alforqan/Alforqan.Api/Controllers/DailyRevision/DailyRevisionController.cs`
- Related docs: `Alforqan-Backend/docs/technical-docs/daily-revision-log.md`
- Automation: `Tests/Alforqan.Core.Tests/Features/DailyRevision/DailyRevisionSubmitEligibilityTests.cs`, `Tests/Alforqan.Core.Tests/Features/DailyRevision/DailyRevisionAccessHelperTests.cs`, `Tests/Alforqan.Core.Tests/Features/DailyRevision/DailyRevisionListenerRulesTests.cs`, `Tests/Alforqan.Core.Tests/Features/DailyRevision/DailyRevisionCoverageHelperTests.cs`

## Scope

Student/parent PUT upsert rules (no client date; coverage and listener validation).

## Test cases

### Submit rules

| ID | Key | Case | Steps | Expected | Related code |
| --- | --- | --- | --- | --- | --- |
| TC-01 | ALFQ-REVDAY-SUB-001 | Verify due day allows first save | 1. Run SUB-001 unit test (due window + same local day). | Student-local today with status Due can submit. | `DailyRevisionSubmitEligibilityTests.cs` |
| TC-02 | ALFQ-REVDAY-SUB-002 | Verify overlapping second day rejected | 1. Run COV-003 / overlap coverage test. | Second day overlapping prior slice returns false before full coverage. | `DailyRevisionCoverageHelperTests.cs` |
| TC-03 | ALFQ-REVDAY-SUB-003 | Verify replace same day | 1. Run COV-007 replace test. | Same-day resubmit allowed when other days exclude today’s slice. | `DailyRevisionCoverageHelperTests.cs` |
| TC-04 | ALFQ-REVDAY-SUB-004 | Verify after midnight blocked | 1. Run SUB-004 unit test. | `IsStillSameLocalDay` false for prior revision date after boundary. | `DailyRevisionSubmitEligibilityTests.cs` |
| TC-05 | ALFQ-REVDAY-SUB-005 | Verify unlinked parent forbidden | 1. Run access helper test. | `CanSubmit` false for parent without linked student. | `DailyRevisionAccessHelperTests.cs` |
| TC-06 | ALFQ-REVDAY-SUB-006 | Verify Self with name rejected | 1. Run listener rules test. | Validation key `DailyRevisionListenerNameMustBeEmpty`. | `DailyRevisionListenerRulesTests.cs` |
| TC-07 | ALFQ-REVDAY-SUB-007 | Verify Father without name rejected | 1. Run listener rules test. | Validation key `DailyRevisionListenerNameRequired`. | `DailyRevisionListenerRulesTests.cs` |
