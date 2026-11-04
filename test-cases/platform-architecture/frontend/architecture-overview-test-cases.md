# Platform architecture overview — Frontend Test Cases

## Metadata
- Project: `alforqan`
- Feature: `platform-architecture`
- Layer: `frontend`
- Routes: Host and tenant apps; docs walkthrough
- APIs: -
- Related code: `Backend/Platform/`, `Frontend/alforqan/src/app/[locale]/(host)/`, `Frontend/alforqan/src/app/[locale]/(tenant)/`
- Related docs: `docs/multi-tenant-platform-overview.md`, `docs/multi-tenant-platform-scope-and-roadmap.md`, `Backend/docs/architecture/deployment-architecture.md`

## Scope

- Sales walkthrough: why the multi-tenant architecture is structured as platform + product.
- Record with microphone while showing docs, solution layout, and short live demos where noted.
- Not Alforqan business flows (halaqat, sard, etc.).

## Test pages

- `docs/multi-tenant-platform-overview.md`
- `docs/multi-tenant-platform-scope-and-roadmap.md`
- Host app and tenant app (for short live proof)

## Test scenarios

- Explain platform vs product separation.
- Explain one codebase, DB-per-tenant, host vs tenant auth, and module gating.
- Tie architecture benefits to client needs (isolation, sell modules, flexible deploy).

## Delivery waves

| Wave | Intent | Keys |
| ---- | ------ | ---- |
| 1 | Architecture narrative for client | `PLAT-ARCH-001`–`006` |

Record after Phase 1 UI demos when the client understands basic screens.

## Test cases

### Wave 1 — Architecture narrative

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-01 | PLAT-ARCH-001 | Verify the presenter explains platform versus product scope using the overview doc | Main | `docs/multi-tenant-platform-overview.md` is open | 1. Open the overview doc section "Platform vs Product".<br>2. Point to the table: platform provides tenant model, resolution, modules; product adds business entities.<br>3. In the repo, briefly show `Backend/Platform/` versus `Backend/Alforqan/` (or product folder). | 1. The narration states the client buys the **platform engine**, not a fixed business vertical.<br>2. The table visible on screen lists Host DB, tenant resolution, and TenantConfig on the platform side.<br>3. The viewer sees product code is a separate layer on top. | - | `Backend/Platform/` | — |
| TC-02 | PLAT-ARCH-002 | Verify the presenter explains one codebase with multiple deployment options | Main | Overview or scope doc open at "Deployment" | 1. Open `docs/multi-tenant-platform-scope-and-roadmap.md` section 4 (Deployment Model).<br>2. Describe Option A (one backend, many tenants) and Option B (dedicated backend per client).<br>3. State that both use the **same build artifact**. | 1. Both options are named on screen or in narration.<br>2. Narration says a client can start shared and move to dedicated without rewriting code.<br>3. No claim that separate products require separate codebases. | - | `docs/multi-tenant-platform-scope-and-roadmap.md` | — |
| TC-03 | PLAT-ARCH-003 | Verify the presenter demonstrates database-per-tenant isolation | Main | Host tenants list available; at least two tenants exist | 1. On host, open tenant detail for two different tenants.<br>2. Explain each tenant row stores its own connection string in the Host DB.<br>3. State tenant business data lives in **separate databases**, not shared tables with a TenantId column. | 1. Two distinct tenants are visible on screen.<br>2. Narration explains Host DB holds metadata only; tenant DBs hold institute data.<br>3. Narration mentions stronger isolation than single-DB multi-tenancy. | PLAT-HOST-TEN-003 | `Backend/docs/DB entities/tenant-entity.md` | Two demo tenants |
| TC-04 | PLAT-ARCH-004 | Verify the presenter explains tenant resolution by domain | Main | Scope or tenant-resolution doc available | 1. Open `Backend/docs/features-explanation-docs/tenant-resolution-feature.md` intro (or overview "Key Concepts").<br>2. Explain requests resolve tenant by Host header / primary domain.<br>3. Mention dev header `X-Tenant-Code` only for local testing if relevant. | 1. "Tenant resolution" and domain-based routing are stated clearly.<br>2. Narration says every tenant API request knows which tenant before using data.<br>3. Host routes (`/API/V01/Host/*`) are described as **not** tenant-scoped. | - | `Backend/docs/features-explanation-docs/tenant-resolution-feature.md` | — |
| TC-05 | PLAT-ARCH-005 | Verify the presenter demonstrates the auth split between host and institute users | Main | Host and tenant login pages reachable | 1. Open host login (`/en/host/auth/login`) — show "Host Admin".<br>2. Open institute login for a demo tenant — show "Institute Login".<br>3. Explain host users live in Host DB; tenant users live in that tenant's DB. | 1. Both login screens are shown in the recording.<br>2. Narration distinguishes platform admin versus institute user.<br>3. Narration states credentials are not shared across the two portals. | PLAT-HOST-AUTH-001, PLAT-TEN-AUTH-001 | `src/features/host/auth/components/login-form.tsx`, `src/features/alforqan/components/tenant-login-form.tsx` | Demo tenant slug |
| TC-06 | PLAT-ARCH-006 | Verify the presenter ties module gating to commercial packaging | Main | Module toggle and tenant nav demos done or ready | 1. On host, open Modules tab for a demo tenant.<br>2. On tenant app, show Reports in nav when enabled.<br>3. Disable Reports on host, refresh tenant app, show Reports hidden.<br>4. Summarize: modules = sellable feature packs enforced in API and UI. | 1. Host module toggles are visible.<br>2. Tenant nav changes when Reports is off.<br>3. Narration states backend `[RequiresModule]` returns forbidden when disabled.<br>4. Client understands features can be sold à la carte. | PLAT-HOST-MOD-002, PLAT-TEN-MOD-002 | `src/features/alforqan/config/tenant-nav.ts`, `src/features/tenant/components/module-gate.tsx` | Demo tenant |
