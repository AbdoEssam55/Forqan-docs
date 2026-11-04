# Daily revision recorder — Frontend Test Cases

## Metadata

- Layer: frontend
- Related code: `src/features/alforqan/daily-revision/use-revision-recorder.ts`, `src/features/alforqan/daily-revision/daily-revision-panel.tsx`
- Related docs: `Alforqan-Frontend/docs/technical/tenant/daily-revision.md`
- Automation: -

## Scope

Optional microphone recording capped at five minutes.

## Test cases

### Recorder

| ID | Key | Case | Steps | Expected |
| --- | --- | --- | --- | --- |
| TC-01 | ALFQ-REVDAY-AUD-005 | Verify recorder stops at five minutes and submit without blob works | 1. Open daily revision submit on a due day.<br>2. Start recording; wait until auto-stop at 5:00 (or simulate max duration).<br>3. Deny microphone on a second attempt and submit without recording. | 1. Recording stops at five minutes.<br>2. Submit succeeds with no audio blob when mic denied. |
