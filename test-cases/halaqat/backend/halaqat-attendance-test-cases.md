# Halaqa session attendance — Backend Test Cases

## Metadata

- Project: `alforqan`
- Feature: `halaqat`
- Layer: `backend`
- Routes: `/API/V01/HalaqaSessions/{sessionId}/attendance/bulk`, `/API/V01/HalaqaSessions/{sessionId}/complete`, `/API/V01/HalaqaSessions/{sessionId}/evaluations/students/{studentId}/outcome`
- APIs: `POST .../attendance/bulk`, `PUT .../complete`, `POST .../evaluations/students/{studentId}/outcome`
- Related code: `Alforqan/Alforqan.Core/Features/HalaqaSessions/Helpers/SessionAttendanceRules.cs`
- Related docs: `docs/technical/tenant/halaqat-guide.md`
- Automation: `Tests/Alforqan.Core.Tests/Features/HalaqaSessions/SessionAttendanceRulesTests.cs`

## Scope

- Late duration rules: Late requires 1–1440 minutes; other statuses drop late minutes.

## Delivery waves

| Wave | Intent | Keys |
| ---- | ------ | ---- |
| 1 | Late duration rules | `ALFQ-HAL-ATT-API-001`–`004` |

## Test cases

### Wave 1 — Late duration rules

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-01 | ALFQ-HAL-ATT-API-001 | Verify Late without minutes is rejected and other statuses are not | Main | - | 1. Call `SessionAttendanceRules.LateMinutesError` with Present and no minutes.<br>2. Call it with Late and no minutes. | 1. Present returns no error.<br>2. Late returns `SessionAttendanceLateMinutesRequired`. | - | `Alforqan/Alforqan.Core/Features/HalaqaSessions/Helpers/SessionAttendanceRules.cs` | - |
| TC-02 | ALFQ-HAL-ATT-API-002 | Verify Late accepts 1 through 1440 minutes | Main | - | 1. Call `LateMinutesError` with Late and 1, 15, and 1440. | Each call returns no error. | ALFQ-HAL-ATT-API-001 | `Alforqan/Alforqan.Core/Features/HalaqaSessions/Helpers/SessionAttendanceRules.cs` | 1, 15, 1440 |
| TC-03 | ALFQ-HAL-ATT-API-003 | Verify non-Late statuses drop late minutes | Main | - | 1. Call `NormalizeLateMinutes` with Present and 15.<br>2. Call it with Late and 15. | 1. Present result is empty.<br>2. Late result is 15. | ALFQ-HAL-ATT-API-001 | `Alforqan/Alforqan.Core/Features/HalaqaSessions/Helpers/SessionAttendanceRules.cs` | 15 |
| TC-04 | ALFQ-HAL-ATT-API-004 | Verify Late rejects 0 and more than 24 hours | Edge | - | 1. Call `LateMinutesError` with Late and 0.<br>2. Call it with Late and 1441. | Both return `SessionAttendanceLateMinutesRequired`. | ALFQ-HAL-ATT-API-001 | `Alforqan/Alforqan.Core/Features/HalaqaSessions/Helpers/SessionAttendanceRules.cs` | 0, 1441 |
