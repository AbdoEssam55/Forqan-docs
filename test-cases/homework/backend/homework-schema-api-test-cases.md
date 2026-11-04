# Homework schema — Backend Test Cases

## Metadata
- Project: `alforqan`
- Feature: `homework`
- Layer: `backend`
- APIs: -
- Related code: `Alforqan/Alforqan.Data/Entities/StudentAssignments/StudentAssignment.cs`, `Alforqan/Alforqan.Data/Entities/StudentAssignments/StudentAssignmentRange.cs`, `Alforqan/Alforqan.Data/Entities/SessionEvaluations/SessionEvaluationMasterRange.cs`, `Alforqan/Alforqan.Core/Features/StudentAssignments/Helpers/HomeworkRangeHelper.cs`
- Related docs: `docs/features/student-assignments.md`, `docs/business/Alforqan.Requirements.md`
- Automation: `Tests/Alforqan.Core.Tests/Features/Homework/QuranHomeworkRangeTests.cs`

## Scope

- Homework range model (from/to surah + ayah), child range rows, assignment status, revision eval master ranges, and range validation helpers.

## Delivery waves

| Wave | Intent | Keys | Depends on |
| ---- | ------ | ---- | ---------- |
| 1 | Schema + range helpers | `ALFQ-HW-API-001`–`003` | — |

## Test cases

### Wave 1 — Schema and range helpers

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-01 | ALFQ-HW-API-001 | Verify hifz homework accepts exactly one valid range | Main | Range helper loaded | 1. Call `ValidateRangesForType` with Memorization and one range Al-Baqarah 1–10. | Validation returns true. | - | `Alforqan/Alforqan.Core/Features/StudentAssignments/Helpers/HomeworkRangeHelper.cs` | Surah 2 |
| TC-02 | ALFQ-HW-API-002 | Verify hifz homework rejects multiple ranges | Main | Two non-overlapping ranges | 1. Call `ValidateRangesForType` with Memorization and two ranges. | Validation returns false. | - | `Alforqan/Alforqan.Core/Features/StudentAssignments/Helpers/HomeworkRangeHelper.cs` | Two ranges |
| TC-03 | ALFQ-HW-API-003 | Verify overlapping revision ranges are rejected | Main | Two overlapping ranges on same surah | 1. Call `HasOverlappingRanges`. | Overlap is detected (returns true). | - | `Alforqan/Alforqan.Core/Features/StudentAssignments/Helpers/HomeworkRangeHelper.cs` | Overlap |
