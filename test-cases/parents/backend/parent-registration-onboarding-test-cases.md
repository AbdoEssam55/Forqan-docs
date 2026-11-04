# Parents — Registration & first-login onboarding (Backend)

## Metadata
- Project: `alforqan`
- Feature: `parents`
- Layer: `backend`
- APIs: `POST /API/V01/Parents/create-with-user`, `GET /API/V01/Authentication/OnboardingStatus`, `PUT /API/V01/Authentication/MyProfile`
- Related code: `Alforqan.Core/Features/Parents/Commands/Handlers/CreateParentWithUserCommandHandler.cs`, `Alforqan.Core/Features/Parents/Commands/Validators/CreateParentWithUserCommandValidator.cs`, `Alforqan.Core/Features/Identity/Tenant/Authentication/Commands/Handlers/UpdateMyProfileCommandHandler.cs`, `Alforqan.Core/Features/Users/Tenant/Helpers/UserCodeGenerator.cs`
- Related docs: `Frontend/alforqan/docs/technical/tenant/parent-registration-onboarding.md`
- Automation: `Tests/Alforqan.Core.Tests/Features/Parents/CreateParentWithUserCommandValidatorTests.cs`, `Tests/Alforqan.Core.Tests/Features/Parents/UpdateMyProfileCommandValidatorTests.cs`

## Scope
Admin minimal parent create (login only + optional profile) and parent row sync on first-login profile completion.

## Delivery waves
| Wave | Intent | Keys |
| ---- | ------ | ---- |
| 1 | Create-with-user validation | `ALFQ-PAR-REG-BE-001`, `ALFQ-PAR-REG-BE-002`, `ALFQ-PAR-REG-BE-003` |
| 2 | Complete-profile parent sync | `ALFQ-PAR-REG-BE-004` |

## Test cases

### Wave 1 — Create-with-user

| ID | Key | Case | Steps | Expected | Related code | Requires |
| -- | --- | ---- | ----- | -------- | ------------ | -------- |
| TC-01 | ALFQ-PAR-REG-BE-001 | Verify create-with-user accepts email, username, and password only | 1. POST `create-with-user` with Email, Username, Password, ConfirmPassword only.<br>2. Omit FirstName, LastName, DateOfBirth, FullName. | 1. Request succeeds (201/200).<br>2. Parent row exists with derived FullName (username or email fallback).<br>3. User has role Parent and user code `PAR-####`. | `Alforqan.Core/Features/Parents/Commands/Handlers/CreateParentWithUserCommandHandler.cs` | - |
| TC-02 | ALFQ-PAR-REG-BE-002 | Verify simple password `abcdef` passes parent create validation | 1. POST create-with-user with Password and ConfirmPassword `abcdef`. | 1. No password complexity errors.<br>2. User is created. | `Alforqan.Core/Features/Parents/Commands/Validators/CreateParentWithUserCommandValidator.cs`, `Alforqan.Shared/HelperClasses/PasswordPolicyHelper.cs` | - |
| TC-03 | ALFQ-PAR-REG-BE-003 | Verify optional phone/ID require nationality and valid format when provided | 1. POST with PhoneNumber but no Nationality.<br>2. POST with Nationality EG and invalid IdNumber. | 1. First request fails with phone/nationality validation.<br>2. Second request fails with ID format validation. | `Alforqan.Core/Features/Parents/Commands/Validators/CreateParentWithUserCommandValidator.cs`, `Alforqan.Core/Features/Users/Tenant/Helpers/NationalityContactValidator.cs` | - |

### Wave 2 — Complete-profile sync

| ID | Key | Case | Steps | Expected | Related code | Requires |
| -- | --- | ---- | ----- | -------- | ------------ | -------- |
| TC-04 | ALFQ-PAR-REG-BE-004 | Verify UpdateMyProfile syncs Parent FullName, phone, nationality, and optional secondary phone | 1. Create parent with minimal fields.<br>2. Sign in as parent.<br>3. PUT MyProfile with first/last name, DOB, nationality, phone, ID, optional SecondaryPhone.<br>4. GET parent by id as admin. | 1. MyProfile returns `mustCompleteProfile` false.<br>2. Parent row FullName matches first + last.<br>3. Parent PhoneNumber and Nationality match AppUser.<br>4. SecondaryPhone saved when provided. | `Alforqan.Core/Features/Identity/Tenant/Authentication/Commands/Handlers/UpdateMyProfileCommandHandler.cs` | ALFQ-PAR-REG-BE-001 |
