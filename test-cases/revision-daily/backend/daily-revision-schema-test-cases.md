# Daily revision schema — Backend Test Cases

## Metadata

- Layer: backend
- Related code: `Alforqan/Alforqan.Data/Entities/StudentRevisionDailyLogs/StudentRevisionDailyLog.cs`, `Alforqan/Alforqan.Data/Enums/RevisionListenerRelation.cs`
- Related docs: `Alforqan-Backend/docs/technical-docs/daily-revision-log.md`
- Automation: `Tests/Alforqan.Core.Tests/Features/DailyRevision/DailyRevisionSchemaTests.cs`

## Scope

Entity shape and unique index on assignment + revision date.

## Test cases

### Model

| ID | Key | Case | Steps | Expected | Related code |
| --- | --- | --- | --- | --- | --- |
| TC-01 | ALFQ-REVDAY-SCHEMA-001 | Verify listener enum and unique index | 1. Run automated schema tests. | 1. `RevisionListenerRelation` includes Father=0 through Self=4.<br>2. `StudentRevisionDailyLog` has unique index on `(AssignmentId, RevisionDate)`. | `StudentRevisionDailyLog.cs` |
