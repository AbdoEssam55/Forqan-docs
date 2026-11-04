# User activation — counts & lists — Backend Test Cases

## Metadata

- Project: `alforqan`
- Feature: `users`
- Layer: `backend`
- Routes: `-`
- APIs: `GET` dashboard stats, `GET` users list with `isActive`
- Related code: `Alforqan/Alforqan.Core/Features/Dashboard/Queries/Handlers/GetDashboardStatsQueryHandler.cs`
- Related docs: `docs/technical/tenant/user-activation.md`
- Automation: `-`

## Scope

- Dashboard and users hub counts split active vs inactive.
- Users list filter by `isActive`.

## Test cases

### Wave 4 — Counts

| ID | Key | Case | Steps | Expected | Related code |
|----|-----|------|-------|----------|--------------|
| TC-01 | ALFQ-USER-ACT-BE-030 | Verify landing active student count drops on deactivate | 1. Note landing active students.<br>2. Deactivate a student.<br>3. Reload landing stats. | 1. Active student count decreases by one. | `Alforqan/Alforqan.Core/Features/Landing/Queries/Handlers/GetLandingStatsQueryHandler.cs` |
| TC-02 | ALFQ-USER-ACT-BE-031 | Verify dashboard returns active and inactive counts | 1. Deactivate one teacher.<br>2. GET dashboard stats. | 1. `teachersCount` is active only.<br>2. `inactiveTeachersCount` includes deactivated teacher. | `Alforqan/Alforqan.Core/Features/Dashboard/Queries/Handlers/GetDashboardStatsQueryHandler.cs` |
| TC-03 | ALFQ-USER-ACT-BE-032 | Verify users list isActive=false returns only inactive | 1. Deactivate a user.<br>2. GET users with `isActive=false`. | 1. Deactivated user appears.<br>2. Active users omitted. | `Alforqan/Alforqan.Core/Features/Users/Tenant/Queries/Handlers/AppUserQueryHandler.cs` |
| TC-04 | ALFQ-USER-ACT-BE-033 | Verify countsByRole counts active users only | 1. Deactivate user in a role.<br>2. GET dashboard stats. | 1. Role bucket count does not include inactive user. | `Alforqan/Alforqan.Core/Features/Dashboard/Queries/Handlers/GetDashboardStatsQueryHandler.cs` |
