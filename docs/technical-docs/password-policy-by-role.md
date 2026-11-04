# Password policy by role

## Policies

| Roles | Min | Max | Complexity |
|-------|-----|-----|------------|
| Student, Parent | 6 | 256 | Length only |
| Teacher, Admin, Supervisor, host users | 6 | 500 | Digit + lower + upper + non-alphanumeric |

## Enforcement

1. **ASP.NET Identity** — `RoleAwarePasswordValidator` (`Platform.Infrastructure/Implementation/Identity/RoleAwarePasswordValidator.cs`) is the authority for `CreateAsync`, `ChangePasswordAsync`, and `AddPasswordAsync`.
2. **Create without role** — `AddAppUserCommand.UseSimplePasswordRules` + `PasswordPolicyContext` during `CreateAsync` (student/parent handlers set the flag).
3. **FluentValidation** — `PasswordPolicyValidationRules` mirrors the same rules for early API errors.

## Key files

- `Alforqan.Shared/HelperClasses/PasswordPolicyHelper.cs`
- `Alforqan.Shared/HelperClasses/PasswordPolicyContext.cs`
- `Alforqan.Core/Features/Users/Tenant/Helpers/TenantPasswordPolicyResolver.cs`
- `ModuleInfrastructureDependencies.cs` / `Program.cs` — built-in `RequireDigit` etc. are **false**; custom validator applies rules.

## Frontend parity

See `Frontend/alforqan/docs/technical/tenant/frontend-backend-validation-parity.md` and `src/lib/validation/tenant-identity-rules.ts`.
