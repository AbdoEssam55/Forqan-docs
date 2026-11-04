# Halaqa session permissions

## Claims

| Claim | Constant | Unlocks |
| ----- | -------- | ------- |
| Manage halaqa sessions | `ManageHalaqaSessionsClaim` | Create session, meeting link, start/end/cancel, attendance |
| Evaluate halaqa sessions | `EvaluateHalaqaSessionsClaim` | Open draft/submitted evaluation, save draft, submit, record outcome |

Both live in the **Halaqat** tenant claim group (`TenantPoliciesHelper`).

## Access rule

```text
canManageSession = isAssignedHalaqaTeacher OR hasClaim(ManageHalaqaSessionsClaim)
canEvaluate      = isAssignedHalaqaTeacher OR hasClaim(EvaluateHalaqaSessionsClaim)
```

- **Teacher** default role claims do **not** include the two session claims → teacher stays scoped to assigned halaqa only.
- **Admin** default gets all tenant claims (including both session claims) when the role is assigned.
- Authorization checks **stored user claims** only (`AspNetUserClaims`); JWT and `HalaqaSessionAccessHelper` do not re-merge role defaults.

## Implementation

1. `HalaqaSessionAccessHelper` — `CanManageHalaqaSession*`, `CanEvaluateHalaqaSessionForSessionAsync`, `ActingUserHasTenantClaimAsync`.
2. Mutations: session lifecycle / attendance → manage helper; evaluation draft / submit / outcome → evaluate helper.
3. `CustomAuthenticationService.GenerateUserTokenAsync` — JWT permission claims = user DB claims with value `true`; role **names** still added for `[Authorize(Roles=...)]`.
4. `AuthorizationUserService.ModifyUserClaimsAsync` (tenant) — persists exactly the claims the editor saved (validated against `TenantClaimsList`).
5. Role assignment (`ModifyUserRolesAsync`) still seeds role default claims when membership **changes**; an unchanged role set does not reseed user claims. Claim tab edits after that win.

## After claim changes

1. Claims-only save does **not** revoke refresh tokens or ForceLogout.
2. Client refreshes the JWT (`PermissionsChanged` or `refreshTenantSession`) so the current session picks up stored claims.
3. Role membership change still revokes tokens and ForceLogout.

## Verify

1. Admin with `EvaluateHalaqaSessionsClaim` can open evaluation for a session they do not teach.
2. Admin without that claim (unchecked and saved) gets 403 on evaluate APIs and no Evaluate UI.
3. Assigned teacher without session claims still manages/evaluates their halaqa.
