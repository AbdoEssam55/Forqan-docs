# Platform host dashboard — Frontend Test Cases

## Metadata
- Project: `alforqan`
- Feature: `platform-host`
- Layer: `frontend`
- Routes: `/{locale}/dashboard`
- APIs: Host tenant statistics endpoints used by dashboard
- Related code: `src/app/[locale]/(host)/dashboard/dashboard-content.tsx`, `src/components/layout/host-navigation.tsx`
- Related docs: `docs/multi-tenant-platform-overview.md`, `docs/multi-tenant-platform-scope-and-roadmap.md`

## Scope

- Host dashboard overview cards and quick navigation into tenant management.
- Demonstrates platform-wide tenant counts and plan distribution to a prospective client.

## Test pages

- `/{locale}/dashboard`

## Test scenarios

- Dashboard shows tenant statistics and plan distribution.
- Quick actions navigate to tenant list and tenant creation.

## Delivery waves

| Wave | Intent | Keys | Depends on (cross-file) |
| ---- | ------ | ---- | ----------------------- |
| 1 | Host dashboard overview | `PLAT-HOST-DSH-001`–`002` | `PLAT-HOST-AUTH-001` |

## Test cases

### Wave 1 — Dashboard overview

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-01 | PLAT-HOST-DSH-001 | Verify the host dashboard shows tenant statistics and plan distribution | Main | Signed in as host admin with permission to view tenants; at least one tenant exists in the system | 1. Open `/{locale}/dashboard`.<br>2. Read the page heading and statistic cards. | 1. The heading shows "Host Dashboard".<br>2. Cards include "Total Tenants", "Active Tenants", "Suspended Tenants", and "Inactive Tenants".<br>3. A "Plan Distribution" section lists plan names such as "Starter", "Standard", or "Enterprise". | PLAT-HOST-AUTH-001 | `src/app/[locale]/(host)/dashboard/dashboard-content.tsx` | Host admin with ViewTenants claim |
| TC-02 | PLAT-HOST-DSH-002 | Verify dashboard quick actions open tenant management | Main | Same as `PLAT-HOST-DSH-001` | 1. On the host dashboard, choose "View All Tenants".<br>2. Use the browser back control, then choose "Create New Tenant". | 1. "View All Tenants" opens the tenants list with heading "Tenants Management".<br>2. "Create New Tenant" opens the create-tenant wizard with heading "Create New Tenant". | PLAT-HOST-DSH-001 | `src/app/[locale]/(host)/dashboard/dashboard-content.tsx`, `src/app/[locale]/(host)/tenants/tenants-list-content.tsx` | — |
