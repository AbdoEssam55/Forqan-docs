# User activation — live pickers — Backend Test Cases

## Metadata

- Project: `alforqan`
- Feature: `users`
- Layer: `backend`
- Routes: `-`
- APIs: Sard session create, homework assignment, staff report filters
- Related code: `Alforqan/Alforqan.Core/Features/Sard/Commands/SardSessionMediators.cs`
- Related docs: `docs/technical/tenant/user-activation.md`
- Automation: `-`

## Scope

- Sard, homework, and current-report pickers exclude inactive students.
- Historical sard results and past reports remain readable.

## Test cases

### Wave 3 — Sard, reports, homework

| ID | Key | Case | Steps | Expected | Related code |
|----|-----|------|-------|----------|--------------|
| TC-01 | ALFQ-USER-ACT-BE-020 | Verify new sard session cannot target inactive student | 1. Deactivate student.<br>2. Attempt create sard for that student. | 1. Request fails (inactive user). | `Alforqan/Alforqan.Core/Features/Sard/Commands/SardSessionMediators.cs` |
| TC-02 | ALFQ-USER-ACT-BE-021 | Verify existing sard result still opens after deactivate | 1. Complete sard for student.<br>2. Deactivate student.<br>3. Open result by id. | 1. Result page loads with student name. | `-` |
| TC-03 | ALFQ-USER-ACT-BE-022 | Verify homework assignment create rejects inactive student | 1. Deactivate student.<br>2. Create assignment for that student. | 1. Create fails. | `Alforqan/Alforqan.Service/Implementation/StudentAssignments/Services/StudentAssignmentService.cs` |
| TC-04 | ALFQ-USER-ACT-BE-023 | Verify staff report current student directory omits inactive | 1. Deactivate student.<br>2. Load current directory/filter options used for live reports. | 1. Student not in picker list. | `Alforqan/Alforqan.Core/Features/StaffReports/Helpers/StaffReportTeacherScopeHelper.cs` |
| TC-05 | ALFQ-USER-ACT-BE-024 | Verify reactivate allows sard/homework again | 1. Activate student.<br>2. Create sard or homework. | 1. Operation succeeds. | `Alforqan/Alforqan.Service/Implementation/Identity/TenantUserActivationService.cs` |
