# Halaqa session permissions — Frontend Test Cases

## Metadata
- Project: `alforqan`
- Feature: `halaqat`
- Layer: `frontend`
- Routes: ``/{locale}/{tenantSlug}/halaqat/{halaqaId}/sessions/{sessionId}``, ``/{locale}/{tenantSlug}/halaqat/{halaqaId}/sessions/{sessionId}/evaluate/{studentId}``
- Related code: `src/features/alforqan/halaqat/use-can-manage-halaqa-session.ts`, `src/features/alforqan/halaqat/use-can-evaluate-halaqa-session.ts`, `src/components/auth/halaqa-teacher-guard.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/halaqat/[id]/sessions/[sessionId]/halaqa-session-detail-content.tsx`
- Related docs: `docs/technical/tenant/halaqat-guide.md`, `Backend/docs/technical-docs/halaqa-session-permissions.md`, `Backend/docs/technical-docs/tenant-user-claims-save.md`

## Scope

- Claim-based access to session manage and evaluate actions (not assigned-teacher-only).
- Admin can uncheck session claims on Roles & permissions; change sticks after re-login.

## Test cases

### Session access by claim

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-01 | ALFQ-HAL-PERM-001 | Verify tenant admin with EvaluateHalaqaSessionsClaim can open a draft evaluation for a halaqa they do not teach | Main | Tenant admin is **not** the assigned teacher; admin has `EvaluateHalaqaSessionsClaim` on user claims; session exists with at least one enrolled student and a draft or submitted evaluation row | 1. Sign in as that admin (re-login after claim setup if needed).<br>2. Open the session detail for the halaqa.<br>3. Choose "Evaluate" for a student with an existing evaluation row. | 1. The evaluate page loads (no teacher-only guard message).<br>2. Student range and evaluation controls are visible. | ALFQ-HAL-SES-003 | `src/features/alforqan/halaqat/use-can-evaluate-halaqa-session.ts`, `src/components/auth/halaqa-teacher-guard.tsx` | Admin not assigned to halaqa |
| TC-02 | ALFQ-HAL-PERM-002 | Verify tenant admin without EvaluateHalaqaSessionsClaim cannot open evaluate UI | Main | Tenant admin is not the assigned teacher; `EvaluateHalaqaSessionsClaim` is unchecked and saved on the user; user re-logged in | 1. Sign in as that admin.<br>2. Open the same session detail URL.<br>3. Try to open the evaluate URL directly for a student. | 1. No "Evaluate" action appears for students on session detail (or it is disabled).<br>2. Direct evaluate URL shows the standard no-access / teacher guard state (not the evaluation form). | ALFQ-HAL-SES-003 | `src/features/alforqan/halaqat/use-can-evaluate-halaqa-session.ts`, `src/components/auth/halaqa-teacher-guard.tsx` | Admin with evaluate claim removed |
| TC-03 | ALFQ-HAL-PERM-003 | Verify assigned teacher without session claims can still evaluate their halaqa | Main | User is the halaqa's assigned teacher; teacher role has no `EvaluateHalaqaSessionsClaim` on user claims | 1. Sign in as the assigned teacher.<br>2. Open session detail and choose "Evaluate" for a student. | 1. The evaluate page loads normally.<br>2. Save draft and submit remain available when session rules allow. | ALFQ-HAL-SES-003 | `src/features/alforqan/halaqat/use-can-evaluate-halaqa-session.ts` | Assigned teacher account |
| TC-04 | ALFQ-HAL-PERM-004 | Verify tenant admin with ManageHalaqaSessionsClaim can schedule a session on a halaqa they do not teach | Main | Tenant admin is not the assigned teacher; admin has `ManageHalaqaSessionsClaim`; user re-logged in after claim setup | 1. Open the halaqa sessions list.<br>2. Choose "Add session" and create a future session with a valid title and time. | 1. "Add session" is visible.<br>2. Create succeeds and the new session appears in the list. | ALFQ-HAL-SES-001 | `src/features/alforqan/halaqat/use-can-manage-halaqa-session.ts`, `src/app/[locale]/(tenant)/[tenantSlug]/halaqat/[id]/sessions/halaqa-sessions-list-content.tsx` | Admin not assigned to halaqa |
| TC-05 | ALFQ-HAL-PERM-005 | Verify parent without session claims cannot open teacher session manage UI | Edge | Parent account signed in; not assigned teacher; no manage/evaluate session claims | 1. Attempt to open a teacher session detail URL for a child's halaqa (if URL known). | 1. Lifecycle and attendance manage actions are not shown.<br>2. User does not get teacher session manage controls. | - | `src/features/alforqan/halaqat/use-can-manage-halaqa-session.ts`, `src/app/[locale]/(tenant)/[tenantSlug]/halaqat/[id]/sessions/[sessionId]/halaqa-session-detail-content.tsx` | Parent linked to student |

### Claim edit fidelity (tenant)

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-06 | ALFQ-HAL-PERM-006 | Verify unchecking EvaluateHalaqaSessionsClaim on an admin persists after save and re-login | Main | Editor has permission to modify admin claims; target user is tenant Admin on a halaqa they do not teach | 1. Open the admin user's Roles & permissions tab (including when the target is the signed-in admin).<br>2. Under Halaqat, uncheck "Evaluate halaqa sessions" (or Arabic equivalent).<br>3. Save roles & permissions.<br>4. If still signed in, reload the permissions tab. Otherwise sign in again.<br>5. Try to open an evaluate page for a non-assigned halaqa session. | 1. Save does not restore the claim by reseeding Admin role defaults.<br>2. The claim checkbox stays unchecked after reload of the editor tab.<br>3. After re-login, evaluate UI is blocked per TC-02.<br>4. Re-checking the claim and saving restores evaluate access after JWT refresh or re-login. | - | `src/components/users/user-roles-and-claims-tab.tsx`, `src/providers/permissions-provider.tsx` | Tenant admin test user |
