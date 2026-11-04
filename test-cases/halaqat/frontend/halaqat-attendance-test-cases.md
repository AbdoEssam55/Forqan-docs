# Halaqa session attendance — Frontend Test Cases

## Metadata

- Project: `alforqan`
- Feature: `halaqat`
- Layer: `frontend`
- Routes: `/{locale}/{tenantSlug}/halaqat/{halaqaId}/sessions/{sessionId}`
- APIs: -
- Related code: `src/features/alforqan/components/sessions/attendance-sheet.tsx`
- Related docs: `docs/technical/tenant/halaqat-guide.md`, `docs/explanation/tenant/halaqat-explained.md`
- Automation: -

## Scope

- Teacher bulk attendance for a halaqa session from the session detail page.
- Required mark for every student before save and before ending the session.
- Late duration (minutes or hours).

## Test pages

- `/{locale}/{tenantSlug}/halaqat/{halaqaId}/sessions/{sessionId}`

## Test scenarios

- Load enrolled students and set presence per student.
- Save attendance and see persisted values after refresh.
- Block save and end-session until every student is marked.
- Late requires how-late duration.

## Delivery waves

| Wave | Intent | Keys (representative) | Depends on (cross-file) |
| ---- | ------ | --------------------- | ----------------------- |
| 1 | Attendance sheet — load, save, read-only when completed | `ALFQ-HAL-ATT-001`–`003` | [sessions](halaqat-sessions-test-cases.md) Wave 2: `ALFQ-HAL-SES-003`, `ALFQ-HAL-SES-005` |
| 2 | Required marks, late duration, end-session gate | `ALFQ-HAL-ATT-004`–`007` | Wave 1; session in progress (`ALFQ-HAL-SES-005` start only if session not yet completed) |

Run after sessions; before [evaluation](halaqat-evaluation-test-cases.md). `ALFQ-HAL-ATT-002` gates `ALFQ-HAL-EVA-001` (evaluation range).

## Test cases

### Wave 1 — Attendance sheet

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-01 | ALFQ-HAL-ATT-001 | Verify each enrolled student appears with a status control | Main | Halaqa has at least two students; session exists | 1. Open the session detail page.<br>2. Scroll to the "Attendance" / "الحضور" section. | 1. Each enrolled student name appears.<br>2. Each row has a status control.<br>3. Unmarked rows show placeholder "Select status" / "اختر الحالة" (not a saved "Present" / "حاضر"). | ALFQ-HAL-SES-003 | `src/features/alforqan/components/sessions/attendance-sheet.tsx`, `src/features/alforqan/components/sessions/session-attendance-mark-fields.tsx` | Students: "Ahmad", "Sara" |
| TC-02 | ALFQ-HAL-ATT-002 | Verify changing status and saving shows the saved status after reload | Main | User can update halaqa; session not completed or cancelled | 1. Set every student to a status (first student "Absent" / "غائب").<br>2. Choose "Save attendance" / "حفظ الحضور".<br>3. Refresh the session detail page. | 1. A success toast appears with "Attendance saved" / "تم حفظ الحضور".<br>2. The first student's status control shows "Absent" / "غائب" after reload. | ALFQ-HAL-ATT-001 | `src/features/alforqan/components/sessions/attendance-sheet.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/halaqat/[id]/sessions/[sessionId]/halaqa-session-detail-content.tsx` | - |
| TC-03 | ALFQ-HAL-ATT-003 | Verify attendance controls are read-only when session is completed | Regression | Session status is "Completed" | 1. Open the same session detail page.<br>2. Try to change a status. | 1. Status controls are disabled or saving is not offered.<br>2. No new attendance save button for teachers. | ALFQ-HAL-SES-005 | `src/features/alforqan/components/sessions/attendance-sheet.tsx` | - |

### Wave 2 — Required marks and late duration

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-04 | ALFQ-HAL-ATT-004 | Verify Late requires a duration before save | Main | Session in progress; attendance still editable | 1. Set one student to "Late" / "متأخر".<br>2. Leave "How late" / "مدة التأخر" empty.<br>3. Choose "Save attendance" / "حفظ الحضور". | 1. Save does not succeed.<br>2. An error matching "Enter how late, from 1 minute up to 24 hours." / "أدخل مدة التأخر من دقيقة واحدة وحتى 24 ساعة." appears. | ALFQ-HAL-ATT-001 | `src/features/alforqan/components/sessions/attendance-sheet.tsx`, `src/features/alforqan/components/sessions/session-attendance-mark-fields.tsx` | Late, empty duration |
| TC-05 | ALFQ-HAL-ATT-005 | Verify save is blocked until every student is marked | Main | Session in progress; at least two students; at least one still "Select status" | 1. Mark only the first student "Present" / "حاضر".<br>2. Leave the second unmarked.<br>3. Choose "Save attendance" / "حفظ الحضور". | 1. Save does not succeed.<br>2. An error matching "Mark every student before saving." / "حدد حضور كل طالب قبل الحفظ." appears. | ALFQ-HAL-ATT-001 | `src/features/alforqan/components/sessions/attendance-sheet.tsx` | Two students |
| TC-06 | ALFQ-HAL-ATT-006 | Verify end session is blocked until attendance is saved | Main | Session status is "In progress"; at least one student has no saved attendance row | 1. Do not save attendance for every student.<br>2. Look at "End session" / "إنهاء الجلسة". | 1. "End session" is disabled.<br>2. Message "Mark every student present, absent, late, or excused before ending the session." / "حدد حضور كل طالب (حاضر أو غائب أو متأخر أو معذور) قبل إنهاء الجلسة." is visible. | ALFQ-HAL-SES-003 | `src/app/[locale]/(tenant)/[tenantSlug]/halaqat/[id]/sessions/[sessionId]/halaqa-session-detail-content.tsx`, `src/features/alforqan/components/sessions/session-lifecycle-confirm-button.tsx` | In-progress session |
| TC-07 | ALFQ-HAL-ATT-007 | Verify Late duration is kept after save and reload | Main | Session in progress; all students can be marked | 1. Set first student to "Late" / "متأخر".<br>2. Enter 15 and unit "Minutes" / "دقائق".<br>3. Mark remaining students Present or Absent.<br>4. Choose "Save attendance" / "حفظ الحضور".<br>5. Refresh the page. | 1. Toast "Attendance saved" / "تم حفظ الحضور".<br>2. First student still shows "Late" / "متأخر".<br>3. Duration still shows 15 minutes. | ALFQ-HAL-ATT-004 | `src/features/alforqan/components/sessions/attendance-sheet.tsx` | 15 minutes |
