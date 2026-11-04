# Session start window — Backend Test Cases

## Metadata
- Project: `alforqan`
- Feature: `halaqat`
- Layer: `backend`
- APIs: `POST .../Halaqat`, `PUT .../Halaqat/{id}`, `POST .../HalaqaSessions/create`, `PUT .../HalaqaSessions/{id}/start`, `GET .../HalaqaSessions/mine`
- Related code: `Alforqan/Alforqan.Core/Features/HalaqaSessions/Helpers/SessionStartWindowHelper.cs`
- Related docs: `docs/technical-docs/session-start-window.md`
- Automation: `Tests/Alforqan.Core.Tests/Features/HalaqaSessions/SessionStartWindowHelperTests.cs`

## Scope

Weekly schedule creates the next session. Start is allowed only inside the halaqa-zone window. Teachers cannot create sessions.

## Delivery waves

| Wave | Intent | Keys |
| ---- | ------ | ---- |
| 1 | Window helper | ALFQ-SESWIN-001–005 |
| 2 | Early/late minutes | ALFQ-SESWIN-006–008 |
| 3 | Auto-create | ALFQ-SESWIN-010–013 |
| 4 | Start gate | ALFQ-SESWIN-015–018 |
| 5 | Teacher list | ALFQ-SESWIN-020–022 |
| 6 | Notify stamp | ALFQ-SESWIN-023–025 |
| 7 | Weekly date roll | ALFQ-SESWIN-033–035 |

## Test cases

### Wave 1 — Window helper

| ID | Key | Case | Steps | Expected | Related code |
| --- | --- | --- | --- | --- | --- |
| TC-01 | ALFQ-SESWIN-001 | Verify the window is open inside the early minutes | 1. Sunday 16:00–17:30 Asia/Riyadh, early 15, late 0.<br>2. UTC now 12:50 on that Sunday. | `IsOpen` is true. | `Alforqan/Alforqan.Core/Features/HalaqaSessions/Helpers/SessionStartWindowHelper.cs` |
| TC-02 | ALFQ-SESWIN-002 | Verify the window is closed before the early minutes | 1. Same schedule.<br>2. UTC now 12:40. | `IsOpen` is false. | `Alforqan/Alforqan.Core/Features/HalaqaSessions/Helpers/SessionStartWindowHelper.cs` |
| TC-03 | ALFQ-SESWIN-003 | Verify the window stays open until the end when late is 0 | 1. Same schedule, late 0.<br>2. UTC now 14:30.<br>3. UTC now 14:30:01. | 1. 14:30 is open.<br>2. 14:30:01 is closed. | `Alforqan/Alforqan.Core/Features/HalaqaSessions/Helpers/SessionStartWindowHelper.cs` |
| TC-04 | ALFQ-SESWIN-004 | Verify the window stays open through end plus late minutes | 1. Late 15.<br>2. UTC now 14:40.<br>3. UTC now 14:46. | 1. 14:40 is open.<br>2. 14:46 is closed. | `Alforqan/Alforqan.Core/Features/HalaqaSessions/Helpers/SessionStartWindowHelper.cs` |
| TC-05 | ALFQ-SESWIN-005 | Verify the next date uses the halaqa zone, not the UTC date | 1. UTC Wednesday 21:30.<br>2. Zone Asia/Riyadh.<br>3. Weekday Thursday. | Occurrence date is Thursday 8 Oct 2026, not Wednesday 7 Oct. | `Alforqan/Alforqan.Core/Features/HalaqaSessions/Helpers/SessionStartWindowHelper.cs` |

### Wave 2 — Early and late minutes

| ID | Key | Case | Steps | Expected | Related code |
| --- | --- | --- | --- | --- | --- |
| TC-06 | ALFQ-SESWIN-006 | Verify 15 and 0 are valid | 1. Validate early 15 and late 0. | No error key. | `Alforqan/Alforqan.Core/Features/Halaqat/Helpers/HalaqaStartWindowMinutesHelper.cs` |
| TC-07 | ALFQ-SESWIN-007 | Verify 181 is rejected | 1. Validate early 181 or late 181. | Key `HalaqaStartWindowMinutesInvalid`. | `Alforqan/Alforqan.Core/Features/Halaqat/Helpers/HalaqaStartWindowMinutesHelper.cs` |
| TC-08 | ALFQ-SESWIN-008 | Verify a negative value is rejected | 1. Validate early -1 or late -1. | Key `HalaqaStartWindowMinutesInvalid`. | `Alforqan/Alforqan.Core/Features/Halaqat/Helpers/HalaqaStartWindowMinutesHelper.cs` |

### Wave 3 — Auto-create

| ID | Key | Case | Steps | Expected | Related code |
| --- | --- | --- | --- | --- | --- |
| TC-10 | ALFQ-SESWIN-010 | Verify planning the next session uses start time and duration | 1. Sunday 16:00–17:30 Asia/Riyadh.<br>2. UTC now 08:00 that Sunday. | 1. Duration is 90.<br>2. `ScheduledAtUtc` is 13:00. | `Alforqan/Alforqan.Core/Features/HalaqaSessions/Services/SessionOccurrencePlanner.cs` |
| TC-11 | ALFQ-SESWIN-011 | Verify a second plan does not duplicate that date | 1. Existing row on that Sunday.<br>2. Plan next. | Result is null. | `Alforqan/Alforqan.Core/Features/HalaqaSessions/Services/SessionOccurrencePlanner.cs` |
| TC-12 | ALFQ-SESWIN-012 | Verify a teacher cannot create a session | 1. `TeacherMayCreate(false)`.<br>2. `TeacherMayCreate(true)`. | 1. False for teacher.<br>2. True for admin. | `Alforqan/Alforqan.Core/Features/HalaqaSessions/Helpers/HalaqaSessionStartPolicy.cs` |
| TC-13 | ALFQ-SESWIN-013 | Verify a future Scheduled row can be moved and a Completed row cannot | 1. `ShouldUpdateTimes` for Scheduled before window.<br>2. Same for Completed. | 1. Scheduled is true.<br>2. Completed is false. | `Alforqan/Alforqan.Core/Features/HalaqaSessions/Services/SessionOccurrencePlanner.cs` |

