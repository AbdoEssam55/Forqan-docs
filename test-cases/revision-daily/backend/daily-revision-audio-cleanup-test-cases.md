# Daily revision audio cleanup — Backend Test Cases

## Metadata

- Layer: backend
- Related code: `Alforqan/Alforqan.Core/Features/DailyRevision/Helpers/DailyRevisionAudioMetadataHelper.cs`, `Alforqan/Alforqan.Core/Features/DailyRevision/Helpers/DailyRevisionAudioCleanupHelper.cs`, `Alforqan/Alforqan.Core/Features/HalaqaSessions/Commands/RecordSessionEvaluationOutcomeCommand.cs`, `Alforqan/Alforqan.Api/BackgroundServices/DailyRevisionAudioCleanupBackgroundService.cs`
- Related docs: `Alforqan-Backend/docs/technical-docs/daily-revision-log.md`
- Automation: `Tests/Alforqan.Core.Tests/Features/DailyRevision/DailyRevisionAudioMetadataHelperTests.cs`, `Tests/Alforqan.Core.Tests/Features/DailyRevision/DailyRevisionAudioCleanupHelperTests.cs`

## Scope

Outcome audio purge and seven-day retention job.

## Test cases

### Cleanup

| ID | Key | Case | Steps | Expected | Related code |
| --- | --- | --- | --- | --- | --- |
| TC-01 | ALFQ-REVDAY-CLN-001 | Verify outcome clears audio only | 1. Run CLN-001 unit test. | 1. Audio columns cleared.<br>2. Range and listener fields unchanged. | `DailyRevisionAudioMetadataHelperTests.cs` |
| TC-02 | ALFQ-REVDAY-CLN-002 | Verify seven-day selection | 1. Run CLN-002 unit test with fixed `utcNow`. | Only logs with `AudioStoredAt` older than 7 days are selected. | `DailyRevisionAudioCleanupHelperTests.cs` |
