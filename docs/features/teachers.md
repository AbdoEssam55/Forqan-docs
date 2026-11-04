# Teachers

## Overview

Teachers belong to a tenant and can be assigned to halaqat. The Teachers API provides CRUD and paginated list with search and ordering. All endpoints require tenant authentication and appropriate tenant claims (e.g. ViewTeachers, CreateTeacher, UpdateTeacher, DeleteTeacher).

## Base route

`/API/V01/Teachers`

## Main operations

- **List**: `GET List` or `POST List` (with query body for pagination, search, orderBy)
- **Get by ID**: `GET {id}`
- **Create**: `POST create`
- **Update**: `PUT {id}`
- **Delete**: `DELETE {id}`

Ordering uses `enTeacherOrdering`.

## Controller

`Alforqan.Api/Controllers/Teachers/TeachersController.cs`
