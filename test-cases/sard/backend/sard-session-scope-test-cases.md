# Sard session start scope — Backend Test Cases

## Metadata
- Project: `alforqan`
- Feature: `sard`
- Layer: `backend`
- Routes: `-`
- APIs: `POST /api/.../Sard/sessions`
- Related code: `Alforqan/Alforqan.Core/Features/Sard/Helpers/SardCheckpointHelper.cs`, `Alforqan/Alforqan.Core/Features/Sard/Commands/SardSessionMediators.cs`, `Alforqan/Alforqan.Core/Features/Sard/Shared/SardDtos.cs`
- Related docs: `_See create sard session scope fields._`
- Automation: `Tests/Alforqan.Core.Tests/Features/Sard/SardSessionScopeHelperTests.cs`

## Scope
- Create-session from/to: end is always the pending checkpoint; start is previous checkpoint by default, or an earlier checkpoint / mushaf start / free ayah per `CreateSardSessionDto.RespectSardCheckpoints` (default true). Not a tenant setting.

## Test pages
- API / xUnit helper

## Test scenarios
- Omitted start uses the segment after the previous checkpoint.
- Respect-on start must snap to mushaf start or a previous checkpoint.
- Respect-off start may be any ayah at or before the pending boundary.

## Delivery waves
- Wave 1 — Default and checkpoint starts (`ALFQ-SARD-BE-SCOPE-001`–`003`, `007`)
- Wave 2 — Free start and rejects (`ALFQ-SARD-BE-SCOPE-004`–`006`)

## Test cases

### Wave 1 — Default and checkpoint starts

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-01 | ALFQ-SARD-BE-SCOPE-001 | Verify omitted start uses the previous checkpoint through the pending one | Main | Plan has checkpoints at 2:10, 2:20, 2:30; pending is 2:30 | 1. Call `TryResolveCreateSessionScope` with no start fields. | 1. Error is null.<br>2. From is 2:21 (ayah after 2:20).<br>3. To is 2:30. | - | `Alforqan/Alforqan.Core/Features/Sard/Helpers/SardCheckpointHelper.cs` | Pending 2:30 |
| TC-02 | ALFQ-SARD-BE-SCOPE-002 | Verify start after an earlier checkpoint widens the range | Main | Same plan; pending 2:30 | 1. Call resolve with `scopeFromCheckpointId` of the 2:10 checkpoint. | 1. Error is null.<br>2. From is 2:11.<br>3. To is 2:30. | ALFQ-SARD-BE-SCOPE-001 | `Alforqan/Alforqan.Core/Features/Sard/Helpers/SardCheckpointHelper.cs` | Start after 2:10 |
| TC-03 | ALFQ-SARD-BE-SCOPE-003 | Verify start from mushaf start reaches the pending checkpoint | Main | Same plan; pending 2:30 | 1. Call resolve with `scopeFromHifzStart` true. | 1. Error is null.<br>2. From is 1:1.<br>3. To is 2:30. | ALFQ-SARD-BE-SCOPE-001 | `Alforqan/Alforqan.Core/Features/Sard/Helpers/SardCheckpointHelper.cs` | Hifz start |
| TC-04 | ALFQ-SARD-BE-SCOPE-007 | Verify a new create-session request respects checkpoints by default | Main | - | 1. Construct `CreateSardSessionDto`. | `RespectSardCheckpoints` is true. | - | `Alforqan/Alforqan.Core/Features/Sard/Shared/SardDtos.cs` | - |

### Wave 2 — Free start and rejects

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-05 | ALFQ-SARD-BE-SCOPE-004 | Verify a free start is allowed when checkpoints are not respected | Main | Same plan; pending 2:30 | 1. Call resolve with respect false and from 2:5. | 1. Error is null.<br>2. From is 2:5.<br>3. To is 2:30. | ALFQ-SARD-BE-SCOPE-001 | `Alforqan/Alforqan.Core/Features/Sard/Helpers/SardCheckpointHelper.cs` | Respect off; 2:5 |
| TC-06 | ALFQ-SARD-BE-SCOPE-005 | Verify a free start is rejected when checkpoints are respected | Main | Same plan; pending 2:30 | 1. Call resolve with respect true and from 2:5 (not a checkpoint start). | The error is `SARD_SCOPE_START_MUST_BE_CHECKPOINT`. | ALFQ-SARD-BE-SCOPE-004 | `Alforqan/Alforqan.Core/Features/Sard/Helpers/SardCheckpointHelper.cs` | Respect on; 2:5 |
| TC-07 | ALFQ-SARD-BE-SCOPE-006 | Verify a start past the pending checkpoint or the pending checkpoint itself is rejected | Edge | Same plan; pending 2:30 | 1. Call resolve with respect false and from 2:31.<br>2. Call resolve with respect true and `scopeFromCheckpointId` of the pending 2:30 row. | 1. From 2:31 returns `SARD_SCOPE_START_INVALID`.<br>2. Pending as start checkpoint returns `SARD_SCOPE_START_CHECKPOINT_INVALID`. | ALFQ-SARD-BE-SCOPE-001 | `Alforqan/Alforqan.Core/Features/Sard/Helpers/SardCheckpointHelper.cs` | Past pending |
