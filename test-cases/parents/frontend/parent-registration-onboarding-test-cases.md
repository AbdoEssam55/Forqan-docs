# Parents — Registration & first-login onboarding (Frontend)

## Metadata
- Project: `alforqan`
- Feature: `parents`
- Layer: `frontend`
- Routes: `/{tenantSlug}/parents/create`, `/{tenantSlug}/auth/change-password`, `/{tenantSlug}/auth/complete-profile`, `/{tenantSlug}/auth/reset-password`
- APIs: `POST /API/V01/Parents/create-with-user`, `GET /API/V01/Authentication/OnboardingStatus`, `GET /API/V01/Authentication/MyProfile`, `PUT /API/V01/Authentication/MyProfile`, `PATCH /API/V01/Authentication/ChangePassword`, `POST /API/V01/Authentication/ResetPassword`
- Related code: `src/features/alforqan/schemas/parent-create-wizard-schema.ts`, `src/features/alforqan/components/parent-create-wizard-form.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/parents/create/parent-create-content.tsx`, `src/components/auth/tenant-auth-guard.tsx`, `src/features/alforqan/components/tenant-complete-profile-form.tsx`, `src/features/alforqan/components/tenant-change-password-form.tsx`, `src/features/alforqan/components/tenant-reset-password-form.tsx`, `src/features/users/components/admin-set-password-form.tsx`, `src/lib/validation/tenant-identity-rules.ts`, `src/lib/validation/nationality-contact-rules.ts`
- Related docs: `docs/technical/tenant/parent-registration-onboarding.md`, `docs/technical/tenant/frontend-backend-validation-parity.md`, `Backend/docs/technical-docs/password-policy-by-role.md`

## Scope
Admin two-step create-parent wizard (minimal login + optional profile with simple password) and parent first-login gates (force password change, complete profile, optional one-time username and secondary phone).

## Test pages
- `/{tenantSlug}/parents/create`
- `/{tenantSlug}/parents/{id}` (admin set password)
- `/{tenantSlug}/auth/login`
- `/{tenantSlug}/auth/change-password`
- `/{tenantSlug}/auth/complete-profile`
- `/{tenantSlug}/auth/reset-password`

## Delivery waves
| Wave | Intent | Keys |
| ---- | ------ | ---- |
| 1 | Admin create wizard + simple password | `ALFQ-PAR-REG-001`–`011` |
| 2 | First-login gates | `ALFQ-PAR-REG-012`–`014` |

## Test cases

### Wave 1 — Admin create wizard & password policy

