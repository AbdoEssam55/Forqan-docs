# Homework evaluation lock

## GET evaluation for student

1. Load `SessionEvaluation` with `MasterRanges`, `RevisionSegments`, `Mistakes`.
2. Find Open `StudentAssignment` where `SessionId` = current session, matching student/halaqa/type.
3. When Open homework exists and eval is `InProgress` with **no mistakes** and **no `AyahsLoaded` segments**:
   - Copy homework ranges onto the evaluation (revision → `SessionEvaluationMasterRanges`; hifz → `From`/`To`).
   - `SyncEvaluationEnvelope` for revision.
4. Response DTO: `RangeLockedFromHomework`, `LockedHomeworkId`, `MasterRanges`.

## Upsert draft

1. If Open homework exists and input ranges differ:
   - Without `ConfirmReplaceHomework` → `400` + `SessionEvaluationConfirmReplaceHomeworkRequired`.
   - With confirm → cancel Open homework, create new Open row, `RemoveMistakesOutsideUnion`.
2. Revision: `SyncMasterRangesAsync` (no overlap), then `SyncEvaluationEnvelope`.
3. Memorization: set `From`/`To` from input; run sard gate.
4. `SyncRevisionSegmentsAsync`: each segment must reference `MasterRangeId` and stay within that master.

## Record outcome

After outcome is saved, matching Open homework for the session → `Status = Done`.

## PDF

When `MasterRanges.Count > 1`, list each master range in the revision summary section.
