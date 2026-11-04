# Student Assignments (Homework)

## Overview

Homework (`StudentAssignments`) is the **next-session Quran range** a teacher assigns after evaluation (or manually). Students and parents view Open/Done homework; Cancelled rows are hidden from them.

- **Memorization (حفظ):** exactly **one** range (`FromSurahId`/`FromAyah` → `ToSurahId`/`ToAyah`), stored in `StudentAssignmentRanges` (one row) plus an envelope on the parent row.
- **Revision:** **one or more** non-overlapping master ranges in `StudentAssignmentRanges`; envelope = mushaf min→max of all ranges.
- **Status:** `Open`, `Done`, `Cancelled`. At most one **Open** per `(StudentId, HalaqaId, AssignmentType)` (filtered unique index).
- **Sessions:** `SessionId` = target next session; `AssignedFromSessionId` = session where teacher set homework after evaluation.

Revision **session evaluations** also use `SessionEvaluationMasterRanges` (multiple masters) with `SessionRevisionSegment.MasterRangeId` linking segments to a master.

## Base route

- **Assignments**: `/API/V01/StudentAssignments`

## Entity notes (ERD deviation)

Legacy ERD used a single `SurahId` + `FromAyah`/`ToAyah`. Implemented as **from/to surah** + child `StudentAssignmentRanges` table.

## Controllers

`Alforqan.Api/Controllers/StudentAssignments/StudentAssignmentsController.cs`

## List filters (POST `List`)

| Field | Purpose |
|-------|---------|
| `StudentId`, `HalaqaId`, `AssignmentType` | Optional filters |
| `Status` | `Open` / `Done` / `Cancelled` |
| `FromUtc`, `ToUtc` | Filter on `DueDate` if set, else `CreatedOn` |

**Scope (server-side, not client-trusted):**

| Role | Sees |
|------|------|
| Admin | All |
| Teacher | Own halaqat only (`Halaqa.TeacherId`) |
| Student | Own rows; **Cancelled hidden** |
| Parent | Linked children; **Cancelled hidden** |
| Other | Empty list |

## Create / update rules

- Validate enrollment (`HalaqaStudents` Active).
- Hifz = 1 range; revision ≥ 1; no overlap; valid surah/ayah.
- **Memorization window (hifz only):** `From` cannot start after `GetStartCap` (the ayah after last reached via `NextAyahInHifz`, or mushaf start by `HifzDirection` when none). From-end: ayah +1 in the surah; after last ayah → ayah 1 of the previous surah (104:9 → 103:1). `To` cannot pass the pending sard checkpoint — same gate as hifz evaluation (`SARD_GATE_MEMORIZATION_RANGE_BLOCKED`). Revision homework is not windowed. Confirm-replace on eval upsert validates before cancelling Open homework.
- New **Open** cancels prior Open for same `(student, halaqa, type)`.
- Default `SessionId` + `DueDate` from next non-cancelled `HalaqaSession` with `ScheduledAt > UtcNow`.
- **Update** only allowed when `Status = Open`.
- **Delete** uses the same mutate scope as create/update (admin = any; teacher = own halaqat). Teacher default claims include `DeleteAssignmentClaim`.
- **Notify** student + linked parents on create and on Open edit (`TenantNotificationType.HomeworkAssigned`).

## Evaluation integration

See `docs/technical-docs/homework-evaluation-lock.md` — copy Open homework into eval, lock range, confirm override, auto-**Done** on outcome.

