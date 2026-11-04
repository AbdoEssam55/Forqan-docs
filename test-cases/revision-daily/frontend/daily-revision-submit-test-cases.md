# Daily revision submit UI — Frontend Test Cases

## Metadata

- Layer: frontend
- Related code: `src/features/alforqan/daily-revision/daily-revision-panel.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/assignments/[id]/assignment-detail-content.tsx`
- Related docs: `Alforqan-Frontend/docs/technical/tenant/daily-revision.md`
- Automation: -

## Scope

Student/parent assignment screen for open revision homework.

## Test cases

### Assignment panel

| ID | Key | Case | Steps | Expected |
| --- | --- | --- | --- | --- |
| TC-01 | ALFQ-REVDAY-UI-001 | Verify student saves slice inside homework range | 1. Sign in as student with `SubmitRevisionClaim`.<br>2. Open open revision assignment.<br>3. Pick range inside a homework range and submit. | 1. Success toast appears.<br>2. Day shows delivered with range. |
| TC-02 | ALFQ-REVDAY-UI-002 | Verify parent self label and hidden name | 1. Sign in as parent.<br>2. Open child revision assignment.<br>3. Choose self listener option. | 1. Label shows "The student revised by himself" (or Arabic equivalent).<br>2. Name field is hidden. |
| TC-03 | ALFQ-REVDAY-UI-003 | Verify delivered day shows no recording | 1. Submit a day without audio.<br>2. View that day in the list. | 1. Range and relation are visible.<br>2. Copy shows "no recording" equivalent. |
| TC-04 | ALFQ-REVDAY-UI-004 | Verify missed day has no edit form | 1. View a past missing day after student midnight passed. | 1. Status shows missing.<br>2. No submit form for that day. |