### Wave 4 — Start gate

| ID | Key | Case | Steps | Expected | Related code |
| --- | --- | --- | --- | --- | --- |
| TC-15 | ALFQ-SESWIN-015 | Verify start from Scheduled is allowed when the window is open | 1. `CanStart(Scheduled, true)`. | True. | `Alforqan/Alforqan.Core/Features/HalaqaSessions/Helpers/HalaqaSessionStartPolicy.cs` |
| TC-16 | ALFQ-SESWIN-016 | Verify start is refused before the window | 1. `CanStart(Scheduled, false)`. | False. | `Alforqan/Alforqan.Core/Features/HalaqaSessions/Helpers/HalaqaSessionStartPolicy.cs` |
| TC-17 | ALFQ-SESWIN-017 | Verify start is refused after the window | 1. `IsMissed` after window end.<br>2. `CanStart` false. | Missed is true and start is false. | `Alforqan/Alforqan.Core/Features/HalaqaSessions/Helpers/HalaqaSessionStartPolicy.cs` |
| TC-18 | ALFQ-SESWIN-018 | Verify start from LinkReady is allowed when the window is open | 1. `CanStart(LinkReady, true)`. | True. | `Alforqan/Alforqan.Core/Features/HalaqaSessions/Helpers/HalaqaSessionStartPolicy.cs` |

### Wave 5 — Teacher list

| ID | Key | Case | Steps | Expected | Related code |
| --- | --- | --- | --- | --- | --- |
| TC-20 | ALFQ-SESWIN-020 | Verify a teacher matches only his own halaqa | 1. Compare the same user ids. | True. | `Alforqan/Alforqan.Core/Features/HalaqaSessions/Helpers/TeacherSessionListHelper.cs` |
| TC-21 | ALFQ-SESWIN-021 | Verify another teacher does not match | 1. Compare different user ids. | False. | `Alforqan/Alforqan.Core/Features/HalaqaSessions/Helpers/TeacherSessionListHelper.cs` |
| TC-22 | ALFQ-SESWIN-022 | Verify the open row can start and a later closed row cannot | 1. Open window `CanStart`.<br>2. Closed window `CanStart`. | 1. True.<br>2. False. | `Alforqan/Alforqan.Core/Features/HalaqaSessions/Helpers/HalaqaSessionStartPolicy.cs` |

### Wave 6 — Notify stamp

| ID | Key | Case | Steps | Expected | Related code |
| --- | --- | --- | --- | --- | --- |
| TC-23 | ALFQ-SESWIN-023 | Verify the first open tick should notify | 1. Window open, stamp null. | `ShouldNotify` is true. | `Alforqan/Alforqan.Core/Features/HalaqaSessions/Helpers/SessionWindowNotifyHelper.cs` |
| TC-24 | ALFQ-SESWIN-024 | Verify a second tick does not notify again | 1. Window open, stamp set. | `ShouldNotify` is false. | `Alforqan/Alforqan.Core/Features/HalaqaSessions/Helpers/SessionWindowNotifyHelper.cs` |
| TC-25 | ALFQ-SESWIN-025 | Verify a closed window does not notify | 1. Window closed, stamp null. | `ShouldNotify` is false. | `Alforqan/Alforqan.Core/Features/HalaqaSessions/Helpers/SessionWindowNotifyHelper.cs` |

### Wave 7 — Weekly date roll

| ID | Key | Case | Steps | Expected | Related code |
| --- | --- | --- | --- | --- | --- |
| TC-33 | ALFQ-SESWIN-033 | Verify a passed Scheduled date rolls to the next future weekday | 1. Sunday 16:00–17:30 Asia/Riyadh, occurrence 4 Oct 2026, Scheduled.<br>2. UTC now 08:00 on Sunday 25 Oct 2026.<br>3. Plan roll forward. | 1. Occurrence date is 25 Oct 2026.<br>2. `ScheduledAtUtc` is 13:00. | `Alforqan/Alforqan.Core/Features/HalaqaSessions/Services/SessionOccurrencePlanner.cs` |
| TC-34 | ALFQ-SESWIN-034 | Verify Completed and InProgress dates do not roll | 1. Same occurrence 4 Oct 2026.<br>2. UTC now 08:00 on 25 Oct 2026.<br>3. Plan roll for Completed then InProgress. | 1. Completed result is null.<br>2. InProgress result is null. | `Alforqan/Alforqan.Core/Features/HalaqaSessions/Services/SessionOccurrencePlanner.cs` |
| TC-35 | ALFQ-SESWIN-035 | Verify two weekdays both roll to their next future dates | 1. Sunday 4 Oct and Tuesday 6 Oct 2026, both Scheduled.<br>2. UTC now 08:00 on Sunday 25 Oct 2026. | 1. Sunday rolls to 25 Oct 2026.<br>2. Tuesday rolls to 27 Oct 2026. | `Alforqan/Alforqan.Core/Features/HalaqaSessions/Services/SessionOccurrencePlanner.cs` |
