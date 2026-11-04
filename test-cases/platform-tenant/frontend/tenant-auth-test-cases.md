# Platform tenant auth — Frontend Test Cases

## Metadata
- Project: `alforqan`
- Feature: `platform-tenant`
- Layer: `frontend`
- Routes: `/{locale}/{tenantSlug}/auth/login`
- APIs: `POST /API/V01/Authentication/Login` (tenant-scoped)
- Related code: `src/features/alforqan/components/tenant-login-form.tsx`, `src/components/auth/auth-card.tsx`, `src/components/auth/social-login-buttons.tsx`, `src/lib/auth/remembered-tenant-login.ts`, `src/components/auth/tenant-auth-guard.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/layout.tsx`, `src/lib/auth/auth-helpers.ts`
- Related docs: `docs/technical/auth/login-page.md`, `docs/technical/host/host-auth-guide.md`, `docs/explanation/host/host-auth-explained.md`, `docs/explanation/api/tenant-api-client-explained.md`, `docs/multi-tenant-platform-overview.md`

## Scope

- Tenant (institute) user sign-in and sign-out — separate from host admin auth.
- Demonstrates the auth split: platform admins use host login; institute users use tenant login.

## Test pages

- `/{locale}/{tenantSlug}/auth/login`
- `/{locale}/{tenantSlug}` (dashboard after login)

## Test scenarios

- Tenant user signs in and reaches the institute dashboard.
- Invalid tenant credentials are rejected.
- Tenant user signs out from the institute app.
- Too many failed login attempts show a short (~2 minute) wait before retry.

## Delivery waves

| Wave | Intent | Keys | Depends on (cross-file) |
| ---- | ------ | ---- | ----------------------- |
| 1 | Tenant institute authentication | `PLAT-TEN-AUTH-001`–`007` | — |

Run before tenant module gating and branding cases. Use a demo tenant slug and tenant user from your seed data.

## Test cases

### Wave 1 — Tenant sign-in and sign-out

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-01 | PLAT-TEN-AUTH-001 | Verify a tenant user can sign in to the institute portal | Main | A tenant user exists for the demo tenant; user is signed out of the tenant app | 1. Open `/{locale}/{tenantSlug}/auth/login` for the demo tenant.<br>2. Confirm the card title shows "Institute Login" and subtitle "Sign in to your institute account".<br>3. Enter valid tenant credentials and choose "Sign In". | 1. A success toast appears.<br>2. The browser navigates to the tenant dashboard.<br>3. The page shows "Dashboard" with subtitle "Overview of your institute". | - | `src/features/alforqan/components/tenant-login-form.tsx` | Demo tenant slug; tenant user email/password |
| TC-02 | PLAT-TEN-AUTH-002 | Verify invalid tenant credentials are rejected | Edge | No active tenant session | 1. Open the institute login page for the demo tenant.<br>2. Enter a valid email and wrong password.<br>3. Choose "Sign In". | 1. The user stays on the login page.<br>2. An error is shown; the dashboard does not load. | - | `src/features/alforqan/components/tenant-login-form.tsx` | Wrong password |
| TC-03 | PLAT-TEN-AUTH-003 | Verify a signed-in tenant user can sign out | Main | User completed `PLAT-TEN-AUTH-001` | 1. Open the institute sidebar.<br>2. Choose "Sign Out". | 1. The browser navigates to the institute login page for that tenant.<br>2. Opening the dashboard URL without signing in redirects to login. | PLAT-TEN-AUTH-001 | `src/features/alforqan/components/tenant-navigation.tsx`, `src/components/auth/tenant-auth-guard.tsx` | Demo tenant slug |
| TC-04 | PLAT-TEN-AUTH-004 | Verify a successful tenant login stays on the dashboard and does not bounce back to login | Main | A tenant user exists; app may be served over HTTP or HTTPS | 1. Open `/{locale}/{tenantSlug}/auth/login`.<br>2. Enter valid tenant credentials and choose "Sign In".<br>3. Wait 3 seconds on the dashboard without refreshing.<br>4. In DevTools Application → Local Storage, confirm `tenant_auth_token` and `tenant_refresh_token` exist.<br>5. Confirm the next API call (e.g. OnboardingStatus) sends `Authorization: Bearer ...` and is not 401. | 1. A success toast appears.<br>2. The browser navigates to the tenant dashboard.<br>3. The user remains on the dashboard (no redirect back to login).<br>4. Auth tokens are present in localStorage after login.<br>5. Authenticated API calls succeed with a Bearer token. | - | `src/lib/auth/auth-helpers.ts`, `src/features/alforqan/components/tenant-login-form.tsx`, `src/components/auth/tenant-auth-guard.tsx` | Demo tenant slug; tenant user email/password |
| TC-05 | PLAT-TEN-AUTH-005 | Verify too many failed tenant logins show a wait of about 2 minutes | Edge | No active tenant session; prefer a fresh IP/tenant bucket or wait out any prior lockout | 1. Open the institute login page for the demo tenant.<br>2. Submit wrong password repeatedly until a rate-limit or lockout message appears (tenant login allows up to 10 attempts per 2 minutes).<br>3. Read the countdown or "try again" message. | 1. Further login attempts are blocked with a clear error (not a silent failure).<br>2. The wait shown is about **2 minutes** (not ~5–6 minutes).<br>3. After the wait ends, a valid login can succeed again. | - | `src/lib/utils/error-handler.ts`, `src/features/alforqan/components/tenant-login-form.tsx` | Wrong password; demo tenant |
| TC-06 | PLAT-TEN-AUTH-006 | Verify social login buttons are only shown when tenant providers are configured | Edge | No active tenant session; app running with and without `NEXT_PUBLIC_TENANT_SOCIAL_PROVIDERS` | 1. Open `/{locale}/{tenantSlug}/auth/login` with `NEXT_PUBLIC_TENANT_SOCIAL_PROVIDERS` unset.<br>2. Confirm no "Or continue with" divider or social provider buttons appear.<br>3. Set `NEXT_PUBLIC_TENANT_SOCIAL_PROVIDERS=google,microsoft` and reload login page.<br>4. Choose any social button. | 1. When the variable is unset, only username/email + password login is shown.<br>2. When the variable is set, social buttons appear under the main submit button.<br>3. Clicking a social button shows the localized "Social login unavailable" message and keeps user on login page. | - | `src/components/auth/social-login-buttons.tsx`, `src/features/alforqan/components/tenant-login-form.tsx` | Demo tenant slug; optional env var override |
| TC-07 | PLAT-TEN-AUTH-007 | Verify "Remember me" stores the username for the next visit | Main | No active tenant session; prefer clearing Local Storage key `tenant_login_remembered_user:{tenantSlug}` first | 1. Open `/{locale}/{tenantSlug}/auth/login`.<br>2. Enter a valid username/email and password.<br>3. Check "Remember me".<br>4. Choose "Sign In" and wait for success redirect.<br>5. Sign out.<br>6. Open the institute login page again. | 1. After sign-out, returning to login shows the previous username/email already filled.<br>2. "Remember me" is checked again.<br>3. Password remains empty (password is never remembered).<br>4. Local Storage contains `tenant_login_remembered_user:{tenantSlug}` with the username only. | PLAT-TEN-AUTH-001 | `src/lib/auth/remembered-tenant-login.ts`, `src/features/alforqan/components/tenant-login-form.tsx` | Demo tenant slug; tenant user email/password |
