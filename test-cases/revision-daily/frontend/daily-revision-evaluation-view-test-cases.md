# Daily revision evaluation view — Frontend Test Cases

## Metadata

- Layer: frontend
- Related code: `src/features/alforqan/evaluation/halaqa-revision-evaluation-panel.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/reports/session/[sessionId]/student/[studentId]/session-student-report-content.tsx`
- Related docs: `Alforqan-Frontend/docs/technical/tenant/daily-revision.md`
- Automation: -

## Scope

Teacher evaluation and session report show daily revision days.

## Test cases

### Teacher views

| ID | Key | Case | Steps | Expected |
| --- | --- | --- | --- | --- |
| TC-01 | ALFQ-REVDAY-UI-005 | Verify evaluation shows missing and delivered days | 1. Open revision evaluation for a student with open revision homework.<br>2. Ensure data has one missing past day and one delivered day without audio. | 1. Missing day listed with missing status.<br>2. Delivered day shows range and "no recording" equivalent. |
