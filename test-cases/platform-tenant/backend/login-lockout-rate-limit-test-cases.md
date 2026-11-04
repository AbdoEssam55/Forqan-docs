# Login lockout and rate limit — Backend Test Cases

## Metadata
- Project: `alforqan`
- Feature: `platform-tenant`
- Layer: `backend`
- Routes: `-`
- APIs: `POST /API/V01/Authentication/Login`, `POST /API/V01/Host/Authentication/Login`
- Related code: `Alforqan/Alforqan.Api/Program.cs`, `Alforqan/Alforqan.Infrastructure/ModuleInfrastructureDependencies.cs`, `Platform/Platform.Infrastructure/PlatformInfrastructureDependencies.cs`, `Alforqan/Alforqan.Shared/HelperClasses/ConstantsHelper.cs`
- Related docs: `docs/features-explanation-docs/per-tenant-rate-limiting-feature.md`

## Scope

- Tenant and host login rate-limit windows and Identity account lockout duration after failed attempts.

## Test pages

- `-` (API / auth login endpoints)

## Test scenarios

- Exceeding the login attempt budget returns HTTP 429 with a ~2 minute retry.
- Identity lockout after failed passwords lasts 2 minutes.

## Delivery waves

| Wave | Intent | Keys | Depends on |
| ---- | ------ | ---- | ---------- |
| 1 | Login throttle and lockout duration | `PLAT-TEN-AUTH-BE-001`–`002` | — |

## Test cases

### Wave 1 — Login throttle duration

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-01 | PLAT-TEN-AUTH-BE-001 | Verify tenant login rate limit retry-after is about 2 minutes | Edge | API reachable; use a test tenant and wrong password | 1. Call tenant login with a wrong password until the API returns HTTP 429.<br>2. Read `Retry-After` and response `meta.retryAfterSeconds` / `meta.retryAfterMinutes`. | 1. Status is 429 Too Many Requests.<br>2. `retryAfterSeconds` is **120** (2 minutes).<br>3. `retryAfterMinutes` is **2**. | - | `Alforqan/Alforqan.Api/Program.cs` | Wrong password; demo tenant |
| TC-02 | PLAT-TEN-AUTH-BE-002 | Verify Identity lockout duration is 2 minutes after max failed logins | Edge | A real tenant user exists; lockout enabled | 1. Fail password validation until the account is locked (5 failed attempts).<br>2. Note `LockoutEnd` (or try login and observe lockout message duration). | 1. Further logins fail while locked out.<br>2. Lockout ends after about **2 minutes**, not 5. | - | `Alforqan/Alforqan.Infrastructure/ModuleInfrastructureDependencies.cs`, `Alforqan/Alforqan.Api/Program.cs` | Known tenant user; wrong password |
