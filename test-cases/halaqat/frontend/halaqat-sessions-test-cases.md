# Halaqa sessions — Frontend Test Cases


## Metadata
- Project: `alforqan`
- Feature: `halaqat`
- Layer: `frontend`
- Routes: ``/{locale}/{tenantSlug}/halaqat/{halaqaId}/sessions``, ``/{locale}/{tenantSlug}/halaqat/{halaqaId}/sessions/{sessionId}``
- APIs: `GET .../HalaqaSessions/{sessionId}/evaluations/summary`
- Related code: `src/app/[locale]/(tenant)/[tenantSlug]/halaqat/[id]/sessions/halaqa-sessions-list-content.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/halaqat/[id]/sessions/[sessionId]/halaqa-session-detail-content.tsx`
- Related docs: `docs/technical/tenant/halaqat-guide.md`, `docs/explanation/tenant/halaqat-explained.md`

## Scope

- Scheduling halaqa sessions, meeting links, and session lifecycle actions from the tenant app.

## Test pages

- `/{locale}/{tenantSlug}/halaqat/{halaqaId}/sessions`
- `/{locale}/{tenantSlug}/halaqat/{halaqaId}/sessions/{sessionId}`

## Test scenarios

- Create a session and see it in the list.
- Open session detail, save meeting link, start and end session, cancel when allowed.
- Navigate from halaqa detail to the sessions list.
- Confirm session status badges show localized labels.

## Delivery waves

| Wave | Intent | Keys (representative) | Depends on (cross-file) |
| ---- | ------ | --------------------- | ----------------------- |
| 1 | Sessions list — open list, schedule, open detail | `ALFQ-HAL-SES-001`–`003` | — |
| 2 | Session detail and lifecycle — link, start/end, cancel, evaluation badges | `ALFQ-HAL-SES-004`–`007` | Wave 1 (`ALFQ-HAL-SES-003` for lifecycle) |
| 3 | Auto-generated bilingual titles — locale display, legacy fallback | `ALFQ-HAL-SES-008`–`010` | `ALFQ-HAL-SES-002` |
| 4 | Same-day second session shows time in title | `ALFQ-HAL-SES-011` | `ALFQ-HAL-SES-002` |

Run this file before [halaqat-attendance-test-cases.md](halaqat-attendance-test-cases.md). Use `tc blockers --wave 1` or Session **Wave** filter when testing one chunk.

## Test cases

### Wave 1 — Sessions list

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-01 | ALFQ-HAL-SES-001 | Verify an authorized user can open the sessions list for a halaqa | Main | User is signed in with permission to view halaqat; a halaqa exists | 1. Open the halaqa detail page.<br>2. Choose the control that opens "Sessions". | 1. The page title shows "Sessions" (English) or the Arabic equivalent<br>2. scheduled sessions or an empty state message appears. | - | `src/app/[locale]/(tenant)/[tenantSlug]/halaqat/[id]/sessions/halaqa-sessions-list-content.tsx` | Tenant: any test tenant; Halaqa: active halaqa |
| TC-02 | ALFQ-HAL-SES-002 | Verify scheduling a new session adds a row with auto-generated title, time, and description | Main | Logged in as the assigned halaqa teacher; user is on the sessions list | 1. Open "Add session".<br>2. Confirm there is no title input; optional description "Focus on juz 5".<br>3. Open the "Date and time" field and use the branded calendar popover to choose a future date (for example "2026-06-10") and set the time to "10:00"; past dates must be disabled.<br>4. Submit the form. | 1. A success toast appears.<br>2. The create dialog shows the generated-title hint (no title field).<br>3. The new list card shows an auto-generated title containing the halaqa name, local date segments, and weekday in the current UI locale (English weekday in `/en`, Arabic weekday in `/ar`).<br>4. Description "Focus on juz 5" and local scheduled time appear.<br>5. Status shows "Scheduled". | ALFQ-HAL-SES-001 | `src/app/[locale]/(tenant)/[tenantSlug]/halaqat/[id]/sessions/halaqa-sessions-list-content.tsx`, `src/features/alforqan/components/sessions/halaqa-session-form.tsx`, `src/features/alforqan/sessions/session-display-title.ts` | Description: "Focus on juz 5"; future date such as "2026-06-10" |
| TC-03 | ALFQ-HAL-SES-003 | Verify opening a session shows detail and status badge | Main | At least one session exists | 1. From the list, choose "Open". | 1. The session detail page loads with the session title and description visible<br>2. a status badge is visible, such as "Scheduled" in English or "مجدولة" in Arabic. The badge must not show "tenant.halaqaSessions". | ALFQ-HAL-SES-002 | `src/app/[locale]/(tenant)/[tenantSlug]/halaqat/[id]/sessions/[sessionId]/halaqa-session-detail-content.tsx` | Session id from list |

