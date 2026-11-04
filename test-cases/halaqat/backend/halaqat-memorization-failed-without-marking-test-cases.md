# Halaqa memorization failed without marking — Backend Test Cases

## Metadata
- Project: `alforqan`
- Feature: `halaqat`
- Layer: `backend`
- APIs: `PUT .../evaluations/students/{studentId}`, `POST .../evaluations/students/{studentId}/submit`, `POST .../evaluations/students/{studentId}/outcome`
- Related code: `Alforqan/Alforqan.Core/Features/HalaqaSessions/Commands/SessionEvaluationMediators.cs`, `Alforqan/Alforqan.Core/Features/HalaqaSessions/Commands/RecordSessionEvaluationOutcomeCommand.cs`, `Alforqan/Alforqan.Data/Entities/SessionEvaluations/SessionEvaluation.cs`
- Related docs: `docs/features/halaqat-grading.md`
- Automation: `Tests/Alforqan.Core.Tests/Features/Grading/GradingEngineTests.cs`

## Scope

- Persist `FailedWithoutMemorization` on memorization drafts, clear mistakes on save, score 0% on submit, and force failed outcome without progress update.

## Test cases

### Wave 1 — Persist and score

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-01 | ALFQ-HAL-HIF-API-001 | Verify draft upsert stores failedWithoutMemorization and clears mistakes | Main | In-progress memorization evaluation | 1. Upsert draft with `failedWithoutMemorization: true` and non-empty mistakes.<br>2. GET evaluation. | 1. `failedWithoutMemorization` is true.<br>2. `mistakes` array is empty.<br>3. Range fields unchanged. | - | `Alforqan/Alforqan.Core/Features/HalaqaSessions/Commands/SessionEvaluationMediators.cs` | Memorization type 0 |
| TC-02 | ALFQ-HAL-HIF-API-002 | Verify submit with flag scores zero percent | Main | Draft with failedWithoutMemorization true | 1. POST submit for memorization. | 1. `totalScore` is 0.<br>2. `maxPossibleScore` is 0.<br>3. `scorePercentage` is 0. | ALFQ-HAL-HIF-API-001 | `Alforqan/Alforqan.Core/Features/HalaqaSessions/Commands/SessionEvaluationMediators.cs` | - |
| TC-03 | ALFQ-HAL-HIF-API-003 | Verify outcome records failed and skips progress | Main | Submitted memorization with failedWithoutMemorization | 1. POST outcome with `failedWithoutMemorization: true`. | 1. `sessionOutcome` is Failed.<br>2. `updateMemorizationProgress` is false or null.<br>3. Certified reached fields are null. | ALFQ-HAL-HIF-API-002 | `Alforqan/Alforqan.Core/Features/HalaqaSessions/Commands/RecordSessionEvaluationOutcomeCommand.cs` | - |
