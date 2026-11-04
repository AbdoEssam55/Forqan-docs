# Single-Tenant Mode — Frontend Test Cases

## Metadata
- Project: `alforqan`
- Feature: `single-tenant-mode`
- Layer: `frontend`
- Routes: `/{locale}/host/*`, `/{locale}/dashboard`, `/{locale}/tenants`, `/{locale}`, `/{locale}/{tenantSlug}`
- APIs: -
- Related code: `src/middleware.ts`, `src/lib/config/env.ts`, `src/app/[locale]/(public)/page.tsx`
- Related docs: `docs/single-tenant-strategy.md`

## Scope
- With `NEXT_PUBLIC_MULTI_TENANCY_MODE=SingleTenant`, the host admin UI is unreachable.
- Users enter the institute app via the fixed tenant slug (`NEXT_PUBLIC_DEFAULT_TENANT_SLUG`, default `alforqan`).
- Setting mode to `MultiHost` restores host admin routes.

## Test scenarios
- Host login and host dashboard URLs redirect away in single-tenant mode.
- Home page offers entry into the fixed tenant app.
- MultiHost mode allows host UI again.

## Test cases

### Host UI hidden in single-tenant mode

| ID | Key | Case | Type | Preconditions | Steps | Expected | Related code |
|----|-----|------|------|---------------|-------|----------|--------------|
| TC-01 | ALF-STM-UI-001 | Verify host auth and admin URLs redirect to the locale home in single-tenant mode | Main | `NEXT_PUBLIC_MULTI_TENANCY_MODE=SingleTenant`.<br>Frontend restarted. | 1. Open `/{locale}/host/auth/login`.<br>2. Open `/{locale}/dashboard`.<br>3. Open `/{locale}/tenants`. | 1. Each URL redirects to `/{locale}` (locale home).<br>2. The host login form and host admin shell are not shown. | `src/middleware.ts` |
| TC-02 | ALF-STM-UI-002 | Verify the home page links into the fixed tenant app | Main | `NEXT_PUBLIC_MULTI_TENANCY_MODE=SingleTenant`.<br>`NEXT_PUBLIC_DEFAULT_TENANT_SLUG=alforqan`. | 1. Open `/{locale}`.<br>2. Click "Enter platform" / "دخول المنصة". | 1. The home page shows the Alforqan product title.<br>2. The link navigates to `/{locale}/alforqan` (or the configured default slug). | `src/app/[locale]/(public)/page.tsx`, `src/lib/config/env.ts` |
| TC-03 | ALF-STM-UI-003 | Verify host UI is reachable again when mode is "MultiHost" | Edge | `NEXT_PUBLIC_MULTI_TENANCY_MODE=MultiHost`.<br>Frontend restarted. | 1. Open `/{locale}/host/auth/login`. | 1. The host login page loads (no redirect to locale home). | `src/middleware.ts` |
