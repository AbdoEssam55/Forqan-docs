# Halaqa sessions schedule save — Backend Test Cases

## Metadata
- Project: `alforqan`
- Feature: `halaqat`
- Layer: `backend`
- Routes: `-`
- APIs: `PUT .../Halaqat/{id}`
- Related code: `Alforqan/Alforqan.Core/Features/HalaqaSessions/Services/SessionOccurrencePlanner.cs`
- Related docs: `docs/technical-docs/session-start-window.md`
- Automation: `Tests/Alforqan.Core.Tests/Features/HalaqaSessions/SessionOccurrencePlannerTests.cs`

## Scope

Saving the weekly schedule creates a current not-started session for each saved weekday. A weekday that already has a Scheduled, LinkReady, or InProgress row is left as-is. Completed rows do not block a new occurrence.

## Delivery waves

| Wave | Intent | Keys |
| ---- | ------ | ---- |
| 1 | Missing weekdays | `ALFQ-HAL-SCH-013`–`015` |

## Test cases

### Wave 1 — Missing weekdays

| ID | Key | Case | Steps | Expected | Requires | Related code |
| --- | --- | --- | --- | --- | --- | --- |
| TC-01 | ALFQ-HAL-SCH-013 | Verify planning missing weekdays creates one occurrence per saved day | 1. Sunday and Tuesday 16:00–17:30 Asia/Riyadh.<br>2. UTC now 08:00 on Sunday 4 Oct 2026.<br>3. No existing sessions. | 1. Two planned rows.<br>2. Sunday occurrence is 4 Oct 2026.<br>3. Tuesday occurrence is 6 Oct 2026. | - | `Alforqan/Alforqan.Core/Features/HalaqaSessions/Services/SessionOccurrencePlanner.cs` |
| TC-02 | ALFQ-HAL-SCH-014 | Verify a Scheduled row on one weekday still plans the other weekday | 1. Same schedule as TC-01.<br>2. Existing Scheduled session on Sunday 4 Oct 2026. | 1. One planned row.<br>2. That row is Tuesday 6 Oct 2026. | ALFQ-HAL-SCH-013 | `Alforqan/Alforqan.Core/Features/HalaqaSessions/Services/SessionOccurrencePlanner.cs` |
| TC-03 | ALFQ-HAL-SCH-015 | Verify a Completed row does not block the next occurrence for that weekday | 1. Sunday schedule only.<br>2. Existing Completed session on Sunday 27 Sep 2026.<br>3. UTC now 08:00 on Sunday 4 Oct 2026. | The planned occurrence is Sunday 4 Oct 2026. | ALFQ-HAL-SCH-013 | `Alforqan/Alforqan.Core/Features/HalaqaSessions/Services/SessionOccurrencePlanner.cs` |
