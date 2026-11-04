# Halaqa schedule — Frontend Test Cases

## Metadata

- Layer: frontend
- Related code: `src/features/alforqan/components/halaqa-form.tsx`, `src/features/alforqan/halaqat/halaqa-schedule-fields.tsx`
- Related docs: `Alforqan-Backend/docs/technical-docs/halaqa-schedule.md`
- Automation: -

## Scope

Halaqa create/edit UI for type, session weekdays, and timezone. Type is fixed as memorization and revision. A weekday row is start and end time only.

## Test cases

### Form

| ID | Key | Case | Steps | Expected | Related code |
| --- | --- | --- | --- | --- | --- |
| TC-06 | ALFQ-REVDAY-SCH-006 | Verify schedule section requires at least one weekday | 1. Open halaqa create as admin.<br>2. Leave all weekdays unchecked.<br>3. Save. | 1. Weekday checklist is visible.<br>2. Save is blocked or shows validation error.<br>3. API is not called with empty schedules. | `src/features/alforqan/components/halaqa-form.tsx` |
| TC-07 | ALFQ-REVDAY-SCH-007 | Verify halaqa type is only memorization and revision | 1. Open halaqa create as admin.<br>2. Read the type field.<br>3. Open an existing halaqa edit page. | 1. Create shows type "Memorization and revision" / "حفظ ومراجعة".<br>2. "Narration", "Mixed", "قراءة", and "مختلط" are not offered.<br>3. Edit shows the same single type. | `src/features/alforqan/components/halaqa-form.tsx` |
| TC-08 | ALFQ-REVDAY-SCH-008 | Verify a session day does not ask for session type | 1. Open halaqa create as admin.<br>2. Check one weekday. | 1. That day shows "Start" / "البداية" and "End" / "النهاية".<br>2. "Session type" / "نوع الجلسة" is not shown. | `src/features/alforqan/halaqat/halaqa-schedule-fields.tsx` |
