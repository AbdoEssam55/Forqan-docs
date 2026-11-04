# Sard teacher effort & reports — Frontend Test Cases

## Metadata

- Project: `alforqan`
- Feature: `sard`
- Layer: `frontend`
- Routes: ``/{locale}/{tenantSlug}/reports/sard``, ``/{locale}/{tenantSlug}/sard``, ``/{locale}/{tenantSlug}/sard/{sardSessionId}/evaluate``, ``/{locale}/{tenantSlug}/teachers/{id}``
- APIs: `PUT/GET /api/.../Sard/sessions/{id}/assignments`, `POST/GET /api/.../Sard/sessions/{id}/daily-progress`, `GET/POST /api/.../SardReports/*`, `GET /api/.../Teachers/{id}/sard-stats`
- Related code: `src/app/[locale]/(tenant)/[tenantSlug]/reports/sard/sard-reports-content.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/sard/sard-sessions-content.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/sard/[sardSessionId]/evaluate/sard-evaluate-content.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/teachers/[id]/teacher-detail-content.tsx`, `src/features/sard/sard-teacher-assignment-fields.tsx`
- Related docs: _See the project documentation indexes for the owning feature._

## Scope

- Multi-teacher assignment when creating a sard session (multiple non-overlapping parts; same teacher may get several parts).
- Daily stop points on the evaluate page (teacher effort per calendar day).
- Admin sard reports (teacher + period filter, summary totals, detail table).
- Teacher profile sard effort card and link to filtered report.

## Test cases

### Session create — assignments

| ID | Key | Case | Type | Steps | Expected | Related code |
|----|-----|------|------|-------|----------|--------------|
| TC-01 | ALFQ-SARD-RPT-001 | Verify a single-teacher sard session does not show manual assignment UI | Main | 1. Open the "Sard" area.<br>2. Click "New sard session".<br>3. Pick a student, date, and exactly one committee teacher.<br>4. Create the session. | 1. The "Divide work among teachers" block does not appear.<br>2. The session is created successfully. | - |
| TC-02 | ALFQ-SARD-RPT-002 | Verify a multi-teacher session requires divided assignments | Main | 1. Open "New sard session".<br>2. Select two committee teachers.<br>3. Confirm the assignment section appears with named surah pickers (not raw surah numbers).<br>4. Add parts so each teacher has at least one part; ranges must cover the full sard scope with no gaps or overlaps.<br>5. Create the session.<br>6. Set two parts to the same range for the same teacher and confirm the overlap error appears immediately.<br>7. Change one part to the other teacher with the same range and confirm the overlap error still appears.<br>8. Give teacher A a first part, teacher B a second part, then add another non-overlapping part for teacher A again. | 1. "Divide work among teachers" is shown.<br>2. Surah fields show surah names; ayah min/max follow the sard scope and each surah's ayah count.<br>3. The session is created when parts are valid and cover the scope.<br>4. Each assigned teacher can open "Evaluate" for that session.<br>5. Overlapping parts show a localized overlap error (e.g. Arabic: "لا يجوز أن تتداخل نطاقات المعلمين.") and red outlines — including when both parts use the same teacher.<br>6. The same teacher can own more than one non-overlapping part. | `src/app/[locale]/(tenant)/[tenantSlug]/sard/sard-sessions-content.tsx`, `src/features/sard/sard-teacher-assignment-fields.tsx`, `src/features/sard/sard-assignment-validation.ts`, `src/features/sard/sard-session-errors.ts` |

### Evaluate — daily stop point

| ID | Key | Case | Type | Steps | Expected | Related code |
|----|-----|------|------|-------|----------|--------------|
| TC-03 | ALFQ-SARD-RPT-003 | Verify the teacher can save today's stop point on the evaluate page | Main | 1. As an assigned committee teacher, open "Evaluate" for a sard session.<br>2. Select a surah within your assignment.<br>3. Set the stop ayah within the segment.<br>4. Click "Save today's stop point". | 1. A success message shows today's ayah count.<br>2. A banner shows today's saved range.<br>3. Saving again the same day updates the last stop for that day. | - |

### Admin sard reports

| ID | Key | Case | Type | Steps | Expected | Related code |
|----|-----|------|------|-------|----------|--------------|
| TC-04 | ALFQ-SARD-RPT-004 | Verify the sard reports page filters by teacher and period | Main | 1. Open "Sard reports" (`/reports/sard`).<br>2. Select a teacher and a date range that includes a saved stop date.<br>3. Click "Apply". | 1. Summary cards show totals for pages, juzs, surahs, and ayahs.<br>2. The table lists rows with student name, email, phone, date, range, counts, and grade.<br>3. Failed or undecided sessions show "----" in the grade column. | - |
| TC-05 | ALFQ-SARD-RPT-005 | Verify report rows show only that teacher's daily effort | Main | 1. Use a multi-teacher session with stop points on different days.<br>2. Open the sard report for teacher A for a single day only. | 1. Each row's range and counts match teacher A's work that day only.<br>2. Teacher B's work on the same session does not appear in teacher A's report. | - |

### Teacher profile

| ID | Key | Case | Type | Steps | Expected | Related code |
|----|-----|------|------|-------|----------|--------------|
| TC-06 | ALFQ-SARD-RPT-006 | Verify the teacher profile shows sard effort totals and a report link | Main | 1. Open a teacher profile that has saved daily stop points.<br>2. Find the "Sard effort" card.<br>3. Click "Open sard report". | 1. The card shows all-time pages, juzs, surahs, and ayahs.<br>2. The link opens sard reports with that teacher pre-selected. | - |
