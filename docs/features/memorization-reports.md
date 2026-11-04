# Memorization Reports

## Overview

Memorization reports record a student's memorization (and optionally revision) of Qur'an verses. The API is gated by the Reports module (`[RequiresModule(TenantModuleKey.Reports)]`). All endpoints require tenant authentication and tenant claims (e.g. ViewMemorizationReportsClaim).

## Base route

`/API/V01/MemorizationReports`

## Main operations

- **List**: `POST List` (body: GetMemorizationReportsListQuery – pagination, filters)
- **Get by ID**: `GET {id}`
- **Get ayah details**: `GET {id}/AyahDetails` – ayah-level details for a report
- **Create**: `POST create`
- **Update**: `PUT {id}`
- **Delete**: `DELETE {id}`

## Related documentation

- [Module gating](../features-explanation-docs/module-gating-on-tenant-endpoints-explained.md) – Reports module must be enabled for the tenant

## Controller

`Alforqan.Api/Controllers/MemorizationReports/MemorizationReportsController.cs`
