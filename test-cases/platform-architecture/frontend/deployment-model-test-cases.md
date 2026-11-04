# Platform deployment model — Frontend Test Cases

## Metadata
- Project: `alforqan`
- Feature: `platform-architecture`
- Layer: `frontend`
- Routes: -
- APIs: Host `/API/V01/Host/*`; tenant `/API/V01/*`
- Related code: `Backend/Alforqan/Alforqan.Api/appsettings.json`, `Backend/docs/architecture/deployment-architecture.md`
- Related docs: `docs/multi-tenant-platform-scope-and-roadmap.md`, `Backend/docs/architecture/deployment-architecture.md`

## Scope

- Sales walkthrough: how to publish **central host** stack versus **dedicated client** stack (separate backend + tenant DB).
- Steps use the deployment architecture doc and configuration examples; optional local two-port demo.
- Recording is narration + diagrams/docs; no production deploy required.

## Test pages

- `Backend/docs/architecture/deployment-architecture.md`
- `docs/multi-tenant-platform-scope-and-roadmap.md` (section 4)

## Test scenarios

- Central admin server + Host DB for platform management.
- Per-client tenant server + tenant DB without Host DB credentials.
- Same codebase artifact deployed with different configuration.

## Delivery waves

| Wave | Intent | Keys |
| ---- | ------ | ---- |
| 1 | Deployment narrative | `PLAT-DEP-001`–`006` |

## Test cases

### Wave 1 — Dedicated client deployment

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-01 | PLAT-DEP-001 | Verify the presenter shows the central host stack responsibilities | Main | Deployment architecture doc open | 1. Open `Backend/docs/architecture/deployment-architecture.md` diagram "Central Admin Server".<br>2. List responsibilities: host endpoints, tenant metadata, SystemAdmin portal.<br>3. Point to Host DB as single central metadata store. | 1. Diagram or bullet list is visible.<br>2. Narration names Host DB and host API prefix `/API/V01/Host/`.<br>3. Narration states this server is for **your company** operations, not end-client daily use. | - | `Backend/docs/architecture/deployment-architecture.md` | — |
| TC-02 | PLAT-DEP-002 | Verify the presenter explains the dedicated client tenant server model | Main | Same doc, "Client Tenant Servers" section | 1. Scroll to client server diagram.<br>2. Explain client backend serves tenant API and institute users only.<br>3. State client server has **no Host DB connection string** in configuration. | 1. Client server box is visible in the doc.<br>2. Narration says each client can have their own VM/container and SQL Server.<br>3. Narration emphasizes tenant DB is on the client side (or client-controlled hosting). | PLAT-DEP-001 | `Backend/docs/architecture/deployment-architecture.md` | — |
| TC-03 | PLAT-DEP-003 | Verify the presenter contrasts admin versus client appsettings configuration | Main | Doc section "Configuration Differences" | 1. Show admin `appsettings` example with `HostDb` and `HostEndpointsEnabled: true`.<br>2. Show client example with only `TenantDb` and no `HostDb`.<br>3. Explain same DLL, different config at deploy time. | 1. Both JSON snippets are visible (doc or your deployment templates).<br>2. Narration states configuration—not forked code—defines the role.<br>3. Client config explicitly omits host database credentials. | PLAT-DEP-002 | `Backend/docs/architecture/deployment-architecture.md` | — |
| TC-04 | PLAT-DEP-004 | Verify the presenter explains how host still manages a dedicated client's tenant row | Main | Host tenants list accessible | 1. On central host portal, open a client tenant record.<br>2. Show connection string / primary domain fields.<br>3. Explain host stores **where** the client DB lives; client server uses that DB at runtime (or fixed single-tenant config). | 1. Tenant detail is on screen.<br>2. Narration clarifies metadata is central; data is on client infrastructure.<br>3. Domain field links end users to the correct tenant on the client deployment. | PLAT-HOST-TEN-003 | `src/app/[locale]/(host)/tenants/[id]/tenant-detail-content.tsx` | Client tenant row |
| TC-05 | PLAT-DEP-005 | Verify the presenter describes security layers for split deployment | Main | Doc "Security Model" section | 1. Walk through layers: network block of host routes on client, no host DB creds, separate auth schemes.<br>2. Describe compromise scenario: client server breach does not expose all tenants' DBs. | 1. At least three security layers are named in narration.<br>2. Narration states SystemAdmin users exist only in Host DB.<br>3. Narration states other tenants are not reachable from one client's tenant DB. | PLAT-DEP-002 | `Backend/docs/architecture/deployment-architecture.md` | — |
| TC-06 | PLAT-DEP-006 | Verify the presenter summarizes publish order for a new dedicated client | Main | Narrator has internal checklist or doc | 1. List steps: provision tenant DB → run tenant migrations → deploy client backend+frontend → register tenant on central host with connection string and domain → smoke-test institute login.<br>2. Mention optional `ITenantProvisioningService` for automation. | 1. Steps are spoken in logical order (at least five steps).<br>2. Narration distinguishes **central host publish** from **client stack publish**.<br>3. Narration mentions same CI artifact can deploy to both roles. | PLAT-DEP-003, PLAT-DEP-004 | `Backend/docs/technical-docs/tenant-provisioning-guide.md` | — |
