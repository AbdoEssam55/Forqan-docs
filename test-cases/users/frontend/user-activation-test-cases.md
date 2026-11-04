# User activation — Frontend Test Cases

## Metadata

- Project: `alforqan`
- Feature: `users`
- Layer: `frontend`
- Routes: `/{locale}/{tenantSlug}/users`, students/teachers/parents lists and details
- APIs: `PATCH AppUser/Activate`, `PATCH AppUser/UnActivate`, dashboard stats
- Related code: `src/app/[locale]/(tenant)/[tenantSlug]/users/users-list-content.tsx`
- Related docs: `docs/technical/tenant/user-activation.md`
- Automation: `-`

## Scope

- Activate/deactivate UI, default Active filters, hub inactive hints, session roster via API.

## Test cases

### Wave 5 — UI

| ID | Key | Case | Steps | Expected | Related code |
|----|-----|------|-------|----------|--------------|
| TC-01 | ALFQ-USER-ACT-FE-001 | Verify deactivate hides student from default active list | 1. Open Students list (default Active).<br>2. Deactivate a student from detail.<br>3. Return to list. | 1. Student not in default list.<br>2. Appears under Inactive filter. | `src/app/[locale]/(tenant)/[tenantSlug]/students/students-list-content.tsx` |
| TC-02 | ALFQ-USER-ACT-FE-002 | Verify activate restores student to active list | 1. Open inactive student detail.<br>2. Click "Activate".<br>3. Open Active list. | 1. Success toast.<br>2. Student visible on Active list. | `src/app/[locale]/(tenant)/[tenantSlug]/students/[id]/student-detail-content.tsx` |
| TC-03 | ALFQ-USER-ACT-FE-003 | Verify users hub shows inactive count hint | 1. Deactivate a teacher.<br>2. Open Users hub. | 1. Teachers card shows active count.<br>2. Inactive hint shows "{count} inactive". | `src/app/[locale]/(tenant)/[tenantSlug]/users/users-list-content.tsx` |
| TC-04 | ALFQ-USER-ACT-FE-004 | Verify session roster omits inactive student | 1. Deactivate enrolled student.<br>2. Start or open halaqa session attendance. | 1. Student not in roster. | `-` |
| TC-05 | ALFQ-USER-ACT-FE-005 | Verify historical report still shows deactivated student name | 1. Open past memorization/report for student.<br>2. Deactivate student.<br>3. Reopen same report. | 1. Name and data still display. | `-` |
| TC-06 | ALFQ-USER-ACT-FE-006 | Verify users list status filter Active/Inactive | 1. Set Users list filter to Inactive.<br>2. Activate user from row actions. | 1. User leaves inactive list after refresh.<br>2. Appears on Active filter. | `src/app/[locale]/(tenant)/[tenantSlug]/users/users-list-content.tsx` |
