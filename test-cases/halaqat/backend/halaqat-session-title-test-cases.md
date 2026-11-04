# Halaqa session auto title — Backend Test Cases

## Metadata
- Project: `alforqan`
- Feature: `halaqat`
- Layer: `backend`
- Routes: `/API/V01/HalaqaSessions/create`, `/API/V01/HalaqaSessions/{id}`
- Related code: `Alforqan/Alforqan.Core/Features/HalaqaSessions/Helpers/HalaqaSessionTitleGenerator.cs`, `Alforqan/Alforqan.Core/Features/HalaqaSessions/Commands/HalaqaSessionMutations.cs`
- Related docs: `docs/features/halaqat.md`
- Automation: `Tests/Alforqan.Core.Tests/Features/HalaqaSessions/HalaqaSessionTitleGeneratorTests.cs`

## Scope

- Server-generated bilingual session titles from halaqa name and local scheduled date.
- Same-day second session appends local time to the title.

## Delivery waves

| Wave | Intent | Keys |
| ---- | ------ | ---- |
| 1 | Generator format and local calendar | `ALFQ-HAL-SES-TTL-API-001`–`003` |
| 2 | Same-day time suffix rules | `ALFQ-HAL-SES-TTL-API-004`–`005` |
| 3 | Display helper for reports | `ALFQ-HAL-SES-TTL-API-006` |

## Test cases

### Wave 1 — Generator format

| ID | Key | Case | Steps | Expected | Requires | Related code |
|----|-----|------|-------|----------|----------|--------------|
| TC-01 | ALFQ-HAL-SES-TTL-API-001 | Verify generated titles include local date and Arabic and English weekdays | 1. Run xUnit `Generate_saturday_local_date_includes_arabic_and_english_weekdays`. | 1. Arabic title contains "14 / 11 / 2026" and "السبت".<br>2. English title contains "Saturday". | - | `Alforqan/Alforqan.Core/Features/HalaqaSessions/Helpers/HalaqaSessionTitleGenerator.cs` |
| TC-02 | ALFQ-HAL-SES-TTL-API-002 | Verify weekday uses local offset not UTC calendar day | 1. Run xUnit `Generate_uses_local_calendar_day_not_utc`. | 1. Title shows local date "14 / 11 / 2026" and Arabic Saturday. | ALFQ-HAL-SES-TTL-API-001 | `Alforqan/Alforqan.Core/Features/HalaqaSessions/Helpers/HalaqaSessionTitleGenerator.cs` |
| TC-03 | ALFQ-HAL-SES-TTL-API-003 | Verify first session of the day has no time suffix in generator | 1. Run xUnit `Generate_without_time_has_no_hh_mm_suffix`. | 1. Generated titles do not contain " / HH:mm". | ALFQ-HAL-SES-TTL-API-001 | `Alforqan/Alforqan.Core/Features/HalaqaSessions/Helpers/HalaqaSessionTitleGenerator.cs` |

### Wave 2 — Same-day time

| ID | Key | Case | Steps | Expected | Requires | Related code |
|----|-----|------|-------|----------|----------|--------------|
| TC-04 | ALFQ-HAL-SES-TTL-API-004 | Verify generator appends local HH:mm when includeTime is true | 1. Run xUnit `Generate_with_time_appends_local_hh_mm`. | 1. Both titles end with " / 16:30" for the fixture UTC instant and offset +180. | ALFQ-HAL-SES-TTL-API-003 | `Alforqan/Alforqan.Core/Features/HalaqaSessions/Helpers/HalaqaSessionTitleGenerator.cs` |
| TC-05 | ALFQ-HAL-SES-TTL-API-005 | Verify ShouldIncludeSessionTime only when another session shares the local date | 1. Run xUnit `ShouldIncludeSessionTime_false_when_no_other_session_same_day`.<br>2. Run xUnit `ShouldIncludeSessionTime_true_when_other_session_same_day`. | 1. Different local date returns false.<br>2. Same local date returns true. | ALFQ-HAL-SES-TTL-API-003 | `Alforqan/Alforqan.Core/Features/HalaqaSessions/Helpers/HalaqaSessionTitleGenerator.cs` |

### Wave 3 — Display

| ID | Key | Case | Steps | Expected | Requires | Related code |
|----|-----|------|-------|----------|----------|--------------|
| TC-06 | ALFQ-HAL-SES-TTL-API-006 | Verify DisplayTitle prefers TitleAr or TitleEn by culture with legacy fallback | 1. Run xUnit `DisplayTitle_prefers_bilingual_fields_by_culture`. | 1. Arabic culture uses TitleAr.<br>2. English culture uses TitleEn.<br>3. Null bilingual fields fall back to Title. | ALFQ-HAL-SES-TTL-API-001 | `Alforqan/Alforqan.Core/Features/HalaqaSessions/Helpers/HalaqaSessionTitleGenerator.cs` |
