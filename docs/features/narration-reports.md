# Narration Reports

## Overview

Narration reports record khatmah/juz narration (recitation) for students. The API is gated by the Reports module (`[RequiresModule(TenantModuleKey.Reports)]`). All endpoints require tenant authentication and tenant claims (e.g. ViewNarrationReportsClaim).

## Base route

`/API/V01/NarrationReports`

## Main operations

- **List**: `POST List` (body: query with pagination and filters)
- **Get by ID**: `GET {id}`
- **Create**: `POST create`
- **Update**: `PUT {id}`
- **Delete**: `DELETE {id}`

## Related documentation

- [Module gating](../features-explanation-docs/module-gating-on-tenant-endpoints-explained.md) – Reports module must be enabled for the tenant

## Controller

`Alforqan.Api/Controllers/NarrationReports/NarrationReportsController.cs`
