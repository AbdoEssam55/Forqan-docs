# User activation — live halaqa & sessions — Backend Test Cases

## Metadata

- Project: `alforqan`
- Feature: `users`
- Layer: `backend`
- Routes: `-`
- APIs: Halaqa students list, complete session, halaqa list counts
- Related code: `Alforqan/Alforqan.Core/Features/Halaqat/Queries/Handlers/GetStudentsInHalaqaQueryHandler.cs`
- Related docs: `docs/technical/tenant/user-activation.md`
- Automation: `-`

## Scope

- Inactive students excluded from live halaqa roster and session completion attendance requirements.
- Historical session attendance unchanged.
- Inactive teacher cannot be assigned on halaqa create/update.

## Test cases

### Wave 2 — Halaqa & sessions

| ID | Key | Case | Steps | Expected | Related code |
|----|-----|------|-------|----------|--------------|
| TC-01 | ALFQ-USER-ACT-BE-010 | Verify halaqa session roster omits inactive student | 1. Enroll active student in halaqa.<br>2. Deactivate student.<br>3. Open students-in-halaqa for session UI. | 1. Inactive student not in roster. | `Alforqan/Alforqan.Core/Features/Halaqat/Queries/Handlers/GetStudentsInHalaqaQueryHandler.cs` |
| TC-02 | ALFQ-USER-ACT-BE-011 | Verify complete session does not require attendance for inactive student | 1. Deactivate enrolled student.<br>2. Complete session without their attendance row. | 1. Completion succeeds.<br>2. No validation forcing missing inactive student. | `Alforqan/Alforqan.Core/Features/HalaqaSessions/Commands/HalaqaSessionMutations.cs` |
| TC-03 | ALFQ-USER-ACT-BE-012 | Verify halaqa list studentsCount drops when student deactivated | 1. Note count for halaqa.<br>2. Deactivate one enrolled student.<br>3. Reload halaqa list. | 1. `studentsCount` decreases by one. | `Alforqan/Alforqan.Core/Features/Halaqat/Queries/Handlers/HalaqaListQueryHandler.cs` |
| TC-04 | ALFQ-USER-ACT-BE-013 | Verify past session attendance still shows deactivated student | 1. Complete session with student present.<br>2. Deactivate student.<br>3. Open that session attendance/history. | 1. Student name/row still visible on completed session. | `-` |
| TC-05 | ALFQ-USER-ACT-BE-014 | Verify reactivate returns student to live roster | 1. Activate student.<br>2. Load students-in-halaqa. | 1. Student appears again (active enrollment). | `Alforqan/Alforqan.Core/Features/Halaqat/Queries/Handlers/GetStudentsInHalaqaQueryHandler.cs` |
| TC-06 | ALFQ-USER-ACT-BE-015 | Verify inactive teacher cannot be assigned on halaqa save | 1. Deactivate teacher.<br>2. Create or edit halaqa with that teacher. | 1. Save fails with localized inactive-teacher message. | `Alforqan/Alforqan.Core/Features/Halaqat/Commands/Handlers/HalaqaEditCommandHandler.cs` |
