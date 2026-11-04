# Student-Parents (linking)

## Overview

StudentParents is the many-to-many relationship between students and parents (guardians). One student can have multiple parents; one parent can have multiple children. The API provides endpoints to list, add, or remove links (and possibly get by student or by parent). All endpoints require tenant authentication and appropriate tenant claims.

## Base route

`/API/V01/StudentParents`

## Main operations

Typical operations: list links (e.g. by student ID or parent ID), add a link (associate a parent with a student, e.g. via StudentCode), remove a link. Exact routes depend on controller design (e.g. `GET ByStudent/{studentId}`, `POST`, `DELETE`). See controller for full list.

## Related documentation

- [Students](students.md) – student entity and CRUD
- [Parents](parents.md) – parent entity and CRUD

## Controller

`Alforqan.Api/Controllers/StudentParents/StudentParentsController.cs`
