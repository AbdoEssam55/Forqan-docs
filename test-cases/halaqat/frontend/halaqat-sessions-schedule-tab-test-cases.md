# Halaqa sessions schedule tab — Frontend Test Cases

## Metadata
- Project: `alforqan`
- Feature: `halaqat`
- Layer: `frontend`
- Routes: `/{locale}/{tenantSlug}/halaqat/{halaqaId}/sessions`, `/{locale}/{tenantSlug}/halaqat/{halaqaId}` (Sessions tab)
- APIs: `GET .../Halaqat/{id}`, `GET .../HalaqaSessions`, `PUT .../Halaqat/{id}` (inline schedule save)
- Related code: `src/features/alforqan/sessions/halaqa-sessions-schedule-panel.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/halaqat/[id]/sessions/halaqa-sessions-list-content.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/halaqat/[id]/halaqa-detail-content.tsx`
- Related docs: `Alforqan-Frontend/docs/technical/tenant/halaqat-guide.md`, `Alforqan-Frontend/docs/explanation/tenant/halaqat-explained.md`
- Automation: `-`

## Scope

- Sessions list and halaqa detail **Sessions** tab show weekly meeting days (chips), filter occurrences by weekday, preview the next calendar occurrence when no upcoming row exists, and let admins edit the weekly schedule inline.

## Delivery waves

| Wave | Intent | Keys |
| ---- | ------ | ---- |
| 1 | Weekday chips + filter | `ALFQ-HAL-SCH-001`–`004` |
| 2 | Upcoming / history / preview per day | `ALFQ-HAL-SCH-005`–`007` |
| 3 | Inline weekly schedule edit | `ALFQ-HAL-SCH-008`–`010` |
| 4 | Detail tab parity | `ALFQ-HAL-SCH-011`–`012` |
| 5 | Save weekdays retarget | `ALFQ-HAL-SCH-016`–`019` |
| 6 | Weekly date roll | `ALFQ-HAL-SCH-020` |

## Test cases

### Wave 1 — Weekday chips

| ID | Key | Case | Steps | Expected | Requires | Related code |
| --- | --- | --- | --- | --- | --- | --- |
| TC-01 | ALFQ-HAL-SCH-001 | Verify the sessions page shows one chip per configured weekday with times | 1. Sign in as tenant admin.<br>2. Open a halaqa with two weekdays configured (for example Sunday and Tuesday).<br>3. Open "Sessions" / "الجلسات". | 1. Exactly two weekday chips appear.<br>2. Each chip shows that day's start and end times from the halaqa schedule.<br>3. No chip appears for weekdays not in the schedule. | - | `src/features/alforqan/sessions/halaqa-session-day-chips.tsx`, `src/features/alforqan/sessions/halaqa-sessions-schedule-panel.tsx` |
| TC-02 | ALFQ-HAL-SCH-002 | Verify the default selected chip matches the next session weekday | 1. Use the halaqa from TC-01 with one materialized upcoming session.<br>2. Open the sessions list. | 1. The chip for the next session's weekday is selected (highlighted). | ALFQ-HAL-SCH-001 | `src/features/alforqan/sessions/halaqa-sessions-schedule-panel.tsx` |
| TC-03 | ALFQ-HAL-SCH-003 | Verify selecting another weekday filters the list to that day only | 1. On the sessions list, click the weekday chip that is not the next session's day. | 1. The list under "Upcoming for this day" / "القادمة لهذا اليوم" shows only sessions on that weekday.<br>2. Sessions from the other weekday are not listed. | ALFQ-HAL-SCH-001 | `src/features/alforqan/sessions/halaqa-session-weekday.ts`, `src/features/alforqan/sessions/halaqa-sessions-schedule-panel.tsx` |
| TC-04 | ALFQ-HAL-SCH-004 | Verify the next session card stays above the weekday chips | 1. Open the sessions list for a halaqa with an upcoming session. | 1. The "Next session" / "الجلسة التالية" card appears above the weekday chips. | ALFQ-HAL-SCH-001 | `src/features/alforqan/sessions/halaqa-sessions-schedule-panel.tsx` |

### Wave 2 — Upcoming, history, preview

| ID | Key | Case | Steps | Expected | Requires | Related code |
| --- | --- | --- | --- | --- | --- | --- |
| TC-05 | ALFQ-HAL-SCH-005 | Verify the day with the live session shows it in upcoming and the other day shows a preview without Open | 1. Open sessions for a two-day halaqa with only one materialized session.<br>2. Select the chip for the day that has the session.<br>3. Select the other configured day. | 1. On the day with the session, the upcoming list shows the session with an "Open" / "فتح" control.<br>2. On the other day, a preview card shows the next date and time.<br>3. The preview card has no "Open" button. | ALFQ-HAL-SCH-001 | `src/features/alforqan/sessions/preview-next-occurrence.ts`, `src/features/alforqan/sessions/halaqa-sessions-schedule-panel.tsx` |
| TC-06 | ALFQ-HAL-SCH-006 | Verify a completed session on a weekday appears under history | 1. Open a halaqa that has at least one completed session on a configured weekday.<br>2. Select that weekday chip. | 1. The completed session appears under "History for this day" / "سجل هذا اليوم".<br>2. It does not appear in the upcoming section. | ALFQ-HAL-SCH-001 | `src/features/alforqan/sessions/halaqa-sessions-schedule-panel.tsx` |
| TC-07 | ALFQ-HAL-SCH-007 | Verify the preview time matches the schedule for that weekday | 1. Select a weekday chip with no upcoming session.<br>2. Read the preview card. | 1. The preview shows start and end times matching that weekday row in the halaqa schedule. | ALFQ-HAL-SCH-001 | `src/features/alforqan/sessions/preview-next-occurrence.ts` |

