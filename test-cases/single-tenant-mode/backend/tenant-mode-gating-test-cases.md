# Single-Tenant Mode — Backend Test Cases

## Metadata
- Project: `alforqan`
- Feature: `single-tenant-mode`
- Layer: `backend`
- Routes: -
- APIs: `GET /API/V01/Host/*`, `GET /API/V01/TenantConfig`, any tenant API under `/API/V01/*`
- Related code: `Alforqan/Alforqan.Api/Middlewares/TenantResolutionMiddleware.cs`, `Alforqan/Alforqan.Shared/Settings/TenantSettings.cs`, `Alforqan/Alforqan.Shared/Settings/MultiTenancyMode.cs`, `Platform/Platform.Infrastructure/Seeding/HostDbSeeding.cs`
- Related docs: `docs/single-tenant-strategy.md`, `Backend/docs/technical-docs/tenant-resolution-guide.md`

## Scope
- The multi-tenant platform code is kept but hidden behind `TenantSettings:Mode` (`SingleTenant` | `MultiHost`).
- In `SingleTenant` mode the app always resolves one fixed tenant (`DefaultTenantCode`, e.g. `alforqan`) and blocks all Host (platform-admin) routes.
- Flipping `Mode` to `MultiHost` restores domain/header resolution and exposes Host routes — no product code change.

## Test scenarios
- Host routes return "not found" while single-tenant mode is on.
- Any tenant request resolves the fixed tenant regardless of Host header / `X-Tenant-Code`.
- The fixed `alforqan` tenant is seeded with Core, Reports, and PaymentsBasic enabled.
- Switching to `MultiHost` re-enables Host routes and domain/header resolution.

## Test cases

### Host routes hidden in single-tenant mode

| ID | Key | Case | Type | Preconditions | Steps | Expected | Related code |
|----|-----|------|------|---------------|-------|----------|--------------|
| TC-01 | ALF-STM-HOST-001 | Verify every Host platform-admin endpoint returns "not found" in single-tenant mode | Main | `TenantSettings:Mode` = "SingleTenant". | 1. Call a Host endpoint under "/API/V01/Host/" (for example "/API/V01/Host/Tenants").<br>2. Call the Host login endpoint "/API/V01/Host/Auth/Login". | 1. Both requests return HTTP 404 (not found).<br>2. No host/tenant management data or token is returned.<br>3. The response is not a 401/403 — the routes appear to not exist. | `Alforqan/Alforqan.Api/Middlewares/TenantResolutionMiddleware.cs` |
| TC-02 | ALF-STM-HOST-002 | Verify Host endpoints become reachable again when mode is "MultiHost" | Edge | `TenantSettings:Mode` = "MultiHost".<br>App restarted. | 1. Call "/API/V01/Host/Auth/Login" with valid host credentials. | 1. The request is no longer blocked with 404.<br>2. The host login flow runs normally (200 with token, or 401 for bad credentials). | `Alforqan/Alforqan.Api/Middlewares/TenantResolutionMiddleware.cs` |

### Fixed tenant resolution

| ID | Key | Case | Type | Preconditions | Steps | Expected | Related code |
|----|-----|------|------|---------------|-------|----------|--------------|
| TC-03 | ALF-STM-RES-001 | Verify a tenant request resolves the fixed tenant regardless of Host header | Main | `TenantSettings:Mode` = "SingleTenant".<br>`DefaultTenantCode` = "alforqan".<br>The `alforqan` tenant is seeded and active. | 1. Send a request to any tenant API (for example "/API/V01/TenantConfig") from an arbitrary host name.<br>2. Do not send any `X-Tenant-Code` header. | 1. The request succeeds.<br>2. The resolved tenant is "alforqan".<br>3. `TenantConfig` returns the alforqan tenant's code and enabled modules. | `Alforqan/Alforqan.Api/Middlewares/TenantResolutionMiddleware.cs`, `Alforqan/Alforqan.Shared/Settings/TenantSettings.cs` |
| TC-04 | ALF-STM-RES-002 | Verify a mismatching `X-Tenant-Code` header is ignored in single-tenant mode | Edge | `TenantSettings:Mode` = "SingleTenant".<br>`DefaultTenantCode` = "alforqan". | 1. Send a tenant API request with header `X-Tenant-Code: some-other-code`. | 1. The request still resolves the "alforqan" tenant.<br>2. The "some-other-code" value has no effect. | `Alforqan/Alforqan.Api/Middlewares/TenantResolutionMiddleware.cs` |
| TC-05 | ALF-STM-RES-003 | Verify a clear error when the fixed tenant is missing | Edge | `TenantSettings:Mode` = "SingleTenant".<br>`DefaultTenantCode` points to a code with no seeded tenant. | 1. Send a request to any tenant API. | 1. The request fails with a "tenant not found" response (HTTP 404).<br>2. The error names the configured tenant code so the misconfiguration is obvious. | `Alforqan/Alforqan.Api/Middlewares/TenantResolutionMiddleware.cs` |

### Fixed tenant seeding

| ID | Key | Case | Type | Preconditions | Steps | Expected | Related code |
|----|-----|------|------|---------------|-------|----------|--------------|
| TC-06 | ALF-STM-SEED-001 | Verify startup seeds the fixed alforqan tenant with modules enabled | Main | A fresh Host DB.<br>`DefaultTenantCode` = "alforqan". | 1. Start the API.<br>2. Inspect the seeded tenant row and its modules. | 1. Exactly one tenant with code "alforqan" exists (or a legacy "default"/localhost tenant was renamed to "alforqan").<br>2. The tenant status is "Active".<br>3. Modules "Core", "Reports", and "PaymentsBasic" are enabled. | `Platform/Platform.Infrastructure/Seeding/HostDbSeeding.cs`, `Alforqan/Alforqan.Api/Program.cs` |
