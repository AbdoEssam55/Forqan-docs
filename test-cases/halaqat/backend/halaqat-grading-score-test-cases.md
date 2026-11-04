# Halaqa per-page grading score — Backend Test Cases

## Metadata
- Project: `alforqan`
- Feature: `halaqat`
- Layer: `backend`
- APIs: -
- Related code: `Alforqan/Alforqan.Service/Implementation/Grading/MushafPageFractionHelper.cs`, `Alforqan/Alforqan.Service/Implementation/Grading/GradingScoreCalculator.cs`, `Alforqan/Alforqan.Service/Implementation/Grading/GradingConfigResolver.cs`, `Alforqan/Alforqan.Core/Features/HalaqaSessions/Commands/GradingConfigMediators.cs`
- Related docs: `docs/features/halaqat-grading.md`
- Automation: `Tests/Alforqan.Core.Tests/Features/Grading/GradingScoreCalculatorTests.cs`, `Tests/Alforqan.Core.Tests/Features/Grading/GradingConfigResolverTests.cs`, `Tests/Alforqan.Core.Tests/Features/Grading/GradingEngineTests.cs`

## Scope

- Pure scoring math for Ottoman mushaf page fractions, admin allowed-per-page, teacher session overrides, inactive types, negative totals, and grading config resolution (halaqa vs institute defaults).

## Delivery waves

| Wave | Intent | Keys | Depends on |
| ---- | ------ | ---- | ---------- |
| 1 | Page fraction + calculator | `ALFQ-HAL-GRA-API-001`–`009` | — |
| 2 | Config resolver | `ALFQ-HAL-GRA-API-010`–`012` | Wave 1 |
| 3 | Grading engine + preview | `ALFQ-HAL-GRA-API-013`–`015` | Waves 1–2 |

## Test cases

### Wave 1 — Scoring math

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-01 | ALFQ-HAL-GRA-API-001 | Verify full mushaf pages sum to whole page count | Main | Test ayahs cover all ayahs on pages 1 and 2 (10 ayahs each) | 1. Run `MushafPageFractionHelper.ComputePageFraction` with page lookup 10+10. | Page fraction equals `2`. | - | `Alforqan/Alforqan.Service/Implementation/Grading/MushafPageFractionHelper.cs` | 20 ayahs, 2 pages |
| TC-02 | ALFQ-HAL-GRA-API-002 | Verify partial page uses ayah fraction | Main | Page 12 has 15 ayahs; tested range has 7 ayahs on page 12 | 1. Compute page fraction for 7 ayahs on page 12. | Page fraction equals `7/15`. | - | `Alforqan/Alforqan.Service/Implementation/Grading/MushafPageFractionHelper.cs` | 7 of 15 ayahs |
| TC-03 | ALFQ-HAL-GRA-API-003 | Verify three full pages with admin config produces expected max and score | Main | Admin: حفظ 5/page weight 3; تجويد 2/page weight 2; تشكيل 3/page weight 1; 3 full pages; marks 2/1/0 | 1. Run `GradingScoreCalculator.Calculate` with lock on. | 1. Allowed حفظ `15`, تجويد `6`, تشكيل `9`.<br>2. Max possible `66`.<br>3. Total score `58`.<br>4. Percentage `87.88`. | - | `Alforqan/Alforqan.Service/Implementation/Grading/GradingScoreCalculator.cs` | 3 pages |
| TC-04 | ALFQ-HAL-GRA-API-004 | Verify one ayah enforces minimum one allowed per active type | Main | One ayah on a 15-ayah page; admin defaults | 1. Calculate with zero marks. | 1. Each active type allowed count is `1`.<br>2. Max possible is `6`. | - | `Alforqan/Alforqan.Service/Implementation/Grading/GradingScoreCalculator.cs` | 1 ayah |
| TC-05 | ALFQ-HAL-GRA-API-005 | Verify marks over allowance produce negative score | Main | 1 full page; حفظ allowed 1 weight 3; 5 حفظ marks | 1. Calculate score. | 1. Total score is `-12`.<br>2. Percentage is `-400`. | - | `Alforqan/Alforqan.Service/Implementation/Grading/GradingScoreCalculator.cs` | 5 marks |
| TC-06 | ALFQ-HAL-GRA-API-006 | Verify teacher whole-range override can set all types to zero | Main | Teacher override whole range 0/0/0; 1 mark each type | 1. Calculate with override unlocked. | 1. All allowed counts are `0`.<br>2. Max possible `0`.<br>3. Total score `-6`. | - | `Alforqan/Alforqan.Service/Implementation/Grading/GradingScoreCalculator.cs` | All zero |
| TC-07 | ALFQ-HAL-GRA-API-007 | Verify inactive mistake type contributes zero | Main | تجويد inactive; 10 تجويد marks | 1. Calculate. | 1. تجويد allowed is `0`.<br>2. تجويد type score is `0`. | - | `Alforqan/Alforqan.Service/Implementation/Grading/GradingScoreCalculator.cs` | Inactive type |
| TC-08 | ALFQ-HAL-GRA-API-008 | Verify failed without attempt returns zero scores | Main | `failedWithoutAttempt` true | 1. Run xUnit `Calculate_failed_without_revision_returns_zero_scores`. | 1. Total score `0`.<br>2. Max possible `0`.<br>3. Percentage `0`. | - | `Alforqan/Alforqan.Service/Implementation/Grading/GradingScoreCalculator.cs` | Failed without attempt |
| TC-09 | ALFQ-HAL-GRA-API-009 | Verify teacher per-page override rounds without minimum | Main | 7/15 page; teacher حفظ 5/page; تجويد 0/page | 1. Calculate per-page override. | 1. حفظ allowed `2` (round 5×7/15).<br>2. تجويد allowed `0`. | - | `Alforqan/Alforqan.Service/Implementation/Grading/GradingScoreCalculator.cs` | Partial page |

