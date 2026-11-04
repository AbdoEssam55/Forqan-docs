# Platform tenant onboarding — Frontend Test Cases

## Metadata
- Project: `alforqan`
- Feature: `platform-onboarding`
- Layer: `frontend`
- Routes: `/{locale}/tenants/create`, `/{locale}/tenants/[id]`, `/{locale}/{tenantSlug}/auth/login`
- APIs: `POST /API/V01/Host/Tenants`; optional `ITenantProvisioningService`
- Related code: `src/app/[locale]/(host)/tenants/create/tenant-create-content.tsx`, `Backend/Alforqan/Alforqan.Service/Implementation/Tenants/Services/TenantService.cs`, `Backend/Alforqan/Alforqan.Service/Abstractions/Tenants/IServices/ITenantProvisioningService.cs`
- Related docs: `Backend/docs/technical-docs/tenant-provisioning-guide.md`, `Backend/docs/features-explanation-docs/tenant-provisioning-explained.md`

## Scope

- How a new tenant is onboarded: host wizard, optional automated DB provisioning, institute first login.
- Includes AI-assisted onboarding narrative (Cursor/agent follows docs and project rules).
- For demo create, use a throwaway tenant code or staging environment.

## Test pages

- Host create-tenant wizard
- Tenant institute login after create
- Provisioning docs

## Test scenarios

- Host admin runs create-tenant wizard end-to-end.
- Explain provisioning hook versus manual DB setup.
- Demonstrate AI-assisted flow (prompt → host row + DB + login).

## Delivery waves

| Wave | Intent | Keys |
| ---- | ------ | ---- |
| 1 | Host UI onboarding | `PLAT-ONB-001`–`003` |
| 2 | Provisioning and AI narrative | `PLAT-ONB-004`–`005` |

## Test cases

### Wave 1 — Host UI onboarding

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-01 | PLAT-ONB-001 | Verify host admin can complete the create-tenant wizard on staging | Main | Signed in as host admin with CreateTenant claim; staging environment; unique throwaway code prepared | 1. Open `/{locale}/tenants/create`.<br>2. Step "Basic Information": enter name, code, primary domain (e.g. `localhost` or staging domain).<br>3. Step "Contact Information": fill contact fields.<br>4. Step "Configuration": choose plan, support plan, connection string (or placeholder if provisioning registered).<br>5. Optionally fill initial tenant admin email/password.<br>6. Choose "Create Tenant". | 1. Success toast shows "Tenant created successfully" (or equivalent).<br>2. New tenant appears on the tenants list.<br>3. Tenant detail opens or is reachable with status "Active". | PLAT-HOST-AUTH-001 | `src/app/[locale]/(host)/tenants/create/tenant-create-content.tsx`, `src/features/tenants/components/tenant-form.tsx` | Unique code e.g. `DEMO-ONB-01` |
| TC-02 | PLAT-ONB-002 | Verify default modules from the chosen plan are applied to the new tenant | Main | Tenant from `PLAT-ONB-001` exists | 1. Open the new tenant detail.<br>2. Select the "Modules" tab.<br>3. Compare enabled modules to the plan's default modules. | 1. "Module Management" lists modules matching the plan defaults (at minimum "Core" enabled).<br>2. Modules enabled by plan match what is shown on the plan form "Default modules". | PLAT-ONB-001 | `src/app/[locale]/(host)/tenants/[id]/tenant-detail-content.tsx`, `Backend/Alforqan/Alforqan.Service/Implementation/Tenants/Services/TenantService.cs` | Same new tenant |
| TC-03 | PLAT-ONB-003 | Verify the initial tenant admin can sign in to the new institute | Main | `PLAT-ONB-001` created tenant with initial admin credentials | 1. Open `/{locale}/{tenantSlug}/auth/login` for the new tenant (use code or slug from create).<br>2. Sign in with the initial admin credentials from the wizard.<br>3. Confirm dashboard loads. | 1. Institute login succeeds.<br>2. Dashboard shows "Overview of your institute".<br>3. No "Failed to load institute settings." banner appears. | PLAT-ONB-001 | `src/features/alforqan/components/tenant-login-form.tsx` | Initial admin from wizard |

### Wave 2 — Provisioning and AI-assisted onboarding

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-04 | PLAT-ONB-004 | Verify the presenter explains the provisioning hook after tenant create | Main | `Backend/docs/technical-docs/tenant-provisioning-guide.md` open | 1. Show `ITenantProvisioningService` interface in docs or code.<br>2. Explain `TenantService.AddAsync` calls provisioning after save when registered.<br>3. Describe implementation steps: create DB, run tenant migrations, update connection string.<br>4. Note platform ships the hook; product registers the implementation. | 1. Interface name and call timing are stated.<br>2. Narration distinguishes platform hook versus product-specific migration logic.<br>3. Viewer understands manual DB setup is optional when provisioning is implemented. | PLAT-ONB-001 | `Backend/Alforqan/Alforqan.Service/Abstractions/Tenants/IServices/ITenantProvisioningService.cs`, `Backend/docs/technical-docs/tenant-provisioning-guide.md` | — |
| TC-05 | PLAT-ONB-005 | Verify the presenter demonstrates AI-assisted tenant onboarding workflow | Main | Cursor or agent session; project rules and provisioning doc available | 1. Show a written prompt to the agent (example: "Add staging tenant Acme with code acme, domain acme.localhost, create DB and run migrations").<br>2. Narrate what the agent should do: follow `tenant-provisioning-guide`, host seed patterns, update Host DB row.<br>3. Show result: tenant visible on host list and institute login works (or explain staging limitation if not run live). | 1. A concrete agent prompt is visible or read aloud.<br>2. Narration references project docs—not magic commands—as the source of truth.<br>3. End state is described as: Host row + tenant DB + working institute login.<br>4. Narration mentions human review before production onboarding. | PLAT-ONB-004 | `Backend/.agent/rules/project-standard-rules.md`, `Backend/docs/technical-docs/tenant-provisioning-guide.md` | Staging only |