### Wave 2 — Session detail and lifecycle

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-04 | ALFQ-HAL-SES-004 | Verify saving a meeting link updates the page | Main | Logged in as the assigned halaqa teacher; session status is before completed | 1. Enter URL "https://example.com/meet".<br>2. Choose platform "Zoom".<br>3. Save the link. | 1. A success toast appears<br>2. the link text "https://example.com/meet" appears on the page. | ALFQ-HAL-SES-003 | `src/app/[locale]/(tenant)/[tenantSlug]/halaqat/[id]/sessions/[sessionId]/halaqa-session-detail-content.tsx`, `src/features/alforqan/halaqat/use-is-halaqa-teacher.ts` | Platform: "zoom" |
| TC-05 | ALFQ-HAL-SES-005 | Verify start and end session actions refresh status | Main | Logged in as the assigned halaqa teacher; session is in "Link ready" then can be started per product rules | 1. When "Start session" is shown, choose it and confirm in the branded dialog (for example "Yes, start session").<br>2. Mark every student present, absent, late, or excused and choose "Save attendance" / "حفظ الحضور".<br>3. When "End session" is shown, choose it and confirm in the dialog (for example "Yes, end session"). | 1. Each action shows a confirmation dialog before running.<br>2. After step 1, status shows "In progress".<br>3. After step 3, status shows "Completed". Choosing "Not now" closes the dialog without changing status. | ALFQ-HAL-SES-003 | `src/features/alforqan/components/sessions/session-lifecycle-confirm-button.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/halaqat/[id]/sessions/[sessionId]/halaqa-session-detail-content.tsx` | - |
| TC-06 | ALFQ-HAL-SES-006 | Verify cancel session marks session cancelled | Edge | Logged in as the assigned halaqa teacher; session not already completed | 1. Choose "Cancel session".<br>2. In the confirmation dialog, choose "Yes, cancel session". | 1. A branded confirmation dialog appears before cancel runs<br>2. status shows "Cancelled"<br>3. attendance save is no longer available for editing. Choosing "Not now" leaves the session unchanged. | ALFQ-HAL-SES-003 | `src/features/alforqan/components/sessions/session-lifecycle-confirm-button.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/halaqat/[id]/sessions/[sessionId]/halaqa-session-detail-content.tsx` | - |
| TC-07 | ALFQ-HAL-SES-007 | Verify evaluation status badges on session detail student rows | Main | Session detail open; at least one student with a submitted evaluation and one with outcome recorded | 1. Open the session detail page.<br>2. Find a student who submitted evaluation but has no outcome yet.<br>3. Find a student whose outcome was recorded (parent notified). | 1. Student with submit only shows badges "Evaluation submitted" and "Awaiting session result" (English) or Arabic equivalents.<br>2. Student with outcome recorded shows "Evaluation submitted" and "Sent to parent".<br>3. Student never evaluated shows no evaluation badges. | ALFQ-HAL-SES-003, ALFQ-HAL-EVA-011 | `src/features/alforqan/components/sessions/session-evaluation-status-badges.tsx` | Use students from evaluation flow |

### Wave 3 — Auto-generated titles

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-08 | ALFQ-HAL-SES-008 | Verify Arabic UI shows Arabic weekday in auto-generated session title | Main | Logged in as teacher; `/ar` locale; halaqa with a known Arabic name | 1. Schedule a new session on a known future Saturday (for example 2026-11-14) at 10:00.<br>2. Read the list card title. | 1. Title contains the halaqa name.<br>2. Title contains "السبت" (not "Saturday").<br>3. Title contains date segments "14 / 11 / 2026". | ALFQ-HAL-SES-002 | `src/features/alforqan/sessions/session-display-title.ts`, `src/app/[locale]/(tenant)/[tenantSlug]/halaqat/[id]/sessions/halaqa-sessions-list-content.tsx` | Saturday 2026-11-14 |
| TC-09 | ALFQ-HAL-SES-009 | Verify English UI shows English weekday in auto-generated session title | Main | Same halaqa; switch to `/en` locale | 1. Open sessions list for the session created in TC-08 (or create another on the same date).<br>2. Read the list card title. | 1. Title contains "Saturday" (not "السبت").<br>2. Title contains "14 / 11 / 2026". | ALFQ-HAL-SES-002 | `src/features/alforqan/sessions/session-display-title.ts` | Session from TC-08 |
| TC-10 | ALFQ-HAL-SES-010 | Verify legacy session without bilingual titles still displays stored title | Main | An older session exists with manual `title` only (no `titleAr`/`titleEn` from API) | 1. Open sessions list in either locale.<br>2. Find the legacy session row. | 1. The row shows the original stored title text unchanged.<br>2. No empty title or raw API field names appear. | ALFQ-HAL-SES-001 | `src/features/alforqan/sessions/session-display-title.ts` | Pre-migration session |

### Wave 4 — Same-day time suffix

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-11 | ALFQ-HAL-SES-011 | Verify second session on the same local day appends time to the title | Main | Teacher on sessions list; no other session that day yet | 1. Create first session on a future day at 10:00.<br>2. Create second session same calendar day at 16:00.<br>3. Compare list titles. | 1. First session title has no " / HH:mm" suffix.<br>2. Second session title ends with " / 16:00" (local time).<br>3. Both titles share the same date and weekday portion. | ALFQ-HAL-SES-002 | `src/app/[locale]/(tenant)/[tenantSlug]/halaqat/[id]/sessions/halaqa-sessions-list-content.tsx` | Same local day; two times |
