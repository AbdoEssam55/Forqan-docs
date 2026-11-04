# Sard session list and view scope — Backend Test Cases

## Metadata
- Project: `alforqan`
- Feature: `sard`
- Layer: `backend`
- Routes: `-`
- APIs: `GET /api/.../Sard/sessions`, `GET /api/.../Sard/sessions/{id}`, `GET /api/.../Sard/sessions/{id}/certificate.html`
- Related code: `Alforqan/Alforqan.Core/Features/Sard/Commands/SardSessionMediators.cs`, `Alforqan/Alforqan.Core/Features/Sard/Helpers/SardSessionAccessHelper.cs`, `Alforqan/Alforqan.Api/Controllers/Sard/SardController.cs`
- Related docs: `Alforqan-Frontend/docs/technical/tenant/sard-session-access.md`
- Automation: `Tests/Alforqan.Core.Tests/Features/Sard/SardSessionAccessHelperTests.cs`

## Scope
- Cert teachers see only sard sessions they are assigned on unless they have Manage sard.
- Manage sard sees every session in list and get-by-id.

## Test cases

### List and view scope

| ID | Key | Case | Type | Steps | Expected | Related code |
|----|-----|------|------|-------|----------|--------------|
| TC-01 | ALFQ-SARD-BE-LIST-001 | Verify Manage sard can view any session without being assigned | Main | 1. Call `CanViewSessionAsync` with `hasManageSard` true and a session assigned to another teacher. | The result is true. | `Alforqan/Alforqan.Core/Features/Sard/Helpers/SardSessionAccessHelper.cs` |
| TC-02 | ALFQ-SARD-BE-LIST-002 | Verify an assigned cert teacher can view their session | Main | 1. Call `CanViewSessionAsync` with `hasManageSard` false and `actingUserId` matching a row on `session.Teachers`. | The result is true. | `Alforqan/Alforqan.Core/Features/Sard/Helpers/SardSessionAccessHelper.cs` |
| TC-03 | ALFQ-SARD-BE-LIST-003 | Verify an unassigned cert teacher cannot view another teacher's session | Edge | 1. Call `CanViewSessionAsync` with `hasManageSard` false, no family access, and `actingUserId` not on `session.Teachers`. | The result is false. | `Alforqan/Alforqan.Core/Features/Sard/Helpers/SardSessionAccessHelper.cs` |
