# Teachers — Registration & first-login onboarding (Frontend)

## Metadata
- Project: `alforqan`
- Feature: `teachers`
- Layer: `frontend`
- Routes: `/{tenantSlug}/teachers/create`, `/{tenantSlug}/auth/change-password`, `/{tenantSlug}/auth/complete-profile`
- APIs: `POST /API/V01/Teachers/create-with-user`, `GET /API/V01/Authentication/OnboardingStatus`, `GET /API/V01/Authentication/MyProfile`, `PUT /API/V01/Authentication/MyProfile`, `PATCH /API/V01/Authentication/ChangePassword`
- Related code: `src/features/alforqan/components/teacher-create-wizard-form.tsx`, `src/features/alforqan/schemas/teacher-create-wizard-schema.ts`, `src/components/auth/tenant-auth-guard.tsx`, `src/features/alforqan/components/tenant-complete-profile-form.tsx`, `src/features/alforqan/components/tenant-change-password-form.tsx`, `src/lib/auth/tenant-onboarding.ts`, `src/lib/validation/nationality-contact-rules.ts`
- Related docs: `docs/technical/tenant/teacher-registration-onboarding.md`, `docs/technical/tenant/frontend-backend-validation-parity.md`, `Backend/docs/technical-docs/password-policy-by-role.md`

## Scope
Admin two-step create-teacher wizard (required admin fields including gender + optional profile with nationality-gated phone/ID) and teacher first-login gates (force password change, complete profile, optional one-time username).

## Test pages
- `/{tenantSlug}/teachers/create`
- `/{tenantSlug}/auth/login`
- `/{tenantSlug}/auth/change-password`
- `/{tenantSlug}/auth/complete-profile`

## Test scenarios
- Admin creates a teacher with only required fields and skips optional profile.
- Admin can require password change on first login.
- Teacher with incomplete profile cannot use the app until completion.
- Teacher sees salary/email as read-only and can edit name, DOB, nationality, contact.

## Delivery waves
| Wave | Intent | Keys |
| ---- | ------ | ---- |
| 1 | Admin create wizard | `ALFQ-TCH-REG-001`, `ALFQ-TCH-REG-002`, `ALFQ-TCH-REG-003`, `ALFQ-TCH-REG-009` |
| 2 | First-login gates | `ALFQ-TCH-REG-004`, `ALFQ-TCH-REG-005`, `ALFQ-TCH-REG-006`, `ALFQ-TCH-REG-010` |
| 3 | Create validation parity | `ALFQ-TCH-REG-007`, `ALFQ-TCH-REG-008`, `ALFQ-TCH-REG-011`, `ALFQ-TCH-REG-012` |

## Test cases

### Wave 1 — Admin create wizard

| ID | Key | Case | Steps | Expected | Related code | Requires |
| -- | --- | ---- | ----- | -------- | ------------ | -------- |
| TC-01 | ALFQ-TCH-REG-001 | Verify the create-teacher wizard shows "Admin fields" then "Optional profile" | 1. Sign in as a tenant admin with create-teacher permission.<br>2. Open `/{tenantSlug}/teachers/create`.<br>3. Read the step labels. | 1. Step 1 is labeled "Admin fields" / Arabic equivalent.<br>2. Step 2 is labeled "Optional profile" / Arabic equivalent.<br>3. There is no separate "Full name" / "Teacher record" step requiring display name. | `src/features/alforqan/components/teacher-create-wizard-form.tsx` | - |
| TC-02 | ALFQ-TCH-REG-002 | Verify admin can create a teacher with only admin fields and skip optional profile | 1. On step 1 enter email, username (letters/digits/`. - _`/space only), and a strong password (upper + lower + digit + symbol, min 6), confirm password, gender Male or Female, base salary, currency, and leave Active on.<br>2. Leave "Require password change on first login" unchecked.<br>3. Click Next.<br>4. On step 2 leave all fields empty.<br>5. Click "Skip and save" (or Save). | 1. Teacher is created successfully.<br>2. No validation error forces first name, last name, date of birth, nationality, phone, or ID for the admin.<br>3. A user code like "TCH-####" is assigned. | `src/features/alforqan/components/teacher-create-wizard-form.tsx`, `src/features/alforqan/schemas/teacher-create-wizard-schema.ts` | ALFQ-TCH-REG-001 |
| TC-03 | ALFQ-TCH-REG-003 | Verify "Require password change on first login" is available on admin fields | 1. Open create teacher step 1.<br>2. Find the checkbox "Require password change on first login". | 1. The checkbox is visible on the Admin fields step.<br>2. Hint text explains the teacher must set a new password before using the app. | `src/features/alforqan/components/teacher-create-wizard-form.tsx` | ALFQ-TCH-REG-001 |
| TC-09 | ALFQ-TCH-REG-009 | Verify phone and ID stay disabled until nationality is selected, then validate while typing | 1. Reach create teacher step 2 with valid step 1 data (including gender).<br>2. Leave nationality empty and confirm phone/ID are disabled.<br>3. Select nationality Egypt.<br>4. Type an invalid phone (for example `123`) and an invalid ID (for example `abc`). | 1. Phone and ID stay disabled until nationality is selected.<br>2. After nationality is selected, both fields become enabled.<br>3. Invalid phone and ID show inline errors while typing (without clicking Next or Save). | `src/features/alforqan/components/teacher-create-wizard-form.tsx`, `src/lib/validation/trigger-nationality-contact.ts`, `src/lib/validation/nationality-contact-rules.ts` | ALFQ-TCH-REG-001 |

