# Students — Registration & first-login onboarding (Frontend)

## Metadata
- Project: `alforqan`
- Feature: `students`
- Layer: `frontend`
- Routes: `/{tenantSlug}/students/create`, `/{tenantSlug}/auth/change-password`, `/{tenantSlug}/auth/complete-profile`
- APIs: `POST /API/V01/Students/create-with-user`, `GET /API/V01/Authentication/OnboardingStatus`, `GET /API/V01/Authentication/MyProfile`, `PUT /API/V01/Authentication/MyProfile`, `PATCH /API/V01/Authentication/ChangePassword`
- Related code: `src/app/[locale]/(tenant)/[tenantSlug]/students/layout.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/students/create/page.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/students/students-list-content.tsx`, `src/features/alforqan/components/student-create-wizard-form.tsx`, `src/features/alforqan/schemas/student-create-wizard-schema.ts`, `src/features/sard/student-pending-sard-checkpoint-field.tsx`, `src/features/sard/student-sard-pending-checkpoint-utils.ts`, `src/components/auth/permission-guard.tsx`, `src/components/auth/tenant-auth-guard.tsx`, `src/features/alforqan/components/tenant-complete-profile-form.tsx`, `src/lib/auth/tenant-onboarding.ts`, `src/lib/validation/nationality-contact-rules.ts`
- Related docs: `docs/technical/tenant/student-registration-onboarding.md`, `docs/technical/tenant/frontend-backend-validation-parity.md`, `Backend/docs/technical-docs/password-policy-by-role.md`

## Scope
Admin two-step create-student wizard (required admin fields including gender and Quran progress + optional skippable profile) and student first-login gates (force password change and complete profile with nationality-gated phone/ID).

## Test pages
- `/{tenantSlug}/students/create`
- `/{tenantSlug}/auth/login`
- `/{tenantSlug}/auth/change-password`
- `/{tenantSlug}/auth/complete-profile`

## Test scenarios
- Admin creates a student with only admin fields and skips optional profile.
- Gender and Quran progress are required for the admin; parents are not linked at create.
- Student with incomplete profile cannot use the app until completion.
- Phone and ID stay disabled until nationality is chosen; both are required on first login.

## Delivery waves
| Wave | Intent | Keys |
| ---- | ------ | ---- |
| 1 | Admin create wizard | `ALFQ-STU-REG-001`, `ALFQ-STU-REG-002`, `ALFQ-STU-REG-003`, `ALFQ-STU-REG-004` |
| 2 | First-login gates | `ALFQ-STU-REG-005`, `ALFQ-STU-REG-006`, `ALFQ-STU-REG-007` |
| 3 | Uniqueness | `ALFQ-STU-REG-008` |
| 4 | Create permission UI | `ALFQ-STU-REG-009`, `ALFQ-STU-REG-010` |
| 5 | Sard checkpoint onboarding | `ALFQ-STU-REG-011`, `ALFQ-STU-REG-012`, `ALFQ-STU-REG-013` |
| 6 | Simple password policy | `ALFQ-STU-REG-014`, `ALFQ-STU-REG-015`, `ALFQ-STU-REG-016` |

## Test cases

### Wave 1 — Admin create wizard

| ID | Key | Case | Steps | Expected | Related code | Requires |
| -- | --- | ---- | ----- | -------- | ------------ | -------- |
| TC-01 | ALFQ-STU-REG-001 | Verify the create-student wizard shows "Admin fields" then "Optional profile" | 1. Sign in as a tenant admin with create-student permission.<br>2. Open `/{tenantSlug}/students/create`.<br>3. Read the step labels. | 1. Step 1 is labeled "Admin fields" / Arabic equivalent.<br>2. Step 2 is labeled "Optional profile" / Arabic equivalent.<br>3. There is no parents-linking step on create. | `src/features/alforqan/components/student-create-wizard-form.tsx` | - |
| TC-02 | ALFQ-STU-REG-002 | Verify admin can create a student with admin fields only and skip optional profile | 1. On step 1 enter email, username (Identity charset), password `abcdef` + confirm (simple policy: min 6, no complexity), gender Male or Female, leave Active on, set juz 1–30, reached surah, last ayah, and hifz direction (forward or reverse).<br>2. Optionally check "Require password change on first login".<br>3. Click Next.<br>4. On step 2 leave all fields empty.<br>5. Click "Skip and save". | 1. Student is created successfully.<br>2. No validation error forces first name, last name, date of birth, nationality, phone, or ID for the admin.<br>3. A user code like "STU-####" is assigned (visible later on profile / student record).<br>4. Creating a second student the same way succeeds with the next code (for example "STU-0002" after "STU-0001"). | `src/features/alforqan/components/student-create-wizard-form.tsx`, `src/features/alforqan/schemas/student-create-wizard-schema.ts`, `src/lib/validation/tenant-identity-rules.ts`, `Backend/Alforqan/Alforqan.Core/Features/Students/Commands/Handlers/CreateStudentWithUserCommandHandler.cs` | ALFQ-STU-REG-001 |
| TC-03 | ALFQ-STU-REG-003 | Verify gender and Quran progress are required on admin fields | 1. Open create student step 1.<br>2. Fill email, username, and password only.<br>3. Leave gender unset and juz/surah/ayah/hifz empty or "not set".<br>4. Try Next. | 1. Gender shows a required error (Male or Female).<br>2. Juz, reached surah, last ayah, and hifz direction each show a required error.<br>3. The wizard does not advance. | `src/features/alforqan/schemas/student-create-wizard-schema.ts` | ALFQ-STU-REG-001 |
| TC-04 | ALFQ-STU-REG-004 | Verify phone and ID stay disabled until nationality is selected, then validate while typing | 1. Reach create student step 2 with valid step 1 data.<br>2. Leave nationality empty and confirm phone/ID are disabled.<br>3. Select nationality Egypt.<br>4. Type an invalid phone (for example `123`) and an invalid ID (for example `abc`). | 1. Phone and ID stay disabled until nationality is selected.<br>2. After nationality is selected, both fields become enabled.<br>3. Invalid phone and ID show inline errors while typing (without clicking Next or Save). | `src/features/alforqan/components/student-create-wizard-form.tsx`, `src/lib/validation/trigger-nationality-contact.ts`, `src/lib/validation/nationality-contact-rules.ts` | ALFQ-STU-REG-001 |

