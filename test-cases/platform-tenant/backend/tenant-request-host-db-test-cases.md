# Tenant request Host DB — Backend Test Cases

## Metadata
- Project: `alforqan`
- Feature: `platform-tenant`
- Layer: `backend`
- Routes: `-`
- APIs: `-`
- Related code: `Alforqan/Alforqan.Core/Features/Tenants/Helpers/TenantJwtStatusHelper.cs`
- Related docs: `docs/technical-docs/tenant-request-host-db.md`
- Automation: `Tests/Alforqan.Core.Tests/Features/Tenants/TenantJwtStatusHelperTests.cs`, `Tests/Alforqan.Core.Tests/Features/Tenants/TenantLastActivityGateTests.cs`, `Tests/Alforqan.Core.Tests/Features/Tenants/ClientAbortTests.cs`

## Scope

- JWT tenant-status check uses the already-resolved tenant (no second Host DB roundtrip).
- LastActivityAt writes to Host DB are throttled.
- Client-cancelled requests are not treated as tenant-resolution 500s.

## Test pages

- `-` (API middleware / JWT events)

## Test scenarios

- Active tenant JWT is accepted from in-memory tenant status.
- Inactive or suspended tenant JWT is rejected without a second tenant lookup.
- LastActivityAt is skipped when a write ran recently for the same tenant.
- Cancelled HTTP requests do not log "Error resolving tenant for host".

## Test cases

### Wave 1 — Host DB load on tenant requests

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-01 | ALFQ-TEN-REQ-001 | Verify JWT tenant status uses the resolved tenant and does not query Host DB again | Main | Tenant already resolved as Active by middleware | 1. Send an authenticated tenant API request.<br>2. Confirm JWT `OnTokenValidated` does not call `ITenantStatusValidationService.IsTenantActiveAsync`. | 1. Active tenant requests are authenticated.<br>2. Suspended or inactive tenant tokens are rejected with session-invalidated.<br>3. Host requests with no tenant still authenticate. | - | `Alforqan/Alforqan.Core/Features/Tenants/Helpers/TenantJwtStatusHelper.cs`, `Alforqan/Alforqan.Api/Program.cs` | Active tenant JWT; suspended tenant JWT |
| TC-02 | ALFQ-TEN-REQ-002 | Verify LastActivityAt is not written to Host DB on every request | Main | Same tenant, two requests within 2 minutes | 1. Send a tenant request (triggers last-activity write).<br>2. Send another request for the same tenant within 2 minutes.<br>3. Send a request after 2 minutes. | 1. First request may write `LastActivityAt`.<br>2. Second request inside 2 minutes does not write.<br>3. Request after 2 minutes may write again. | - | `Alforqan/Alforqan.Shared/HelperClasses/TenantLastActivityGate.cs`, `Alforqan/Alforqan.Service/Implementation/Tenants/Services/TenantMetricsWriter.cs` | One active tenant |
| TC-03 | ALFQ-TEN-REQ-003 | Verify a cancelled request is not logged as tenant resolution failure | Edge | Client aborts while Host DB is slow | 1. Start a tenant request.<br>2. Cancel the client before SQL finishes.<br>3. Read API logs and the HTTP status. | 1. Log does not say "Error resolving tenant for host".<br>2. Response is not HTTP 500 with "A task was canceled."<br>3. Status is 499 when the response has not started. | - | `Alforqan/Alforqan.Shared/HelperClasses/ClientAbort.cs`, `Alforqan/Alforqan.Api/Middlewares/ErrorHandlerMiddleware.cs`, `Alforqan/Alforqan.Api/Middlewares/TenantResolutionMiddleware.cs` | Slow Host DB or cancelled browser tab |
