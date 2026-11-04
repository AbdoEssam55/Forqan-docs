# Sard per-page grading — Backend Test Cases

## Metadata
- Project: `alforqan`
- Feature: `sard`
- Layer: `backend`
- APIs: Sard evaluation GET/upsert, submit
- Related code: `Alforqan/Alforqan.Service/Implementation/Grading/SardGradingHelper.cs`, `Alforqan/Alforqan.Core/Features/Sard/Commands/SardEvaluationMediators.cs`
- Related docs: `docs/features/halaqat-grading.md`
- Automation: `-`

## Scope

- Sard evaluation uses `GradingEvaluationScope.Sard` configs and per-page mushaf scoring.
- Allowance override fields on `SardEvaluation` match session evaluation behavior.

## Test cases

### Sard scoring

| ID | Key | Case | Type | Steps | Expected |
|----|-----|------|------|-------|----------|
| TC-01 | ALFQ-SARD-GRA-API-001 | Verify sard upsert returns gradingPreview with sard scope | Main | 1. Upsert sard evaluation draft with mistakes in scope.<br>2. Read `gradingPreview` on response. | 1. `pages` > 0.<br>2. `allowances` lists active types with admin and computed allowed.<br>3. `teacherOverrideLocked` matches sard grading config. |
| TC-02 | ALFQ-SARD-GRA-API-002 | Verify teacher override applies when unlocked on sard submit | Main | 1. Upsert with whole-range override all zeros.<br>2. Submit evaluation. | 1. `maxPossibleScore` is `0`.<br>2. Total score can be negative when marks exist. |
