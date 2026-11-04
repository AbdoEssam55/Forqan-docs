# Student Actions — Frontend Test Cases (page)

## Metadata

- Project: `alforqan`
- Feature: `student-actions`
- Layer: `frontend`
- Routes: `/{locale}/{tenantSlug}/student-actions`
- APIs: `POST /API/V01/StudentActions/match`
- Related code: `src/features/alforqan/student-actions/student-actions-page.tsx`
- Related docs: `docs/technical/tenant/student-actions.md`
- Automation: `-`

## Scope

- Nav, layout guard, rules UI, Apply → table. No notify/deactivate buttons in this wave.

## Delivery waves

| Wave | Intent | Keys |
| ---- | ------ | ---- |
| 1 | Page shell + match UI | `ALFQ-ACT-UI-001`–`005` |

## Test cases

### Wave 1 — Page shell + match UI

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-01 | ALFQ-ACT-UI-001 | Verify People nav shows Student actions for Admin | Main | Tenant Admin with ViewStudentActionsClaim | 1. Sign in as Admin.<br>2. Open People nav. | 1. A link labeled "Student actions" / "إجراءات الطلاب" appears after Students. | - | `src/features/alforqan/config/tenant-nav.ts` | - |
| TC-02 | ALFQ-ACT-UI-002 | Verify Teacher does not see the Student actions nav item | Main | Teacher without ViewStudentActionsClaim | 1. Sign in as Teacher.<br>2. Open People nav. | 1. "Student actions" / "إجراءات الطلاب" is not shown. | - | `src/features/alforqan/config/tenant-nav.ts` | - |
| TC-03 | ALFQ-ACT-UI-003 | Verify default rule Absent greater than or equal to 3 and Apply lists only matching students | Main | Admin; attendance data in last 30 days | 1. Open Student actions.<br>2. Click Apply without changing the default rule. | 1. First rule shows Absent, ≥, 3.<br>2. Table lists only students with at least 3 unexcused Absent marks in the period. | ALFQ-ACT-UI-001 | `src/features/alforqan/student-actions/student-actions-page.tsx` | - |
| TC-04 | ALFQ-ACT-UI-004 | Verify AND vs OR changes results for two rules | Main | Students with mixed Absent/Late counts | 1. Add rules Absent gte 3 and Late gte 5.<br>2. Apply with AND.<br>3. Switch to OR and Apply again. | 1. AND shows only students meeting both rules.<br>2. OR shows students meeting at least one rule. | ALFQ-ACT-UI-003 | `src/features/alforqan/student-actions/student-actions-page.tsx` | - |
| TC-05 | ALFQ-ACT-UI-005 | Verify Notify and Deactivate are not on the page yet | Main | Admin on Student actions after Apply | 1. Apply any rule.<br>2. Scan toolbar above the table. | 1. No "Notify" or "Deactivate" buttons are visible. | ALFQ-ACT-UI-003 | `src/features/alforqan/student-actions/student-actions-page.tsx` | - |
