# Session start window — Frontend Test Cases

## Metadata
- Project: `alforqan`
- Feature: `halaqat`
- Layer: `frontend`
- Routes: `/halaqat/create`, `/halaqat/{id}/sessions`, `/my-sessions`
- APIs: `-`
- Related code: `src/features/alforqan/halaqat/halaqa-schedule-fields.tsx`
- Related docs: `Alforqan-Backend/docs/technical-docs/session-start-window.md`
- Automation: `-`

## Scope

Admin sets early/late minutes. Sessions page has no create form. Teacher sees Start inside the window, a missed label, a bottom-left button, and a my-sessions page.

## Delivery waves

| Wave | Intent | Keys |
| ---- | ------ | ---- |
| 2 | Form minutes | ALFQ-SESWIN-009 |
| 3 | No create form | ALFQ-SESWIN-014 |
| 4 | Start button | ALFQ-SESWIN-019 |
| 7 | Next card and missed | ALFQ-SESWIN-026–028 |
| 8 | Launcher | ALFQ-SESWIN-029–032 |

## Test cases

### Wave 2 — Form

| ID | Key | Case | Steps | Expected | Related code |
| --- | --- | --- | --- | --- | --- |
| TC-09 | ALFQ-SESWIN-009 | Verify the halaqa form shows early and late minutes and blocks 181 | 1. Open halaqa create as admin.<br>2. Read the two minute fields.<br>3. Enter 181 in early minutes.<br>4. Save. | 1. "Minutes before start" / "دقائق قبل البداية" is visible.<br>2. "Minutes after end" / "دقائق بعد النهاية" is visible.<br>3. Save is blocked. | `src/features/alforqan/halaqat/halaqa-schedule-fields.tsx` |

### Wave 3 — Sessions list

| ID | Key | Case | Steps | Expected | Related code |
| --- | --- | --- | --- | --- | --- |
| TC-14 | ALFQ-SESWIN-014 | Verify the sessions page has no new-session form | 1. Open a halaqa sessions page as teacher. | 1. "Add session" / "إضافة جلسة" is not shown.<br>2. The create session form is not shown. | `src/app/[locale]/(tenant)/[tenantSlug]/halaqat/[id]/sessions/halaqa-sessions-list-content.tsx` |

### Wave 4 — Start

| ID | Key | Case | Steps | Expected | Related code |
| --- | --- | --- | --- | --- | --- |
| TC-19 | ALFQ-SESWIN-019 | Verify Start is enabled only inside the window | 1. Open a scheduled session outside the window.<br>2. Open the same session inside the window. | 1. Start is not enabled outside the window.<br>2. Start is enabled inside the window. | `src/app/[locale]/(tenant)/[tenantSlug]/halaqat/[id]/sessions/[sessionId]/halaqa-session-detail-content.tsx` |

### Wave 7 — Next session card

| ID | Key | Case | Steps | Expected | Related code |
| --- | --- | --- | --- | --- | --- |
| TC-26 | ALFQ-SESWIN-026 | Verify the next session shows its schedule time | 1. Open the halaqa sessions page with a scheduled next session. | 1. The "Next session" / "الجلسة التالية" card shows the schedule time.<br>2. The create form is not shown. | `src/app/[locale]/(tenant)/[tenantSlug]/halaqat/[id]/sessions/halaqa-sessions-list-content.tsx` |
| TC-27 | ALFQ-SESWIN-027 | Verify past sessions remain on the page | 1. Open the halaqa sessions page that has a completed session. | The completed session stays in the list. | `src/app/[locale]/(tenant)/[tenantSlug]/halaqat/[id]/sessions/halaqa-sessions-list-content.tsx` |
| TC-28 | ALFQ-SESWIN-028 | Verify a missed row shows the missed label and no enabled Start | 1. Open a session whose window has ended and status is still Scheduled. | 1. "Missed" / "فائتة" is shown.<br>2. Start is not enabled. | `src/app/[locale]/(tenant)/[tenantSlug]/halaqat/[id]/sessions/halaqa-sessions-list-content.tsx` |

### Wave 8 — Launcher

| ID | Key | Case | Steps | Expected | Related code |
| --- | --- | --- | --- | --- | --- |
| TC-29 | ALFQ-SESWIN-029 | Verify the teacher sees the bottom-left button when he has a session | 1. Log in as the halaqa teacher with a next session.<br>2. Open any tenant page. | The bottom-left "Sessions" / "الجلسات" button is visible. | `src/features/alforqan/sessions/teacher-session-launcher-button.tsx` |
| TC-30 | ALFQ-SESWIN-030 | Verify a user who is not that teacher does not see the button | 1. Log in as a parent or another teacher with an empty mine list.<br>2. Open any tenant page. | The bottom-left sessions button is not shown. | `src/features/alforqan/sessions/teacher-session-launcher-button.tsx` |
| TC-31 | ALFQ-SESWIN-031 | Verify My sessions lists his sessions and Start is enabled only on the open one | 1. Open `/my-sessions` as the teacher.<br>2. Read Start on the open row and on a later row. | 1. Both sessions are listed.<br>2. Start is enabled only on the open row. | `src/features/alforqan/sessions/teacher-my-sessions-content.tsx` |
| TC-32 | ALFQ-SESWIN-032 | Verify a popup appears when the window opens while he is online | 1. Stay on a tenant page as the teacher.<br>2. Wait until the session window opens. | 1. One popup says the session window is open.<br>2. The bottom-left badge updates without a manual refresh. | `src/hooks/use-tenant-notifications-signalr.ts` |
