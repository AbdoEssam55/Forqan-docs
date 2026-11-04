# Students

## Overview

Students are the core learners in a tenant (institute). The Students API provides CRUD and paginated list with search and ordering. All endpoints require tenant authentication and appropriate tenant claims (e.g. ViewStudents, CreateStudent, UpdateStudent, DeleteStudent). Student creation may be subject to plan limits (see plan-based limits).

## Base route

`/API/V01/Students`

## Main operations

- **List**: `GET List` (query: pageNumber, pageSize, search, orderByField, orderByDesc) or `POST List` (body: GetEntityListQuery with OrderBy)
- **Get by ID**: `GET {id}`
- **Create**: `POST create`
- **Update**: `PUT {id}`
- **Delete**: `DELETE {id}`

Ordering uses `enStudentOrdering` (e.g. name, created date).

## Related documentation

- [Plan-based limits](../features-explanation-docs/plan-based-limits-explained.md) – plan may limit max students per tenant
- [Plan limits guide](../technical-docs/plan-limits-guide.md)

## Student profile fields (memorization progress)

- **`ReachedSurahId`** / **`ReachedSurahName`** (DTO): last surah the student reached (optional).
- **`ReachedAyahNumber`**: last ayah number within that surah (optional, 1-based; use with `ReachedSurahId`).
- **`CurrentJuzNumber`**: current juz for filtering / display (0 = not set in UI).

`CreateUpdateStudentDto` and multipart `create-with-user` accept **`ReachedAyahNumber`** and optional **`PendingSardCheckpointId`**.

### Sard checkpoint onboarding

When **`ReachedSurahId`** / **`ReachedAyahNumber`** are set on create or update:

1. **`PendingSardCheckpointId`** (optional): admin-selected pending checkpoint. Must be strictly **after** the student's reached position for their **`HifzDirection`**.
2. When omitted, the API sets pending to the **first checkpoint boundary after** the reached ayah (on a boundary, the **next** checkpoint is used so memorization is not blocked).
3. **`SardCheckpointPassSyncHelper`** marks earlier checkpoints as passed using bootstrap **`StudentCheckpointPass`** rows (`SardSessionId` null). Rows linked to real **`SardSession`** decisions are **never** removed or changed.
4. Invalid selections return localized **`SardPendingCheckpointBeforeReached`** / **`SardPendingCheckpointNotFound`** errors.

Implementation: `Alforqan.Core/Features/Sard/Helpers/SardCheckpointPassSyncHelper.cs`; wired in `StudentAddCommandHandler`, `StudentEditCommandHandler`, and `CreateStudentWithUserCommandHandler`.

## Create with user (`POST create-with-user`)

Multipart endpoint creates the tenant app user (Student role + default claims) and the student row in one request.

- **`DateOfBirth`** is optional for admin create (student may complete it on first-login profile). Plain `POST create` uses the same optional rule on create.
- **`StudentCode`** is optional; when omitted the API assigns the next `STU-####` code from both `AppUser.UserCode` and `Students.StudentCode` so codes never collide when rows are out of sync.
- **`UserCode`** on the app user matches **`StudentCode`** on the student record.

**Update note:** `PUT {id}` clears included navigation properties before save so **`ReachedSurahId`** / **`StageId`** scalars from the body are not overridden by a stale `ReachedSurah` / `StudyStage` loaded for display. The response is reloaded with includes so **`ReachedSurahName`** / **`StageName`** match the saved FKs.

## Controller

`Alforqan.Api/Controllers/Students/StudentsController.cs`
