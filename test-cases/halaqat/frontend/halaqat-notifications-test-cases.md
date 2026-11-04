# Tenant notifications — Frontend Test Cases


## Metadata
- Project: `alforqan`
- Feature: `halaqat`
- Layer: `frontend`
- Routes: `Any authenticated tenant layout (bell fixed at top of screen)`
- APIs: -
- Related code: `src/features/alforqan/components/notifications/tenant-notification-bell.tsx`, `src/hooks/use-tenant-notifications-signalr.ts`, `src/app/[locale]/(tenant)/[tenantSlug]/sessions/[sessionId]/halaqa-session-participant-content.tsx`
- Related docs: `docs/explanation/auth/permissions-and-signalr-explained.md`, `docs/technical/auth/permissions-and-signalr-guide.md`, `docs/technical/tenant/halaqat-guide.md`

## Scope

- In-app notification bell, unread badge, list panel, mark read, deep link to session evaluation report, and deep link to student/parent session view when the meeting link is ready.

## Test pages

- Any authenticated tenant layout (bell fixed at top of screen)

## Test scenarios

- Unread count updates after new notification (real-time or after refresh).
- Open panel, mark one read and mark all read.

## Delivery waves

| Wave | Intent | Keys (representative) | Depends on (cross-file) |
| ---- | ------ | --------------------- | ----------------------- |
| 1 | Bell and list — visibility, panel, mark all read | `ALFQ-HAL-NOT-001`–`003` | — |
| 2 | Deep links — evaluation report and meeting link session view | `ALFQ-HAL-NOT-004`–`005` | `ALFQ-HAL-EVA-011` on TC-04; `ALFQ-HAL-SES-004` on TC-05 |

Wave 2 is independent of Wave 1 in this file. Run evaluation deep link after evaluation submit; run meeting-link deep link after a teacher saves a meeting link (`ALFQ-HAL-SES-004`).

## Test cases

### Wave 1 — Bell and list

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-01 | ALFQ-HAL-NOT-001 | Verify notification bell is visible after sign-in | Main | User is signed in to the tenant app | 1. Open any main tenant screen (for example dashboard). | A bell control appears in the top area of the screen. | - | `src/features/alforqan/components/notifications/tenant-notification-bell.tsx` | - |
| TC-02 | ALFQ-HAL-NOT-002 | Verify opening the panel shows title "Notifications" | Main | At least zero notifications | 1. Choose the bell. | A panel or sheet opens with title "Notifications" in English or the Arabic equivalent. | - | `src/features/alforqan/components/notifications/tenant-notification-bell.tsx` | - |
| TC-03 | ALFQ-HAL-NOT-003 | Verify mark all read clears the badge | Main | Unread count greater than zero | 1. Open the panel.<br>2. Choose "Mark all read". | 1. The red badge on the bell disappears or shows zero<br>2. items appear as read. | - | `src/features/alforqan/components/notifications/tenant-notification-bell.tsx`, `src/hooks/use-tenant-notifications-signalr.ts` | - |

### Wave 2 — Deep link

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-04 | ALFQ-HAL-NOT-004 | Verify tapping an evaluation notification opens the report page | Main | A notification exists whose title includes "Evaluation passed", "Evaluation not passed", or the Arabic equivalent | 1. Open the panel.<br>2. Tap that notification row. | 1. The app navigates to the evaluation report page<br>2. the page title includes "Session evaluation" or the Arabic equivalent. | ALFQ-HAL-EVA-011 | `src/features/alforqan/components/notifications/tenant-notification-bell.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/reports/evaluation/[evaluationId]/session-evaluation-report-content.tsx` | Notification created by submitting an evaluation as a teacher |
| TC-05 | ALFQ-HAL-NOT-005 | Verify tapping a meeting link ready notification opens the session view for a student or parent | Main | Signed in as a student enrolled in the halaqa, or as a parent linked to such a student; a notification exists whose title is "Meeting link ready" or "رابط الاجتماع جاهز" after the teacher saved a link | 1. Open the panel.<br>2. Tap that notification row. | 1. The notifications panel closes.<br>2. The page title shows "Your session" or "جلستك".<br>3. A "Join meeting" (or "انضم للاجتماع") control appears and opens the saved meeting URL. | ALFQ-HAL-SES-004 | `src/features/alforqan/components/notifications/tenant-notification-bell.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/sessions/[sessionId]/halaqa-session-participant-content.tsx` | Student or linked parent account |
