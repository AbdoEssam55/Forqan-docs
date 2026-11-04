# Homework evaluation lock — Backend Test Cases

## Metadata
- Project: `alforqan`
- Feature: `homework`
- Layer: `backend`
- APIs: `HalaqaSessions` session evaluation GET/upsert/outcome
- Related code: `Alforqan/Alforqan.Core/Features/HalaqaSessions/Commands/SessionEvaluationMediators.cs`, `Alforqan/Alforqan.Core/Features/HalaqaSessions/Helpers/HomeworkEvaluationLockHelper.cs`, `Alforqan/Alforqan.Core/Features/HalaqaSessions/Commands/RecordSessionEvaluationOutcomeCommand.cs`, `Alforqan/Alforqan.Service/Implementation/Reports/EvaluationReportPdfGenerator.cs`
- Related docs: `docs/technical-docs/homework-evaluation-lock.md`, `docs/features/student-assignments.md`
- Automation: `Tests/Alforqan.Core.Tests/Features/Homework/HomeworkEvaluationLockTests.cs`

## Scope

- Homework locks session evaluation ranges; override with confirm; multi-master revision; auto-Done on outcome; PDF multi-master list.

## Delivery waves

| Wave | Intent | Keys | Depends on |
| ---- | ------ | ---- | ---------- |
| 3 | Evaluation lock + override + PDF | `ALFQ-HW-API-013`–`020` | Wave 2 |

## Test cases

### Wave 3 — Evaluation lock

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-13 | ALFQ-HW-API-013 | Verify GET evaluation copies Open homework ranges when draft is clean | Main | Open homework for session; InProgress eval with no mistakes or loaded segments | 1. GET session evaluation for student/type. | 1. Response `RangeLockedFromHomework` is true.<br>2. `LockedHomeworkId` matches Open homework.<br>3. Evaluation ranges match homework ranges. | ALFQ-HW-API-004 | `Alforqan/Alforqan.Core/Features/HalaqaSessions/Commands/SessionEvaluationMediators.cs` | Open homework |
| TC-14 | ALFQ-HW-API-014 | Verify GET does not overwrite ranges when mistakes or loaded segments exist | Main | InProgress eval with mistakes or `AyahsLoaded` segment | 1. GET session evaluation. | 1. Existing evaluation ranges are unchanged.<br>2. Homework ranges are not re-applied. | ALFQ-HW-API-013 | `Alforqan/Alforqan.Core/Features/HalaqaSessions/Helpers/HomeworkEvaluationLockHelper.cs` | Draft with marks |
| TC-15 | ALFQ-HW-API-015 | Verify upsert returns 400 when locked ranges differ without confirm | Main | Open homework; locked InProgress eval | 1. Upsert draft with different ranges and `ConfirmReplaceHomework` false. | API returns 400 with localized `SessionEvaluationConfirmReplaceHomeworkRequired`. | ALFQ-HW-API-013 | `Alforqan/Alforqan.Core/Features/HalaqaSessions/Commands/SessionEvaluationMediators.cs` | Different range |
| TC-16 | ALFQ-HW-API-016 | Verify upsert with confirm replaces Open homework and trims mistakes outside union | Main | Open homework; locked eval with marks outside new range | 1. Upsert with new ranges and `ConfirmReplaceHomework` true. | 1. Prior Open homework is Cancelled.<br>2. New Open homework exists with new ranges.<br>3. Mistakes outside the new union are removed. | ALFQ-HW-API-015 | `Alforqan/Alforqan.Core/Features/HalaqaSessions/Helpers/HomeworkEvaluationLockHelper.cs` | Marks outside range |
| TC-17 | ALFQ-HW-API-017 | Verify revision segment must reference MasterRangeId and stay within that master | Main | Revision eval with master ranges | 1. Upsert segment outside its `MasterRangeId` bounds. | API returns 400 with `SessionEvaluationSegmentOutsideMaster`. | ALFQ-HW-API-013 | `Alforqan/Alforqan.Core/Features/HalaqaSessions/Commands/SessionEvaluationMediators.cs` | Invalid segment |
| TC-18 | ALFQ-HW-API-018 | Verify overlapping revision master ranges are rejected | Main | Revision upsert | 1. Upsert with two overlapping `MasterRanges`. | API returns 400 with `SessionEvaluationMasterRangesOverlap`. | ALFQ-HW-API-003 | `Alforqan/Alforqan.Core/Features/HalaqaSessions/Helpers/HomeworkEvaluationLockHelper.cs` | Overlap |
| TC-19 | ALFQ-HW-API-019 | Verify recording outcome marks matching Open homework Done | Main | Submitted eval; Open homework for session | 1. POST record session evaluation outcome. | Matching Open homework `Status` becomes Done. | ALFQ-HW-API-013 | `Alforqan/Alforqan.Core/Features/HalaqaSessions/Commands/RecordSessionEvaluationOutcomeCommand.cs` | Submitted eval |
| TC-20 | ALFQ-HW-API-020 | Verify evaluation PDF lists each master when more than one exists | Main | Revision eval with 2+ master ranges and outcome recorded | 1. Download evaluation PDF. | PDF revision summary lists each master range separately (not only envelope). | ALFQ-HW-API-013 | `Alforqan/Alforqan.Service/Implementation/Reports/EvaluationReportPdfGenerator.cs` | Multi-master revision |