| ID | Key | Case | Steps | Expected | Related code | Requires |
| -- | --- | ---- | ----- | -------- | ------------ | -------- |
| TC-01 | ALFQ-PAR-REG-001 | Verify simple password `abcdef` is accepted when creating a parent | 1. Sign in as tenant admin with create-parent permission.<br>2. Open `/{tenantSlug}/parents/create`.<br>3. Enter email, username, password `abcdef` + confirm on Admin fields.<br>4. Click Next and use "Skip and save". | 1. No digit/upper/symbol complexity error on Password.<br>2. Parent is created successfully. | `src/features/alforqan/schemas/parent-create-wizard-schema.ts` | - |
| TC-02 | ALFQ-PAR-REG-002 | Verify password shorter than 6 characters is rejected on parent create | 1. On parent create enter password `abcde` with matching confirm.<br>2. Try Save. | 1. Password shows a minimum-length error.<br>2. Parent is not created. | `src/features/alforqan/schemas/parent-create-wizard-schema.ts` | - |
| TC-03 | ALFQ-PAR-REG-003 | Verify password longer than 256 characters is rejected on parent create | 1. On parent create enter a 257-character password with matching confirm.<br>2. Try Save. | 1. Password shows a maximum-length error (256).<br>2. Parent is not created. | `src/features/alforqan/schemas/parent-create-wizard-schema.ts` | - |
| TC-04 | ALFQ-PAR-REG-004 | Verify admin can set a simple password on parent detail | 1. Open a parent detail page as admin.<br>2. In set-password, enter `parent1` + confirm.<br>3. Save. | 1. No complexity error.<br>2. Success message appears.<br>3. Parent can sign in with the new password. | `src/features/users/components/admin-set-password-form.tsx`, `src/features/users/schemas/admin-set-password-schema.ts` | ALFQ-PAR-REG-001 |
| TC-05 | ALFQ-PAR-REG-005 | Verify parent can change password to a simple password | 1. Sign in as a parent.<br>2. Open change-password (or forced change).<br>3. Set new password `newpass` + confirm. | 1. No complexity error.<br>2. Password change succeeds. | `src/features/alforqan/components/tenant-change-password-form.tsx` | ALFQ-PAR-REG-001 |
| TC-06 | ALFQ-PAR-REG-006 | Verify parent reset-password accepts simple password | 1. Request forgot-password code for a parent email.<br>2. On reset page enter code and new password `reset12` + confirm.<br>3. Submit. | 1. No complexity error on the form.<br>2. Reset succeeds and parent can sign in. | `src/features/alforqan/components/tenant-reset-password-form.tsx` | ALFQ-PAR-REG-001 |
| TC-07 | ALFQ-PAR-REG-007 | Verify the create-parent wizard shows "Admin fields" then "Optional profile" | 1. Sign in as tenant admin with create-parent permission.<br>2. Open `/{tenantSlug}/parents/create`.<br>3. Read the step labels. | 1. Step 1 is labeled "Admin fields" / Arabic equivalent.<br>2. Step 2 is labeled "Optional profile" / Arabic equivalent.<br>3. There is no separate step requiring full name, first name, last name, or date of birth on create. | `src/features/alforqan/components/parent-create-wizard-form.tsx` | - |
| TC-08 | ALFQ-PAR-REG-008 | Verify admin can create a parent with only admin fields and skip optional profile | 1. On step 1 enter email, username, password `abcdef` + confirm, leave Active on.<br>2. Leave "Require password change on first login" unchecked.<br>3. Click Next.<br>4. On step 2 leave all fields empty.<br>5. Click "Skip and save". | 1. Parent is created successfully.<br>2. No validation error forces first name, last name, date of birth, nationality, phone, or ID for the admin.<br>3. A user code like "PAR-####" is assigned. | `src/features/alforqan/components/parent-create-wizard-form.tsx`, `src/features/alforqan/schemas/parent-create-wizard-schema.ts` | ALFQ-PAR-REG-007 |
| TC-09 | ALFQ-PAR-REG-009 | Verify "Require password change on first login" is available on admin fields | 1. Open create parent step 1.<br>2. Find the checkbox "Require password change on first login". | 1. The checkbox is visible on the Admin fields step.<br>2. Hint text explains the parent must set a new password before using the app. | `src/features/alforqan/components/parent-create-wizard-form.tsx` | ALFQ-PAR-REG-007 |
| TC-10 | ALFQ-PAR-REG-010 | Verify phone and ID stay disabled until nationality is selected, then validate while typing | 1. Reach create parent step 2 with valid step 1 data.<br>2. Leave nationality empty and confirm phone/ID are disabled.<br>3. Select nationality Egypt.<br>4. Type an invalid phone (for example `123`) and an invalid ID (for example `abc`). | 1. Phone and ID stay disabled until nationality is selected.<br>2. After nationality is selected, both fields become enabled.<br>3. Invalid phone and ID show inline errors while typing (without clicking Next or Save). | `src/features/alforqan/components/parent-create-wizard-form.tsx`, `src/lib/validation/trigger-nationality-contact.ts`, `src/lib/validation/nationality-contact-rules.ts` | ALFQ-PAR-REG-007 |
| TC-11 | ALFQ-PAR-REG-011 | Verify invalid username characters show an inline field error before or without a bare 500 | 1. Open create parent Admin fields.<br>2. Enter a valid email and a username containing `@` (for example `bad@name`).<br>3. Fill password `abcdef` and confirm.<br>4. Try Next / Save. | 1. An inline validation message appears on Username (allowed chars).<br>2. The user is not left with only a generic unexplained failure and no field message. | `src/features/alforqan/schemas/parent-create-wizard-schema.ts`, `src/lib/validation/tenant-identity-rules.ts`, `src/app/[locale]/(tenant)/[tenantSlug]/parents/create/parent-create-content.tsx` | ALFQ-PAR-REG-007 |

### Wave 2 — First-login gates

| ID | Key | Case | Steps | Expected | Related code | Requires |
| -- | --- | ---- | ----- | -------- | ------------ | -------- |
| TC-12 | ALFQ-PAR-REG-012 | Verify forced password change blocks the app until the parent changes password | 1. Admin creates a parent with "Require password change on first login" checked and skips optional profile.<br>2. Sign out.<br>3. Sign in as that parent.<br>4. Submit a valid new simple password on the change-password page. | 1. The parent is redirected to the change-password page (not the dashboard).<br>2. The change-password request completes successfully.<br>3. After a successful password change, the parent is sent to complete-profile (because profile is still incomplete). | `src/features/alforqan/components/tenant-change-password-form.tsx`, `src/components/auth/tenant-auth-guard.tsx` | ALFQ-PAR-REG-008, ALFQ-PAR-REG-009 |
| TC-13 | ALFQ-PAR-REG-013 | Verify incomplete profile blocks the app until required fields are saved | 1. As a parent with empty first name / last name / DOB / nationality / phone / ID, open any tenant app URL after login (password already changed if forced).<br>2. Complete first name, last name, date of birth, nationality, phone, and ID valid for that nationality.<br>3. Save. | 1. Before save, the parent cannot use the main app shell routes.<br>2. Nationality, phone, and ID are required.<br>3. After save, the parent can open the tenant home/dashboard and is not sent back to complete-profile asking for nationality again. | `src/features/alforqan/components/tenant-complete-profile-form.tsx`, `src/components/auth/tenant-auth-guard.tsx`, `src/lib/validation/nationality-contact-rules.ts` | ALFQ-PAR-REG-008 |
| TC-14 | ALFQ-PAR-REG-014 | Verify parent sees admin-controlled fields as read-only on complete profile | 1. Open the complete-profile page as a parent.<br>2. Look at email, user code, gender, and active status.<br>3. Optionally fill secondary phone.<br>4. If `canChangeUsername` is true, read the username hint and optionally change username once.<br>5. Save a complete profile. | 1. Email, user code, gender, and active status are shown as read-only.<br>2. First name, last name, date of birth, nationality, address, phone, national ID, profile picture, and optional secondary phone are editable.<br>3. Username shows a clear optional / once-only hint when allowed.<br>4. After save, the parent cannot change username again. | `src/features/alforqan/components/tenant-complete-profile-form.tsx` | ALFQ-PAR-REG-013 |
