# User activation — Backend Test Cases

## Metadata

- Project: `alforqan`
- Feature: `users`
- Layer: `backend`
- Routes: `-`
- APIs: `PATCH /API/V01/AppUser/UnActivate`, `PATCH /API/V01/AppUser/Activate`
- Related code: `Alforqan/Alforqan.Service/Implementation/Identity/TenantUserActivationService.cs`
- Related docs: `docs/technical/tenant/user-activation.md`
- Automation: `Tests/Alforqan.Core.Tests/Features/Users/TenantUserActivationHelperTests.cs`

## Scope

- Tenant user pause: sync `AppUser` and profile `IsActive` on deactivate/activate.
- `ModifyUserRolesClaim` on Activate/UnActivate.
- Login blocked while inactive.

## Test cases

### Wave 1 — Core sync

| ID | Key | Case | Steps | Expected | Related code |
|----|-----|------|-------|----------|--------------|
| TC-01 | ALFQ-USER-ACT-BE-001 | Verify deactivate student syncs AppUser and Student IsActive false | 1. Deactivate student via `PATCH AppUser/UnActivate` or student edit `IsActive` off.<br>2. Load student and linked AppUser. | 1. `AppUser.IsActive` is false.<br>2. `Student.IsActive` is false.<br>3. `HalaqaStudent` rows unchanged. | `Alforqan/Alforqan.Service/Implementation/Identity/TenantUserActivationService.cs` |
| TC-02 | ALFQ-USER-ACT-BE-002 | Verify deactivate teacher syncs AppUser and Teacher IsActive false | 1. Deactivate teacher login user.<br>2. Load teacher record. | 1. Both flags false.<br>2. `Halaqa.TeacherId` unchanged. | `Alforqan/Alforqan.Service/Implementation/Identity/TenantUserActivationService.cs` |
| TC-03 | ALFQ-USER-ACT-BE-003 | Verify deactivate parent syncs AppUser and Parent IsActive false | 1. Deactivate parent login user.<br>2. Load parent record. | 1. Both flags false. | `Alforqan/Alforqan.Service/Implementation/Identity/TenantUserActivationService.cs` |
| TC-04 | ALFQ-USER-ACT-BE-004 | Verify activate restores AppUser and profile IsActive true | 1. Deactivate then `PATCH AppUser/Activate`.<br>2. Reload user and profile. | 1. `AppUser.IsActive` is true.<br>2. Profile `IsActive` is true. | `Alforqan/Alforqan.Core/Features/Users/Tenant/Commands/Handlers/AppUserCommandHandler.cs` |
| TC-05 | ALFQ-USER-ACT-BE-005 | Verify login blocked while AppUser inactive | 1. Deactivate user.<br>2. Attempt tenant login. | 1. Login fails or shows inactive account message. | `Alforqan/Alforqan.Service/Implementation/Identity/TenantUserActivationService.cs` |
| TC-06 | ALFQ-USER-ACT-BE-006 | Verify Activate requires ModifyUserRolesClaim | 1. Call `PATCH AppUser/Activate` without claim. | 1. Response is 403 Forbidden. | `Alforqan/Alforqan.Api/Controllers/Identity/AppUsers/AppUserController.cs` |
