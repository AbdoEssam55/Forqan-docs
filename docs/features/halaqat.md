# Halaqat

## Overview

Halaqat are study groups (circles) within a tenant. Each halaqa can have a teacher and enrolled students. The Halaqat API provides CRUD and paginated list with search. All endpoints require tenant authentication and appropriate tenant claims (e.g. ViewHalaqat, CreateHalaqa, UpdateHalaqa, DeleteHalaqa).

**Teacher list scope:** A user with the Teacher role (and not Admin or Supervisor) only receives halaqat assigned to that teacher (`TeacherId`). List pagination, get-by-id, and students-in-halaqa all use this scope. Admins and supervisors still see every halaqa.

**Halaqa session titles:** Create/update **generate** bilingual titles from the halaqa name and the session’s local date and weekday (`TitleAr`, `TitleEn`; `Title` = `TitleAr` for uniqueness). Client-supplied titles are ignored. Format: `{Name} - dd / MM / yyyy / {weekday}`; when another session already exists on the same local day in that halaqa, append ` / HH:mm` (seconds if still duplicate). `UtcOffsetMinutes` on create/update (default 180). Legacy rows keep manual `Title` only (`TitleAr`/`TitleEn` null) — no backfill. Within one halaqa, each non-deleted meeting `Title` must be unique (trimmed, case-insensitive). Index `IX_HalaqaSessions_HalaqaId_Title`. Max length 300.

## Base route

`/API/V01/Halaqat`

## Main operations

- **List**: `GET List` (query: pageNumber, pageSize, search) or `POST List` (body)
- **Get by ID**: `GET {id}`
- **Create**: `POST create`
- **Update**: `PUT {id}`
- **Delete**: `DELETE {id}`

Custom endpoints may exist for students-in-halaqa or teacher assignment; see controller for full list.

## Session evaluation + homework lock

- Open homework for the **target session** seeds evaluation ranges on first GET when the draft is still clean (no mistakes, no loaded revision segments).
- Changing ranges while homework is Open requires `ConfirmReplaceHomework` on upsert; otherwise `400`.
- Recording evaluation outcome marks matching Open homework **Done**.
- See `docs/technical-docs/homework-evaluation-lock.md` and `docs/features/student-assignments.md`.

## Controller

`Alforqan.Api/Controllers/Halaqat/HalaqatController.cs`
