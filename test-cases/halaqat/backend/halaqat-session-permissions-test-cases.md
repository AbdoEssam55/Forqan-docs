# Halaqa session permissions — Backend Test Cases

## Metadata
- Project: `alforqan`
- Feature: `halaqat`
- Layer: `backend`
- APIs: `PUT/POST .../HalaqaSessions/*`, `PUT .../evaluations/students/{studentId}`, `POST .../evaluations/students/{studentId}/submit`, `POST .../evaluations/students/{studentId}/outcome`
- Related code: `Alforqan/Alforqan.Core/Features/HalaqaSessions/Helpers/HalaqaSessionAccessHelper.cs`, `Alforqan/Alforqan.Shared/HelperClasses/TenantPoliciesHelper.cs`, `Alforqan/Alforqan.Shared/HelperClasses/TenantUserClaimEditRules.cs`, `Alforqan/Alforqan.Shared/HelperClasses/UserRoleAssignmentRules.cs`, `Alforqan/Alforqan.Service/Implementation/Identity/Authorization/AuthorizationUserService.cs`, `Alforqan/Alforqan.Service/Implementation/Identity/Authentication/CustomAuthenticationService.cs`
- Related docs: `docs/technical-docs/halaqa-session-permissions.md`, `docs/technical-docs/tenant-user-claims-save.md`
- Automation: `Tests/Alforqan.Core.Tests/Features/Identity/TenantUserClaimEditRulesTests.cs`, `Tests/Alforqan.Core.Tests/Features/Identity/UserRoleAssignmentRulesTests.cs`

## Scope

- API authorization for session manage and evaluate uses stored user claims OR assigned-teacher check.
- Tenant claim save and JWT do not re-inject unchecked role default claims.

## Test cases

### Evaluate and manage gates

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-01 | ALFQ-HAL-BE-PERM-001 | Verify admin with EvaluateHalaqaSessionsClaim can load evaluation draft via API on a non-assigned halaqa | Main | Bearer token for tenant admin who is not the halaqa teacher; user claim `EvaluateHalaqaSessionsClaim` = true in DB; session and student exist | 1. `GET .../HalaqaSessions/{sessionId}/evaluations/students/{studentId}` (or equivalent draft load). | 1. Response is 200 with evaluation DTO.<br>2. Response is not 403 teacher-only. | - | `Alforqan/Alforqan.Core/Features/HalaqaSessions/Helpers/HalaqaSessionAccessHelper.cs` | Admin JWT after re-login |
| TC-02 | ALFQ-HAL-BE-PERM-002 | Verify admin without EvaluateHalaqaSessionsClaim gets 403 on evaluation mutate | Main | Admin user has Admin role but `EvaluateHalaqaSessionsClaim` removed from `AspNetUserClaims`; not assigned teacher | 1. `PUT .../evaluations/students/{studentId}` with a valid draft body. | 1. HTTP 403.<br>2. Message matches localized teacher-only / forbidden text. | - | `Alforqan/Alforqan.Core/Features/HalaqaSessions/Commands/SessionEvaluationMediators.cs` | Admin without evaluate claim |
| TC-03 | ALFQ-HAL-BE-PERM-003 | Verify assigned teacher can submit evaluation without EvaluateHalaqaSessionsClaim on user | Main | Teacher is assigned to session's halaqa; user claims omit `EvaluateHalaqaSessionsClaim` | 1. Save draft then `POST .../evaluations/students/{studentId}/submit`. | 1. Submit returns success (200) when draft is valid.<br>2. Not 403. | - | `Alforqan/Alforqan.Core/Features/HalaqaSessions/Helpers/HalaqaSessionAccessHelper.cs` | Assigned teacher token |
| TC-04 | ALFQ-HAL-BE-PERM-004 | Verify admin with ManageHalaqaSessionsClaim can start session on non-assigned halaqa | Main | Admin has `ManageHalaqaSessionsClaim` in DB; not assigned teacher; session in startable state | 1. `POST .../HalaqaSessions/{sessionId}/start` (or product start route). | 1. HTTP 200 and session status updates.<br>2. Not 403. | - | `Alforqan/Alforqan.Core/Features/HalaqaSessions/Commands/HalaqaSessionMutations.cs` | Admin with manage claim |

### Claim persistence

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-05 | ALFQ-HAL-BE-PERM-005 | Verify ModifyUserClaimsAsync does not re-add unchecked tenant claims from role | Main | Tenant admin user; editor has ModifyAdminClaimsClaim | 1. `PUT` modify user claims with `EvaluateHalaqaSessionsClaim` = false and other admin claims true.<br>2. `GET` user claims for same user. | 1. `EvaluateHalaqaSessionsClaim` is absent or false in response.<br>2. Other requested claims remain true. | - | `Alforqan/Alforqan.Shared/HelperClasses/TenantUserClaimEditRules.cs`, `Alforqan/Alforqan.Service/Implementation/Identity/Authorization/AuthorizationUserService.cs` | Admin user id |
| TC-06 | ALFQ-HAL-BE-PERM-006 | Verify JWT after login omits role-default claims not stored on user | Main | Admin user with `EvaluateHalaqaSessionsClaim` removed from DB; Admin role still assigned | 1. Login as that user.<br>2. Decode access token permission claims. | 1. Token does not contain `EvaluateHalaqaSessionsClaim`.<br>2. Token still contains `ClaimTypes.Role` = Admin. | ALFQ-HAL-BE-PERM-005 | `Alforqan/Alforqan.Service/Implementation/Identity/Authentication/CustomAuthenticationService.cs` | Same admin user |
| TC-07 | ALFQ-HAL-BE-PERM-007 | Verify unchanged role save does not reseed user claims | Main | Admin with `ViewHalaqatClaim` already removed from `AspNetUserClaims`; Admin role still assigned | 1. `PUT` modify user roles with the same Admin membership (no role add/remove).<br>2. `GET` user claims. | 1. `ViewHalaqatClaim` stays absent or false.<br>2. Response does not restore Admin role default claims.<br>3. Refresh tokens are not revoked for this no-op. | ALFQ-HAL-BE-PERM-005 | `Alforqan/Alforqan.Shared/HelperClasses/UserRoleAssignmentRules.cs`, `Alforqan/Alforqan.Service/Implementation/Identity/Authorization/AuthorizationUserService.cs` | Same admin user |
