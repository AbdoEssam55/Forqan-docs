# Student Actions — Frontend Test Cases (commands UI)

## Metadata

- Project: `alforqan`
- Feature: `student-actions`
- Layer: `frontend`
- Routes: `/{locale}/{tenantSlug}/student-actions`
- APIs: `POST /API/V01/StudentActions/notify`, `POST /API/V01/StudentActions/deactivate`
- Related code: `src/features/alforqan/student-actions/student-actions-page.tsx`
- Related docs: `docs/technical/tenant/student-actions.md`
- Automation: `-`

## Scope

- Notify and deactivate selected rows from match results; claim-gated toolbar.

## Delivery waves

| Wave | Intent | Keys |
| ---- | ------ | ---- |
| 1 | Commands UI | `ALFQ-ACT-UI-010`–`014` |

## Test cases

### Wave 1 — Commands UI

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-01 | ALFQ-ACT-UI-010 | Verify Notify on one matching student shows the title and body in that student bell | Main | Admin with TakeStudentActionsClaim; student with login in match results | 1. Apply match; select one student with active login.<br>2. Click Notify selected; enter title and body; send.<br>3. Sign in as that student; open notifications. | 1. Send succeeds.<br>2. Student bell shows one unread item with the same title and body. | ALFQ-ACT-UI-003 | `src/features/alforqan/student-actions/student-actions-page.tsx` | - |
| TC-02 | ALFQ-ACT-UI-011 | Verify Deactivate confirm then student cannot log in | Main | Active student in match results | 1. Select student; Deactivate selected; confirm.<br>2. Try student login. | 1. Success toast appears.<br>2. Login is blocked or inactive message shown. | ALFQ-ACT-UI-003 | `src/features/alforqan/student-actions/student-actions-page.tsx` | - |
| TC-03 | ALFQ-ACT-UI-012 | Verify bulk Notify and Deactivate only apply to checked rows | Main | At least two students in results | 1. Check one row only; send Notify.<br>2. Check a different single row; Deactivate. | 1. Only the notified student receives the alert.<br>2. Only the deactivated student loses login. | ALFQ-ACT-UI-010 | `src/features/alforqan/student-actions/student-actions-page.tsx` | - |
| TC-04 | ALFQ-ACT-UI-013 | Verify Deactivate selected is disabled when the selection includes a student with no login | Main | Match row with no UserId | 1. Select only a student with account "No login". | 1. Deactivate selected button is disabled. | ALFQ-ACT-UI-003 | `src/features/alforqan/student-actions/student-actions-page.tsx` | - |
| TC-05 | ALFQ-ACT-UI-014 | Verify user with only ViewStudentActionsClaim sees results but not Notify or Deactivate | Main | Role granted View only, not Take | 1. Open Student actions; Apply.<br>2. Look for action toolbar and row checkboxes. | 1. Table loads.<br>2. No Notify selected, Deactivate selected, or row checkboxes. | ALFQ-ACT-UI-001 | `src/features/alforqan/student-actions/student-actions-page.tsx`, `src/lib/auth/tenant-claims.ts` | - |