### Wave 2 — Config resolver

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-10 | ALFQ-HAL-GRA-API-010 | Verify institute default used when halaqa row missing | Main | Institute revision تشكيل inactive, lock on | 1. Resolve for any halaqa without halaqa row. | 1. Allowed per page is `2`.<br>2. Active is false.<br>3. Teacher override locked is true. | ALFQ-HAL-GRA-API-001 | `Alforqan/Alforqan.Service/Implementation/Grading/GradingConfigResolver.cs` | Institute only |
| TC-11 | ALFQ-HAL-GRA-API-011 | Verify halaqa config wins over institute default | Main | Institute Mem 3/page; halaqa Mem 7/page | 1. Run `GradingConfigResolver.ResolveRow`. | 1. Allowed per page is `7`.<br>2. Weight is `2` from the halaqa row. | ALFQ-HAL-GRA-API-001 | `Alforqan/Alforqan.Service/Implementation/Grading/GradingConfigResolver.cs` | Halaqa id |
| TC-12 | ALFQ-HAL-GRA-API-012 | Verify sard scope and lock flag resolve | Main | Sard Mem 4/page, lock on | 1. Run `ToTypeConfigs` and `IsTeacherOverrideLocked`. | 1. Mem allowed per page is `4`.<br>2. Teacher override locked is true. | ALFQ-HAL-GRA-API-011 | `Alforqan/Alforqan.Service/Implementation/Grading/GradingConfigResolver.cs` | Sard scope |

### Wave 3 — Engine

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-13 | ALFQ-HAL-GRA-API-013 | Verify memorization engine uses page allowance not word count | Main | 1 full page, 10 ayahs, admin 1/page weights 3/2/1, 2 marks | 1. Run `GradingEngine.RecalculateMemorization`. | 1. Max `6`, total `1`, pct `16.67`.<br>2. Loaded score fields null. | ALFQ-HAL-GRA-API-003 | `Alforqan/Alforqan.Service/Implementation/Grading/GradingEngine.cs` | xUnit |
| TC-14 | ALFQ-HAL-GRA-API-014 | Verify revision engine scores loaded ayahs only | Main | 5 loaded ayahs, revision scope configs | 1. Run `RecalculateRevision` with empty mistakes. | 1. Max `3`, total `3`, pct `100`.<br>2. `LoadedMaxPossibleScore` null. | ALFQ-HAL-GRA-API-008 | `Alforqan/Alforqan.Service/Implementation/Grading/GradingEngine.cs` | xUnit |
| TC-15 | ALFQ-HAL-GRA-API-015 | Verify locked config ignores teacher override on engine | Main | Override all zero, configs locked | 1. Run `RecalculateMemorization`. | Max possible remains admin `3` (not zero). | ALFQ-HAL-GRA-API-006 | `Alforqan/Alforqan.Service/Implementation/Grading/GradingEngine.cs` | xUnit |
| TC-16 | ALFQ-HAL-HIF-001 | Verify memorization engine returns zero when failed without memorization | Main | `failedWithoutMemorization` true; ayahs in range present | 1. Run xUnit `RecalculateMemorization_failed_without_memorization_returns_zero_scores`. | 1. Total score `0`.<br>2. Max possible `0`.<br>3. Percentage `0`. | ALFQ-HAL-GRA-API-008 | `Alforqan/Alforqan.Service/Implementation/Grading/GradingEngine.cs` | xUnit |
