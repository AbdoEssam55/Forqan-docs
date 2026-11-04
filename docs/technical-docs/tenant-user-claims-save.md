# Tenant user claims save

## Save from Roles & permissions

1. If roles did not change, skip `ModifyUserRoles`.
2. `PUT .../Claims/ModifyUserClaims` with checked claims `claimValue: true`.
3. Tenant persist = exactly those claims (`TenantUserClaimEditRules`). Role defaults are not re-added.
4. Refresh the tenant JWT (`refreshToken`). Do not rely on ForceLogout for claims-only edits.

## Unchanged roles

1. `UserRoleAssignmentRules.RoleSetEquals` → service returns Data `"unchanged"`.
2. Handler does **not** reseed user claims, revoke tokens, or ForceLogout.

## After save (self)

1. Stay signed in.
2. New JWT must omit unchecked claims (e.g. `ViewNarrationReportsClaim`).
3. UI (Sard tab, session evaluate) follows the new JWT.

## Files

- `Alforqan.Service/.../AuthorizationUserService.cs`
- `Alforqan.Core/.../AuthorizationCommandHandler.cs`
- `Frontend/.../user-roles-and-claims-tab.tsx`
- `Frontend/.../permissions-provider.tsx`
