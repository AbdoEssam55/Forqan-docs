# Platform host plans — Frontend Test Cases

## Metadata
- Project: `alforqan`
- Feature: `platform-host`
- Layer: `frontend`
- Routes: `/{locale}/plans`, `/{locale}/plans/create`, `/{locale}/plans/[id]`
- APIs: Host tenant plan endpoints
- Related code: `src/app/[locale]/(host)/plans/plans-list-content.tsx`, `src/app/[locale]/(host)/plans/plan-form.tsx`
- Related docs: `docs/multi-tenant-platform-overview.md`, `docs/multi-tenant-platform-scope-and-roadmap.md`

## Scope

- Tenant module plans (Starter / Standard / Enterprise style) and their default module bundles.
- Demonstrates how the platform ties subscription plans to enabled modules.

## Test pages

- `/{locale}/plans`
- `/{locale}/plans/[id]` or create flow

## Test scenarios

- Host admin views the plan catalog.
- Host admin opens a plan and sees default modules (Core, Reports, Payments (Basic)).

## Delivery waves

| Wave | Intent | Keys | Depends on (cross-file) |
| ---- | ------ | ---- | ----------------------- |
| 1 | Module plans overview | `PLAT-HOST-PLN-001`–`002` | `PLAT-HOST-AUTH-001` |

## Test cases

### Wave 1 — Module plans

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-01 | PLAT-HOST-PLN-001 | Verify the tenant plans list shows plan names and default modules | Main | Signed in as host admin with permission to view plans | 1. Open the sidebar "Plans" group and choose "Module Plans" (`/{locale}/plans`).<br>2. Review the table. | 1. The heading shows "Tenant Plans".<br>2. A "Create Plan" action is visible.<br>3. Columns include Name, Active, and Default modules.<br>4. At least one plan row lists default modules such as "Core" or "Reports". | PLAT-HOST-AUTH-001 | `src/app/[locale]/(host)/plans/plans-list-content.tsx` | — |
| TC-02 | PLAT-HOST-PLN-002 | Verify editing a plan shows default module checkboxes | Main | At least one editable plan exists | 1. From the plans list, open a plan for edit.<br>2. Find the "Default modules" section. | 1. The form shows "Plan name" and an "Active" control.<br>2. "Default modules" offers "Core", "Reports", and "Payments (Basic)" (or equivalent).<br>3. Saving changes shows a success confirmation without validation errors. | PLAT-HOST-PLN-001 | `src/app/[locale]/(host)/plans/plan-form.tsx` | Existing plan id |
