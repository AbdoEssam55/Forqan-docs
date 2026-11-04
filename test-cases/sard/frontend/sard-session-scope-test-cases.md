# Sard session start scope — Frontend Test Cases

## Metadata
- Project: `alforqan`
- Feature: `sard`
- Layer: `frontend`
- Routes: `/{locale}/{tenantSlug}/sard`
- APIs: `POST /api/.../Sard/sessions`
- Related code: `src/app/[locale]/(tenant)/[tenantSlug]/sard/sard-sessions-content.tsx`, `src/features/sard/sard-session-scope-start-fields.tsx`
- Related docs: `_See the new sard session dialog start fields._`
- Automation: `-`

## Scope
- Per new sard session, for the chosen student, a Manage-sard admin can check or uncheck "Respect checkpoints for this sard start" (default on). Not a tenant setting.
- End is always that student's pending checkpoint.

## Test pages
- `/{tenantSlug}/sard`

## Test scenarios
- Admin leaves the checkbox on and may start after any earlier checkpoint (or mushaf start) for this student.
- Admin unchecks it for this session and may pick any surah/ayah before the pending checkpoint.
- Changing student or opening a new dialog resets the checkbox on.
- Committee teachers without Manage sard cannot create sessions.

## Delivery waves
- Wave 1 — Per-session checkbox (`ALFQ-SARD-PLAN-008`–`009`)
- Wave 2 — Create-session start (`ALFQ-SARD-SES-008`–`012`)

## Test cases

### Wave 1 — Per-session respect-checkpoints checkbox

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-01 | ALFQ-SARD-PLAN-008 | Verify "Respect checkpoints for this sard start" is on by default for a new session | Main | Logged in with Manage sard; student has a pending checkpoint | 1. Open "Sard".<br>2. Click "New sard session".<br>3. Pick that student.<br>4. Find the checkbox labeled "Respect checkpoints for this sard start". | 1. The checkbox is on.<br>2. "Start after checkpoint" is shown (not "From surah" / "From ayah").<br>3. The hint says this applies only to this session. | ALFQ-SARD-SES-003 | `src/features/sard/sard-session-scope-start-fields.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/sard/sard-sessions-content.tsx` | - |
| TC-02 | ALFQ-SARD-PLAN-009 | Verify unchecking respect checkpoints applies only to this session | Main | Same as TC-01 | 1. Open "New sard session" and pick a student.<br>2. Uncheck "Respect checkpoints for this sard start".<br>3. Confirm "From surah" and "From ayah" appear.<br>4. Pick a different student, or close the dialog and open "New sard session" again for the first student. | 1. After uncheck, start fields are surah and ayah.<br>2. After changing student or opening a new dialog, the checkbox is on again.<br>3. The sard plan settings page has no "Respect checkpoints" switch. | ALFQ-SARD-PLAN-008 | `src/features/sard/sard-session-scope-start-fields.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/sard/sard-sessions-content.tsx` | - |

