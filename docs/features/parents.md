# Parents

## Overview

Parents (guardians) are linked to students via a many-to-many relationship (StudentParents). The Parents API provides CRUD and paginated list with search and ordering. All endpoints require tenant authentication and appropriate tenant claims (e.g. ViewParents, CreateParent, UpdateParent, DeleteParent).

## Base route

`/API/V01/Parents`

## Main operations

- **List**: `GET List` or `POST List` (with query body for pagination, search, orderBy)
- **Get by ID**: `GET {id}`
- **Create**: `POST create`
- **Update**: `PUT {id}`
- **Delete**: `DELETE {id}`

Ordering uses `enParentOrdering`. Linking parents to students is done via the StudentParents API.

## Related documentation

- [Student-parents feature](student-parents.md) – student–parent linking endpoints

## Controller

`Alforqan.Api/Controllers/Parents/ParentsController.cs`
