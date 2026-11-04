# Daily revision correction UI — Frontend Test Cases

## Metadata

- Layer: frontend
- Related code: `src/features/alforqan/daily-revision/daily-revision-correction-dialog.tsx`, `src/features/alforqan/evaluation/halaqa-revision-evaluation-panel.tsx`
- Related docs: `Alforqan-Frontend/docs/technical/tenant/daily-revision.md`
- Automation: -

## Scope

Correction button visibility and dialog on teacher evaluation.

## Test cases

### Evaluation UI

| ID | Key | Case | Steps | Expected |
| --- | --- | --- | --- | --- |
| TC-01 | ALFQ-REVDAY-COR-005 | Verify correction button hidden without claim | 1. Sign in as teacher without `CorrectDailyRevisionClaim`.<br>2. Open revision evaluation for a student with a missing day. | No correction button on missing days. |
