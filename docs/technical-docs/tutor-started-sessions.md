# Tutor-started sessions and late daily revision

Replaces the start-window model (`session-start-window`). Migration: `20261117120000_TutorStartedSessionsAndLateRevision`.

## Why

Sessions used to be generated from the weekly schedule, and Start was allowed only from 15 minutes before
the slot until its end. When the window passed, the row rolled forward a week, so tutors who were late
could not start a session and saw it disappear. Students had no "live now" view.

## Sessions

- The weekly schedule (`HalaqaSchedule`) is a **reminder only**. No session rows are generated from it
  (`SessionOccurrenceMaterializer` and `SessionOccurrencePlanner` are removed).
- `POST HalaqaSessions/halaqa/{halaqaId}/start-now` (teacher of the halaqa or `ManageHalaqaSessionsClaim`):
  - Returns the halaqa's in-progress session if it started within the last 6 hours (double click, second device).
  - Otherwise creates a session with `Status = InProgress` and `ScheduledAt = StartedAt = now`.
    `OccurrenceLocalDate` is null, and the duration comes from today's slot (60 minutes without one).
  - The meeting link comes from the body. When it is empty, the halaqa's last used link is reused.
- `PUT HalaqaSessions/{id}/start` (legacy rows) has no time check. Scheduled or LinkReady rows can be started at any time.
- `PUT HalaqaSessions/{id}/link` works on Scheduled, LinkReady and InProgress sessions. Participants are
  notified only when the link changes.
- Restore of a cancelled session has no time limit.
- Start effects (`HalaqaSessionStartEffects`, run by both start endpoints):
  1. Open homework of the halaqa whose target (`SessionId`) is empty or not started now targets the
     new session, and `DueDate = now`. Homework given in this session is left alone.
  2. Students and parents get `TenantNotificationType.SessionStarted` (11), referencing the session.
- `GET HalaqaSessions/mine/halaqat` is the tutor board. For each own active halaqa it returns the live
  session id, the next scheduled start (reminder text) and the last meeting link.
- `GET HalaqaSessions/live` returns in-progress sessions (started in the last 6 hours) of the signed-in
  student's or parent's halaqat.
- `GET HalaqaSessions/mine` lists in-progress and not-started sessions always, and finished ones for 8 days.

### Reminder job

`SessionWindowBackgroundService` runs every 60 seconds. For each active halaqa with a time zone and a schedule:

- **When it fires:** halaqa-local time is between `start − StartEarlyMinutes` and the slot end, no session is
  live, and `Halaqa.LastSessionReminderLocalDate` is not today.
- **What it does:** sends an `AppNotification` (`SessionStartingSoon`, reference `Halaqa`) to the tutor and the
  SignalR event `SessionWindowOpened`, whose payload is now the halaqa id.
- **After firing:** it stamps `LastSessionReminderLocalDate`.

It also expires deferred attendance. Production runs in `SingleTenant` mode, so the default tenant
connection used by the job is the right database.

### Migration cleanup

The migration does the following for session rows with `CreatedByUserId = 'system'`, status Scheduled or
LinkReady, no `StartedAt`, and no attendance or evaluations:

- Soft-deletes them.
- Clears `SessionId` and `DueDate` on open homework that pointed at them. That homework is attached when
  the tutor next presses Start.

## Daily revision deadline

- **Window:** the window covers halaqa-local days after the assigning session (`StartedAt ?? ScheduledAt`).
  - **Closed:** once the target session has a `StartedAt`, the window runs until the day before the date of that `StartedAt`.
  - **Open:** while no target session has started, the window runs through the student's today. It never rolls or
    disappears.
- **Legacy homework:** homework that is no longer open and has no started target closes at the target's
  planned time, then `DueDate`, then the assignment time (nothing owed).
- **Submitting:** any owed day up to today with nothing sent can be submitted (catch-up). Retries follow
  the existing review rules.
- **Late:** a submission made at or after the target session's `StartedAt` is stored with
  `StudentRevisionDailyLog.IsLate = true`.
- **Late is never counted.** Everywhere it would count, a late part is treated as not sent:
  - Day status in the student report (a day with only late parts is Missing or Due, with `IsLate`).
  - `IsHomeworkFullyCovered`.
  - The staff revision report: day-mode sent, not-sent and pending counts, and all range-mode totals. Day mode still
    lists late parts, flagged.
  - The session evaluation PDF (`HasLog` is false for late-only days).
- Daily revision no longer requires the halaqa to have a schedule (`HalaqaScheduleRequired` is not returned).
- Points and perks: daily revision earns none today. Points come only from Sard decisions. Any future
  reward must skip `IsLate` parts.

## Frontend

- **My sessions (tutor):** a halaqa board with Start session (optional link dialog) or Continue. Older
  sessions still in progress are listed so they can be completed.
- **Halaqa → Sessions:** a Start session button when nothing is live.
- **Students and parents:** a floating "Your halaqa is live" card with Join, shown while a session is live.
  It refreshes on `NewNotification` and every 30 seconds.
- **Daily revision panel:** shows the deadline hint before the deadline and a past-deadline notice after it.
  Each past owed day has a Send button (Send late after the deadline). Late parts show a "Late – not
  counted" badge.