### Wave 2 — First-login gates

| ID | Key | Case | Steps | Expected | Related code | Requires |
| -- | --- | ---- | ----- | -------- | ------------ | -------- |
| TC-04 | ALFQ-TCH-REG-004 | Verify forced password change blocks the app until the teacher changes password | 1. Admin creates a teacher with "Require password change on first login" checked and skips optional profile.<br>2. Sign out.<br>3. Sign in as that teacher.<br>4. Submit a valid new password on the change-password page. | 1. The teacher is redirected to the change-password page (not the dashboard).<br>2. The change-password request completes quickly (no long hang or timeout).<br>3. No password-changed notification email is sent.<br>4. After a successful password change, the teacher is sent to complete-profile (because profile is still incomplete). | `src/features/alforqan/components/tenant-change-password-form.tsx`, `Alforqan.Core/Builders/Implementations/Identity/Base/Authentication/Commands/ChangePasswordCommandBuilderBase.cs` | ALFQ-TCH-REG-002, ALFQ-TCH-REG-003 |
| TC-05 | ALFQ-TCH-REG-005 | Verify incomplete profile blocks the app until required fields are saved | 1. As a teacher with empty first name / last name / DOB / nationality / phone / ID, open any tenant app URL after login (password already changed if forced).<br>2. Complete first name, last name, date of birth, nationality, phone, and ID valid for that nationality.<br>3. Save. | 1. Before save, the teacher cannot use the main app shell routes.<br>2. Nationality, phone, and ID are required.<br>3. After save, the teacher can open the tenant home/dashboard and is not sent back to complete-profile asking for nationality again. | `src/features/alforqan/components/tenant-complete-profile-form.tsx`, `src/components/auth/tenant-auth-guard.tsx`, `src/lib/validation/nationality-contact-rules.ts` | ALFQ-TCH-REG-002 |
| TC-06 | ALFQ-TCH-REG-006 | Verify teacher sees admin-controlled fields as read-only on complete profile | 1. Open the complete-profile page as a teacher.<br>2. Look at email, user code, gender, salary, currency, and active status. | 1. Email, user code, gender, salary, currency, and active status are shown as read-only.<br>2. First name, last name, date of birth, nationality, address, phone, ID, and profile picture are editable. | `src/features/alforqan/components/tenant-complete-profile-form.tsx` | ALFQ-TCH-REG-005 |
| TC-10 | ALFQ-TCH-REG-010 | Verify optional one-time username change on first login then locks | 1. Open complete-profile as a teacher with `canChangeUsername` true.<br>2. Read the username hint.<br>3. Optionally change username or leave it.<br>4. Save a complete profile.<br>5. Re-open profile / complete-profile is no longer required. | 1. Username shows a clear optional / once-only hint.<br>2. After save, the teacher cannot change username again (admin-only later). | `src/features/alforqan/components/tenant-complete-profile-form.tsx` | ALFQ-TCH-REG-005 |

### Wave 3 — Create validation parity

| ID | Key | Case | Steps | Expected | Related code | Requires |
| -- | --- | ---- | ----- | -------- | ------------ | -------- |
| TC-07 | ALFQ-TCH-REG-007 | Verify invalid username characters show an inline field error before or without a bare 500 | 1. Open create teacher Admin fields.<br>2. Enter a valid email and a username containing `@` (for example `bad@name`).<br>3. Fill a strong password (digit, upper, lower, symbol) and salary.<br>4. Try Next / Save. | 1. An inline validation message appears on Username (allowed chars).<br>2. The user is not left with only a generic unexplained failure and no field message. | `src/features/alforqan/schemas/teacher-create-wizard-schema.ts`, `src/lib/validation/tenant-identity-rules.ts`, `src/app/[locale]/(tenant)/[tenantSlug]/teachers/create/teacher-create-content.tsx` | ALFQ-TCH-REG-001 |
| TC-08 | ALFQ-TCH-REG-008 | Verify a weak password is rejected with an inline password rule message | 1. Open create teacher Admin fields.<br>2. Enter valid email/username.<br>3. Enter password `abcdef` (no digit/upper/symbol).<br>4. Try Next. | 1. Password shows a clear complexity message (digit / upper / non-alphanumeric as applicable).<br>2. The wizard does not proceed until the password meets Identity rules. | `src/features/alforqan/schemas/teacher-create-wizard-schema.ts`, `src/lib/validation/tenant-identity-rules.ts` | ALFQ-TCH-REG-001 |
| TC-11 | ALFQ-TCH-REG-011 | Verify duplicate email or username is rejected when creating a teacher | 1. Note an existing teacher email and username.<br>2. Open create teacher and enter that same email (or username) with a new other credential, gender, salary, and strong password.<br>3. Save. | 1. Create fails with a clear email or username already-exists message (inline or toast).<br>2. No second teacher is created with that email/username. | `src/app/[locale]/(tenant)/[tenantSlug]/teachers/create/teacher-create-content.tsx` | ALFQ-TCH-REG-002 |
| TC-12 | ALFQ-TCH-REG-012 | Verify duplicate national ID is rejected on create or complete-profile | 1. Create or complete a teacher profile with a valid unique ID for a nationality.<br>2. Create another teacher (or complete another user's profile) using the same ID number.<br>3. Save. | 1. The second save fails with an ID-already-used message on the ID field or toast.<br>2. The second user does not keep that duplicate ID. | `src/features/alforqan/components/tenant-complete-profile-form.tsx`, `src/features/alforqan/components/teacher-create-wizard-form.tsx` | ALFQ-TCH-REG-002 |
