# Auth pages overhaul — Frontend Test Cases

## Metadata
- Project: `alforqan`
- Feature: `auth-ui-overhaul`
- Layer: `frontend`
- Routes: `/{locale}/sign-up`, `/{locale}/{tenantSlug}/auth/forgot-password`, `/{locale}/{tenantSlug}/auth/reset-password`, `/{locale}/{tenantSlug}/auth/change-password`, `/{locale}/{tenantSlug}/auth/complete-profile`
- APIs: tenant forgot/reset/change-password/complete-profile endpoints (behavior unchanged)
- Related code: `src/components/auth/auth-layout.tsx`, `src/components/auth/auth-card.tsx`, `src/components/auth/auth-success-panel.tsx`, `src/components/auth/password-strength-indicator.tsx`, `src/features/auth/components/sign-up-form.tsx`, `src/features/alforqan/components/tenant-forgot-password-form.tsx`, `src/features/alforqan/components/tenant-reset-password-form.tsx`, `src/features/alforqan/components/tenant-change-password-form.tsx`, `src/features/alforqan/components/tenant-complete-profile-form.tsx`, `src/components/brand/brand-logo.tsx`, `src/components/brand/brand-background.tsx`
- Related docs: `docs/technical/auth/auth-pages-overhaul.md`, `docs/technical/auth/login-page.md`, `docs/technical/brand-assets.md`

## Scope

Phase 2 UI overhaul for registration, password reset, and onboarding auth screens. Does not change API contracts.

## Delivery waves

| Wave | Intent | Keys | Depends on (cross-file) |
| ---- | ------ | ---- | ----------------------- |
| 1 | Registration + reset + onboarding UI | `ALFQ-AUTH-P2-001`–`007` | — |

## Test cases

### Wave 1 — Auth pages redesign

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-01 | ALFQ-AUTH-P2-001 | Verify sign-up page uses split-screen AuthLayout and AuthCard | Main | None | 1. Open `/{locale}/sign-up` at desktop width.<br>2. Inspect left brand panel and right form card. | 1. Left panel shows emerald/teal/cyan gradient with brand content.<br>2. Right panel shows AuthCard titled "Create Account" / "إنشاء حساب".<br>3. Form includes full name, email, password, confirm password, terms checkbox, and submit. | - | `src/app/[locale]/(auth)/sign-up/page.tsx`, `src/features/auth/components/sign-up-form.tsx` | - |
| TC-02 | ALFQ-AUTH-P2-002 | Verify password strength indicator updates while typing | Main | On sign-up page | 1. Focus the password field.<br>2. Type a weak password (e.g. `abc`).<br>3. Strengthen it with mixed case, digits, and a symbol. | 1. Strength bars appear under the password field.<br>2. Label moves through Weak → Fair/Good → Strong as the password improves. | - | `src/components/auth/password-strength-indicator.tsx`, `src/features/auth/components/sign-up-form.tsx` | Sample passwords |
| TC-03 | ALFQ-AUTH-P2-003 | Verify terms checkbox is required before create account | Edge | On sign-up page | 1. Fill valid name/email/password/confirm.<br>2. Leave terms unchecked and submit.<br>3. Check terms and submit again. | 1. Unchecked submit shows the terms validation error and does not toast success.<br>2. Checked submit shows "Self-registration unavailable" / "التسجيل الذاتي غير متاح" toast (API not wired). | - | `src/features/auth/components/sign-up-form.tsx` | - |
| TC-04 | ALFQ-AUTH-P2-004 | Verify tenant forgot-password page uses AuthLayout and sends reset code UI | Main | Demo tenant slug | 1. Open `/{locale}/{tenantSlug}/auth/forgot-password`.<br>2. Confirm split-screen layout and AuthCard title "Forgot Password".<br>3. Enter a valid email and submit. | 1. Layout matches login redesign (brand left, form right).<br>2. On success, success panel/message appears then redirect to reset-password with email query. | - | `src/features/alforqan/components/tenant-forgot-password-form.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/auth/forgot-password/page.tsx` | Demo tenant email |
| TC-05 | ALFQ-AUTH-P2-005 | Verify tenant reset-password page shows strength meter and AuthCard | Main | Arrived from forgot flow or open reset with `?email=` | 1. Open reset-password page.<br>2. Confirm AuthCard fields: email, code, new password, confirm.<br>3. Type a new password. | 1. Page uses AuthLayout + AuthCard.<br>2. Password strength indicator appears under new password.<br>3. Back to Sign In link is visible. | - | `src/features/alforqan/components/tenant-reset-password-form.tsx` | Demo email |
| TC-06 | ALFQ-AUTH-P2-006 | Verify forced change-password page uses AuthLayout + AuthCard | Main | User redirected to change-password onboarding | 1. Open `/{locale}/{tenantSlug}/auth/change-password`.<br>2. Inspect layout and card title. | 1. Split-screen AuthLayout is shown.<br>2. AuthCard shows the forced-change title/subtitle from onboarding copy.<br>3. Current/new/confirm password fields and submit are visible. | - | `src/features/alforqan/components/tenant-change-password-form.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/auth/change-password/page.tsx` | Onboarding user |
| TC-07 | ALFQ-AUTH-P2-007 | Verify complete-profile page uses wider AuthLayout and AuthCard | Main | User with incomplete profile | 1. Open `/{locale}/{tenantSlug}/auth/complete-profile` at desktop width.<br>2. Inspect form container width and card. | 1. AuthLayout brand panel is visible.<br>2. Form column is wider than login (max-w-2xl).<br>3. AuthCard shows complete-profile title/subtitle and the existing profile fields. | - | `src/features/alforqan/components/tenant-complete-profile-form.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/auth/complete-profile/page.tsx` | Incomplete-profile user |