### Wave 3 — Inline schedule edit

| ID | Key | Case | Steps | Expected | Requires | Related code |
| --- | --- | --- | --- | --- | --- | --- |
| TC-08 | ALFQ-HAL-SCH-008 | Verify admin can change a weekday time from the sessions page | 1. Sign in as tenant admin with UpdateHalaqa.<br>2. Open sessions list.<br>3. Expand "Weekly schedule" / "الجدول الأسبوعي".<br>4. Change one day's end time.<br>5. Save.<br>6. Open halaqa Edit. | 1. Success toast appears.<br>2. The chip for that day shows the new end time.<br>3. Halaqa Edit shows the same schedule times. | ALFQ-HAL-SCH-001 | `src/features/alforqan/sessions/halaqa-schedule-inline-editor.tsx` |
| TC-09 | ALFQ-HAL-SCH-009 | Verify at least one weekday is required when saving inline schedule | 1. As admin, expand weekly schedule.<br>2. Uncheck all weekdays.<br>3. Save. | 1. Save is blocked or shows validation that at least one weekday is required. | ALFQ-HAL-SCH-001 | `src/features/alforqan/sessions/halaqa-schedule-inline-editor.tsx` |
| TC-10 | ALFQ-HAL-SCH-010 | Verify teacher without UpdateHalaqa does not see the inline editor | 1. Sign in as the assigned teacher without UpdateHalaqa claim.<br>2. Open the sessions list. | 1. Weekday chips are visible.<br>2. "Weekly schedule" expand control is not shown. | ALFQ-HAL-SCH-001 | `src/features/alforqan/sessions/halaqa-schedule-inline-editor.tsx` |

### Wave 4 — Detail tab

| ID | Key | Case | Steps | Expected | Requires | Related code |
| --- | --- | --- | --- | --- | --- | --- |
| TC-11 | ALFQ-HAL-SCH-011 | Verify halaqa detail Sessions tab shows the same weekday chips as the full sessions page | 1. Open halaqa detail.<br>2. Open the "Sessions" tab.<br>3. Compare chips to the full sessions list for the same halaqa. | 1. The same weekday chips and times appear on both surfaces. | ALFQ-HAL-SCH-001 | `src/app/[locale]/(tenant)/[tenantSlug]/halaqat/[id]/halaqa-detail-content.tsx` |
| TC-12 | ALFQ-HAL-SCH-012 | Verify admin can save weekly schedule from the detail Sessions tab | 1. On halaqa detail Sessions tab as admin, expand weekly schedule.<br>2. Add or remove a weekday and save. | 1. Success toast appears.<br>2. Chips update without opening halaqa Edit. | ALFQ-HAL-SCH-008 | `src/app/[locale]/(tenant)/[tenantSlug]/halaqat/[id]/halaqa-detail-content.tsx`, `src/features/alforqan/sessions/halaqa-schedule-inline-editor.tsx` |

### Wave 5 — Save weekdays retarget

| ID | Key | Case | Steps | Expected | Requires | Related code |
| --- | --- | --- | --- | --- | --- | --- |
| TC-13 | ALFQ-HAL-SCH-016 | Verify saving different weekdays updates the chips to those days | 1. Sign in as tenant admin with UpdateHalaqa.<br>2. Open sessions for a halaqa that currently has two weekdays.<br>3. Expand "Weekly schedule" / "الجدول الأسبوعي".<br>4. Uncheck one existing day, check a new day, and save. | 1. Success toast appears.<br>2. Chips match the saved weekdays.<br>3. The removed weekday chip is gone. | ALFQ-HAL-SCH-008 | `src/features/alforqan/sessions/halaqa-schedule-inline-editor.tsx`, `src/features/alforqan/sessions/halaqa-session-day-chips.tsx` |
| TC-14 | ALFQ-HAL-SCH-017 | Verify each newly saved weekday has an upcoming session | 1. After TC-13, select each remaining weekday chip. | 1. Each saved weekday shows an upcoming session or a next-date preview for that day. | ALFQ-HAL-SCH-016 | `src/features/alforqan/sessions/halaqa-sessions-schedule-panel.tsx` |
| TC-15 | ALFQ-HAL-SCH-018 | Verify a removed weekday no longer lists an upcoming not-started session | 1. After TC-13, confirm the removed weekday chip is absent. | 1. No chip exists for the removed weekday. | ALFQ-HAL-SCH-016 | `src/features/alforqan/sessions/halaqa-session-day-chips.tsx` |
| TC-16 | ALFQ-HAL-SCH-019 | Verify a completed session keeps its original date after a weekday save | 1. Use a halaqa with a completed session on a weekday that stays in the schedule.<br>2. Change another weekday and save. | 1. The completed session stays under "History for this day" / "سجل هذا اليوم" on the original date. | ALFQ-HAL-SCH-016 | `src/features/alforqan/sessions/halaqa-sessions-schedule-panel.tsx` |

### Wave 6 — Weekly date roll

| ID | Key | Case | Steps | Expected | Requires | Related code |
| --- | --- | --- | --- | --- | --- | --- |
| TC-17 | ALFQ-HAL-SCH-020 | Verify passed not-started sessions show the next week's dates | 1. Use a two-weekday halaqa whose last Scheduled dates are more than a week in the past (for example 12 Oct when today is 25 Oct).<br>2. Open the halaqa sessions page.<br>3. Select each weekday chip. | 1. Upcoming sessions show the next future date for each weekday, not the old 12 Oct dates.<br>2. Both selected weekdays have a current upcoming date. | ALFQ-HAL-SCH-001 | `src/features/alforqan/sessions/halaqa-sessions-schedule-panel.tsx` |
