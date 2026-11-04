# Daily revision coverage — Backend Test Cases

## Metadata

- Layer: backend
- Related code: `Alforqan/Alforqan.Core/Features/DailyRevision/Helpers/DailyRevisionCoverageHelper.cs`
- Related docs: `Alforqan-Backend/docs/technical-docs/daily-revision-log.md`
- Automation: `Tests/Alforqan.Core.Tests/Features/DailyRevision/DailyRevisionCoverageHelperTests.cs`

## Scope

Homework range containment and overlap rules before full coverage.

## Test cases

### Coverage helper

| ID | Key | Case | Steps | Expected | Related code |
| --- | --- | --- | --- | --- | --- |
| TC-01 | ALFQ-REVDAY-COV-001 | Verify segment inside one range | 1. Run COV-001 unit test. | Segment resolves to exactly one homework range. | `DailyRevisionCoverageHelper.cs` |
| TC-02 | ALFQ-REVDAY-COV-002 | Verify segment outside homework | 1. Run COV-002 unit test. | `TryResolveRangeId` returns false. | same |
| TC-03 | ALFQ-REVDAY-COV-003 | Verify overlap before full coverage | 1. Run COV-003 unit test. | `CanAddSlice` returns false. | same |
| TC-04 | ALFQ-REVDAY-COV-004 | Verify non-overlapping surahs | 1. Run COV-004 unit test. | Later then earlier surah without overlap is allowed. | same |
| TC-05 | ALFQ-REVDAY-COV-005 | Verify completing slice without overlap | 1. Run COV-005 unit test. | Final non-overlapping slice completing coverage is allowed. | same |
| TC-06 | ALFQ-REVDAY-COV-006 | Verify overlap after full coverage | 1. Run COV-006 unit test. | Overlap allowed when homework already fully covered. | same |
| TC-07 | ALFQ-REVDAY-COV-007 | Verify replace excludes same day | 1. Run COV-007 unit test. | Same ayahs can be saved again when other-day list excludes today. | same |