### Wave 2 — New sard session start

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-03 | ALFQ-SARD-SES-008 | Verify the default start is the previous checkpoint and the end is the pending one | Main | Respect checkpoints is on; student pending is a later checkpoint (e.g. "2 جزء" after "1 جزء" passed); no open sard session | 1. Open "Sard" as a user with Manage sard.<br>2. Click "New sard session".<br>3. Pick that student. | 1. "Until checkpoint" shows the pending checkpoint label.<br>2. "Start after checkpoint" defaults to after the previous checkpoint (not mushaf start).<br>3. The scope line shows previous-boundary → pending boundary. | ALFQ-SARD-SES-003, ALFQ-SARD-PLAN-008 | `src/app/[locale]/(tenant)/[tenantSlug]/sard/sard-sessions-content.tsx`, `src/features/sard/sard-session-scope-start-fields.tsx`, `src/features/sard/sard-scope-utils.ts` | Pending later than first checkpoint |
| TC-04 | ALFQ-SARD-SES-009 | Verify the admin can start after an earlier checkpoint | Main | Same student as TC-03; respect checkpoints on | 1. Open "New sard session" and pick the student.<br>2. In "Start after checkpoint", pick an earlier checkpoint (not the immediate previous one).<br>3. Pick date, time, and a committee teacher.<br>4. Click "New sard session". | 1. The scope line widens from after that earlier checkpoint through the pending checkpoint.<br>2. The created session is "Scheduled" with that wider range (end still the pending checkpoint). | ALFQ-SARD-SES-008 | `src/app/[locale]/(tenant)/[tenantSlug]/sard/sard-sessions-content.tsx`, `src/features/sard/sard-session-scope-start-fields.tsx` | Earlier checkpoint selected |
| TC-05 | ALFQ-SARD-SES-010 | Verify the admin can start from the mushaf start | Main | Same student as TC-03; respect checkpoints on | 1. Open "New sard session" and pick the student.<br>2. Choose "From the start of the Mushaf".<br>3. Create the session with a teacher and time. | 1. The scope line starts at the mushaf start for that hifz direction.<br>2. The end stays the pending checkpoint. | ALFQ-SARD-SES-008 | `src/app/[locale]/(tenant)/[tenantSlug]/sard/sard-sessions-content.tsx`, `src/features/sard/sard-session-scope-start-fields.tsx` | - |
| TC-06 | ALFQ-SARD-SES-011 | Verify any earlier surah and ayah can be chosen when respect checkpoints is off | Main | Student has a later pending checkpoint; no open session | 1. Open "New sard session" and pick the student.<br>2. Uncheck "Respect checkpoints for this sard start".<br>3. Confirm surah and ayah start fields are shown (not a checkpoint list).<br>4. Pick a surah and ayah before the pending checkpoint (not a checkpoint boundary).<br>5. Create the session. | 1. "From surah" and "From ayah" are shown.<br>2. The scope uses the chosen start and still ends at the pending checkpoint.<br>3. The session is created. | ALFQ-SARD-PLAN-009, ALFQ-SARD-SES-008 | `src/features/sard/sard-session-scope-start-fields.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/sard/sard-sessions-content.tsx` | Respect checkpoints off for this session |
| TC-07 | ALFQ-SARD-SES-012 | Verify a committee teacher without Manage sard cannot create a session | Edge | Logged in as a committee teacher who does not have Manage sard | 1. Open the "Sard" area. | 1. The "New sard session" button is not shown.<br>2. The list shows only sessions where this teacher appears in the session "Teachers" line. | ALFQ-SARD-SES-002 | `src/app/[locale]/(tenant)/[tenantSlug]/sard/sard-sessions-content.tsx` | Manage sard off |
| TC-08 | ALFQ-SARD-SES-013 | Verify a cert teacher does not see other cert teachers' sessions | Main | Tenant has sard sessions for students assigned to teachers Ahmed and Muhtazah; Muhtazah is logged in and is on the sard committee | 1. Log in as Muhtazah.<br>2. Open "Sard".<br>3. Read each row's "Teachers" line. | 1. Every listed session includes Muhtazah in "Teachers".<br>2. Sessions where only Ahmed (or Omar) is the assigned teacher are not listed. | ALFQ-SARD-SES-012 | `src/app/[locale]/(tenant)/[tenantSlug]/sard/sard-sessions-content.tsx` | Sessions exist for multiple committee teachers |
| TC-09 | ALFQ-SARD-SES-014 | Verify Manage sard still sees every sard session | Main | Logged in as tenant admin with Manage sard; same data as TC-08 | 1. Open "Sard". | 1. Sessions for all students and all assigned teachers appear in one list. | ALFQ-SARD-SES-013 | `src/app/[locale]/(tenant)/[tenantSlug]/sard/sard-sessions-content.tsx` | Manage sard on |
