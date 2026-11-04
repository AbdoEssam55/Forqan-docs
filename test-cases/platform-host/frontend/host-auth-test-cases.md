# Platform host auth — Frontend Test Cases

## Metadata
- Project: `alforqan`
- Feature: `platform-host`
- Layer: `frontend`
- Routes: `/{locale}/host/auth/login`, `/{locale}/host/auth/forgot-password`
- APIs: `POST /API/V01/Host/Authentication/Login`
- Related code: `src/features/host/auth/components/login-form.tsx`, `src/app/[locale]/host/auth/login/page.tsx`, `src/components/auth/host-auth-guard.tsx`
- Related docs: `docs/multi-tenant-platform-overview.md`, `Backend/docs/features-explanation-docs/host-identity-system.md`, `Backend/docs/features-explanation-docs/per-tenant-rate-limiting-feature.md`

## Scope

- Host (platform admin) sign-in and sign-out for the multi-tenant admin portal.
- Entry point before any host demo (tenants, modules, plans, branding).

## Test pages

- `/{locale}/host/auth/login`
- Any authenticated host route (for example `/{locale}/dashboard`)

## Test scenarios

- Host admin signs in with valid credentials and lands on the dashboard.
- Invalid credentials show an error without signing in.
- Signed-in host admin can sign out and return to the login page.
- Too many failed login attempts show a short (~2 minute) wait before retry.

## Delivery waves

| Wave | Intent | Keys | Depends on |
| ---- | ------ | ---- | ---------- |
| 1 | Host portal authentication | `PLAT-HOST-AUTH-001`–`004` | — |

Run this file first in the platform-host demo order.

## Test cases

### Wave 1 — Host sign-in and sign-out

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-01 | PLAT-HOST-AUTH-001 | Verify a host admin can sign in and reach the host dashboard | Main | A host user account exists with permission to view the dashboard | 1. Open `/{locale}/host/auth/login` (for example `/en/host/auth/login`).<br>2. Confirm the page title area shows "Host Admin" and subtitle "Sign in to access your account".<br>3. Enter a valid host email or username and password.<br>4. Choose "Sign In". | 1. A success toast shows "Login successful" (or equivalent).<br>2. The browser navigates to the host dashboard.<br>3. The page heading shows "Host Dashboard". | - | `src/features/host/auth/components/login-form.tsx` | Host user with dashboard access |
| TC-02 | PLAT-HOST-AUTH-002 | Verify invalid host credentials are rejected on login | Edge | No active host session | 1. Open the host login page.<br>2. Enter a valid-looking email and an incorrect password.<br>3. Choose "Sign In". | 1. The user remains on the login page (not the dashboard).<br>2. An error message is shown; no "Login successful" toast appears. | - | `src/features/host/auth/components/login-form.tsx` | Wrong password for a known host user |
| TC-03 | PLAT-HOST-AUTH-003 | Verify a signed-in host admin can sign out from the host portal | Main | User completed `PLAT-HOST-AUTH-001` and is on any host page with the sidebar visible | 1. In the host sidebar, choose "Sign Out". | 1. The browser navigates to the host login page.<br>2. Opening `/{locale}/dashboard` without signing in again redirects to the host login page. | PLAT-HOST-AUTH-001 | `src/components/layout/host-navigation.tsx`, `src/middleware.ts` | — |
| TC-04 | PLAT-HOST-AUTH-004 | Verify too many failed host logins show a wait of about 2 minutes | Edge | No active host session; prefer a fresh IP bucket or wait out any prior lockout | 1. Open `/{locale}/host/auth/login`.<br>2. Submit wrong password repeatedly until a rate-limit or lockout message appears (host login allows up to 5 attempts per 2 minutes).<br>3. Read the countdown or "try again" message. | 1. Further login attempts are blocked with a clear error.<br>2. The wait shown is about **2 minutes** (not ~5–6 minutes).<br>3. After the wait ends, a valid login can succeed again. | - | `src/lib/utils/error-handler.ts`, `src/features/host/auth/components/login-form.tsx` | Wrong password for a known host user |
