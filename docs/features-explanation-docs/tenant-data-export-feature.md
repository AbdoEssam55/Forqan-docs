# Tenant Data Export Feature (GDPR Compliance)

## Overview

The tenant data export feature allows platform (Host) administrators to export a tenant's full database in a structured, machine-readable way. This supports **GDPR Article 20 – Right to data portability**: data subjects (and, in a B2B context, tenants acting on behalf of their users) must be able to receive their personal data in a commonly used, machine-readable format.

Export is triggered via a Host-only API endpoint. Data is read from the tenant's database using the same pattern as other tenant-scoped Host operations (e.g. refresh token revocation).

## Problem Statement

- **Regulatory:** GDPR Article 20 requires that data be provided in a structured, commonly used, machine-readable format when requested.
- **Operational:** Platform admins need a single, auditable way to produce a full tenant data export for compliance, support, or migration.
- **Security:** Export contains sensitive tenant data; access must be restricted to authorized Host users and connection strings must not be logged or exposed.

## Architecture and Design Decisions

### Endpoint and Authorization

- **Route:** `POST /API/V01/Host/Tenants/{tenantId}/Export`
- **Body:** `TenantExportRequest` with `Format` (`Json` or `Csv`).
- **Authorization:** Host scheme + `ViewTenantsClaim` (export is an extension of viewing tenant context). No new claim for MVP.
- **Response:** On success, the response is the file stream (`application/json` or `application/zip`) with `Content-Disposition: attachment` and a filename such as `tenant-{code}-export-{date:yyyyMMddHHmmss}.json` or `.zip`.

### Tenant Database Access

- **Pattern:** Same as `TenantRefreshTokenRevocationService`: resolve tenant by ID from Host DB via `ITenantRepo`, decrypt connection string if encryption is enabled (`EncryptionSettings`, `ISecretsProtector`), build `DbContextOptions<AppDbContext>` and instantiate `AppDbContext` for that tenant.
- **Read-only:** All queries use `AsNoTracking()`; no writes to the tenant DB.
- **Scope:** Only the requested tenant; tenant ID comes from the route and is validated against the Host DB.

### Export Logic (TenantDataExportService)

- **Entity discovery:** Uses `context.Model.GetEntityTypes()` to iterate over mapped entity types; shadow/owned types can be skipped as needed.
- **JSON format:** Builds an object with `metadata` (export date UTC, tenant Id/Code/Name) and `tables` (dictionary: table name → list of rows as key-value dictionaries). Serialized to UTF-8 JSON (camelCase, no circular references). Content-Type: `application/json`.
- **CSV format:** One CSV per table (header = property names, rows = values with proper escaping). All CSVs are packed into a single ZIP file. Content-Type: `application/zip`.
- **Limits:** A configurable max total rows (e.g. 100,000) is enforced; if exceeded, the service returns 413 (Request Entity Too Large) with a localized message. Document async export + download link as a future enhancement.

### Security and Compliance

- Export is **Host-only**: only users authenticated with the Host scheme and with the appropriate claim can call the endpoint.
- Tenant ID is taken from the route; the service ensures the tenant exists in the Host DB (no cross-tenant access).
- Connection strings are decrypted only in memory and are not logged or included in the response.
- Export files are served over HTTPS in the same request; optional encryption of the export file itself is a future enhancement.

## How It Works

1. Platform admin calls `POST /API/V01/Host/Tenants/{tenantId}/Export` with body `{ "format": "Json" }` or `{ "format": "Csv" }`.
2. The controller authorizes (Host + ViewTenantsClaim), then calls `ITenantDataExportService.ExportAsync(tenantId, format, cancellationToken)`.
3. The service loads the tenant from the Host DB; if not found, returns 404.
4. The service decrypts the tenant connection string (if encryption is enabled), creates an `AppDbContext` for that tenant, and enumerates entity types.
5. For each table, data is loaded with `AsNoTracking()`, converted to a serializable shape (dictionaries for JSON; CSV rows for CSV), and the total row count is checked against the limit.
6. If the limit is exceeded, the service returns 413 with a localized “export too large” message.
7. Otherwise, the service builds the JSON document or ZIP and returns `TenantExportResult` (byte[] content, filename, content type).
8. The controller returns `File(content, contentType, fileName)` so the client receives the download in the same response.

## Performance and Limits

- **Synchronous MVP:** The first version is synchronous; the full export is produced and streamed in the same request.
- **Row limit:** A max total rows (e.g. 100,000) is enforced to avoid long-running requests and OOM; when exceeded, the API returns 413 with a clear message.
- **Streaming:** Where possible, serialization targets the response stream to reduce memory usage; for very large tenants, async export with a download link is the intended future enhancement.

## Localization

- Error messages use `IStringLocalizer` and keys in `SharedResourcesKeys.Tenant` (e.g. `Tenant.NotFound`, `Tenant.ExportTooLarge`, `Tenant.ExportFailed`).
- English and Arabic entries are provided in `en.json` and `ar.json` under the Api Localization folder.

## Future Enhancements (Out of Scope for MVP)

- **Async export + download link:** For large tenants, run export in a background job and return a link to download the file when ready.
- **Full SQL dump:** Schema + data as a SQL script (documented as future).
- **Encryption of export file:** Optional password or key-based encryption of the export file before download.
- **Frontend UI:** Roadmap specifies the backend endpoint only; a Host admin UI for triggering export can be added later.

## Reference

- Plan: Tenant data export/backup (GDPR) implementation plan.
- Same tenant DB access pattern: `TenantRefreshTokenRevocationService`.
- Roadmap: `docs/multi-tenant-platform-scope-and-roadmap.md` § 8.10.