### Wave 2 — First-login gates

| ID | Key | Case | Steps | Expected | Related code | Requires |
| -- | --- | ---- | ----- | -------- | ------------ | -------- |
| TC-05 | ALFQ-STU-REG-005 | Verify forced password change blocks the app until the student changes password | 1. Admin creates a student with "Require password change on first login" checked and skips optional profile (password `abcdef`).<br>2. Sign out.<br>3. Sign in as that student.<br>4. Submit a new simple password (for example `newpass`) on the change-password page. | 1. The student is redirected to the change-password page (not the dashboard).<br>2. The change-password request completes quickly (no long hang or timeout).<br>3. No password-changed notification email is sent.<br>4. After a successful password change, the student is sent to complete-profile when the profile is still incomplete. | `src/features/alforqan/components/tenant-change-password-form.tsx`, `src/lib/validation/tenant-identity-rules.ts`, `Backend/Alforqan/Alforqan.Core/Builders/Implementations/Identity/Base/Authentication/Commands/ChangePasswordCommandBuilderBase.cs` | ALFQ-STU-REG-002 |
| TC-06 | ALFQ-STU-REG-006 | Verify student must complete name, DOB, nationality, phone, and ID before using the app | 1. As a student with incomplete profile (password already changed if forced), open any tenant app URL.<br>2. Fill first name, last name, date of birth, nationality, phone, and ID valid for that nationality.<br>3. Save. | 1. Before save, the student cannot use the main app shell routes.<br>2. Nationality, phone, and ID are required.<br>3. After save, the student can open the tenant home/dashboard and is not sent back to complete-profile asking for nationality again. | `src/features/alforqan/components/tenant-complete-profile-form.tsx`, `src/components/auth/tenant-auth-guard.tsx`, `src/lib/validation/nationality-contact-rules.ts` | ALFQ-STU-REG-002 |
| TC-07 | ALFQ-STU-REG-007 | Verify student complete-profile shows admin fields read-only and optional one-time username | 1. Open complete-profile as a student with `canChangeUsername` true.<br>2. Check read-only and editable areas. | 1. Email, user code, gender, stage, juz, surah, ayah, hifz, and active status are read-only.<br>2. Username shows as optional with a clear once-only hint (or remains editable only in this window).<br>3. After a successful save, username is no longer changeable by the student. | `src/features/alforqan/components/tenant-complete-profile-form.tsx` | ALFQ-STU-REG-006 |
| TC-08 | ALFQ-STU-REG-008 | Verify duplicate email, username, or national ID is rejected for students | 1. Note an existing student's email, username, or ID number.<br>2. Try to create another student (or complete another profile) reusing that value.<br>3. Save. | 1. Create/save fails with a clear already-exists message for the duplicated field.<br>2. No second student keeps the duplicate email, username, or ID. | `src/app/[locale]/(tenant)/[tenantSlug]/students/create/student-create-content.tsx`, `src/features/alforqan/components/tenant-complete-profile-form.tsx` | ALFQ-STU-REG-002 |

### Wave 4 — Create permission UI

| ID | Key | Case | Steps | Expected | Related code | Requires |
| -- | --- | ---- | ----- | -------- | ------------ | -------- |
| TC-09 | ALFQ-STU-REG-009 | Verify a teacher without CreateStudentClaim does not see Add student | 1. Sign in as a tenant teacher (ViewStudents only).<br>2. Open `/{tenantSlug}/students`. | 1. The students list loads.<br>2. The "Add student" / "Create new" button is not shown. | `src/app/[locale]/(tenant)/[tenantSlug]/students/students-list-content.tsx` | - |
| TC-10 | ALFQ-STU-REG-010 | Verify direct URL to student create shows no-permission for a teacher | 1. While signed in as a teacher without CreateStudentClaim, open `/{tenantSlug}/students/create` directly. | 1. The page shows the standard no-permission card (not the create wizard).<br>2. No create-with-user API call is required to validate the UI gate. | `src/app/[locale]/(tenant)/[tenantSlug]/students/create/page.tsx`, `src/components/auth/permission-guard.tsx` | - |

### Wave 5 — Sard checkpoint onboarding

| ID | Key | Case | Steps | Expected | Related code | Requires |
| -- | --- | ---- | ----- | -------- | ------------ | -------- |
| TC-11 | ALFQ-STU-REG-011 | Verify pending Sard checkpoint defaults to the first checkpoint after reached ayah on create | 1. Sign in as tenant admin with create-student permission.<br>2. Open create student step 1.<br>3. Set hifz direction "From the beginning", juz 2, reached surah Al-Baqara (2), last ayah 100.<br>4. Read the "Pending Sard checkpoint" field. | 1. The field appears after hifz direction is set.<br>2. The default selection is the first checkpoint **after** ayah 100 (for example "3 أجزاء" / Al-Imran 3:62 in the default forward plan).<br>3. Earlier checkpoints (for example 2:21, 2:41) are not offered. | `src/features/sard/student-pending-sard-checkpoint-field.tsx`, `src/features/alforqan/components/student-create-wizard-form.tsx` | ALFQ-STU-REG-001 |
| TC-12 | ALFQ-STU-REG-012 | Verify creating a student with default pending checkpoint does not block memorization at reached position | 1. Complete create with step 1 data from TC-11 and skip optional profile.<br>2. Enroll the student in a halaqa and open a memorization evaluate page.<br>3. Set a range within the student's reached progress (for example up to Al-Baqara 2:100).<br>4. Click "Save draft".<br>5. Click "Submit evaluation". | 1. Student is created successfully.<br>2. No Sard gate banner blocks memorization within the reached range.<br>3. Draft save succeeds (no Sard gate dialog).<br>4. Submit opens the session outcome modal (no Sard gate dialog). | `Backend/Alforqan/Alforqan.Core/Features/Sard/Helpers/SardCheckpointPassSyncHelper.cs`, `src/features/alforqan/components/student-create-wizard-form.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/halaqat/[id]/sessions/[sessionId]/evaluate/[studentId]/halaqa-evaluation-content.tsx` | ALFQ-STU-REG-011 |
| TC-13 | ALFQ-STU-REG-013 | Verify admin can choose a later pending checkpoint on create | 1. On create step 1 with the same reached position as TC-11, open "Pending Sard checkpoint".<br>2. Select a checkpoint later than the default (for example "4 أجزاء").<br>3. Finish create. | 1. Only checkpoints after the reached ayah are listed.<br>2. After save, the student profile Sard status shows the selected checkpoint as pending. | `src/features/sard/student-pending-sard-checkpoint-field.tsx`, `Backend/Alforqan/Alforqan.Core/Features/Sard/Helpers/SardCheckpointPassSyncHelper.cs` | ALFQ-STU-REG-011 |

### Wave 6 — Simple password policy

| ID | Key | Case | Steps | Expected | Related code | Requires |
| -- | --- | ---- | ----- | -------- | ------------ | -------- |
| TC-14 | ALFQ-STU-REG-014 | Verify simple password `abcdef` is accepted on student create | 1. Open create student step 1.<br>2. Enter valid email/username and password `abcdef` with matching confirm.<br>3. Complete required gender and Quran fields.<br>4. Skip optional profile and save. | 1. No complexity error on Password.<br>2. Student is created successfully. | `src/features/alforqan/schemas/student-create-wizard-schema.ts`, `src/lib/validation/tenant-identity-rules.ts` | ALFQ-STU-REG-001 |
| TC-15 | ALFQ-STU-REG-015 | Verify password shorter than 6 characters is rejected on student create | 1. On create step 1 enter password `abc12` (5 chars) with matching confirm.<br>2. Try Next. | 1. Password shows a minimum-length error.<br>2. The wizard does not advance. | `src/features/alforqan/schemas/student-create-wizard-schema.ts` | ALFQ-STU-REG-001 |
| TC-16 | ALFQ-STU-REG-016 | Verify password longer than 256 characters is rejected on student create | 1. On create step 1 enter a password of 257 lowercase letters with matching confirm.<br>2. Try Next. | 1. Password shows a maximum-length error (256).<br>2. The wizard does not advance. | `src/features/alforqan/schemas/student-create-wizard-schema.ts` | ALFQ-STU-REG-001 |
