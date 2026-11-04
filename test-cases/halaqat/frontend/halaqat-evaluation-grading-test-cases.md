# Halaqa evaluation grading — Frontend Test Cases

## Metadata
- Project: `alforqan`
- Feature: `halaqat`
- Layer: `frontend`
- Routes: `/{locale}/{tenantSlug}/halaqat/{id}/sessions/{sessionId}/evaluate/{studentId}`
- APIs: Session evaluation GET/upsert with `gradingPreview` and allowance override fields
- Related code: `src/features/alforqan/evaluation/grading-allowance-override-panel.tsx`, `src/features/alforqan/evaluation/negative-score-display.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/halaqat/[id]/sessions/[sessionId]/evaluate/[studentId]/halaqa-evaluation-content.tsx`, `src/features/alforqan/evaluation/halaqa-revision-evaluation-panel.tsx`
- Related docs: `docs/technical/tenant/grading-settings.md`

## Scope

- Teacher allowance override on memorization and revision evaluate screens.
- Live score preview with negative (red) styling and stamp.
- Autosave persists override fields with draft upsert.

## Test cases

### Evaluate — allowance override

| ID | Key | Case | Type | Steps | Expected |
|----|-----|------|------|-------|----------|
| TC-01 | ALFQ-HAL-EVA-GRA-001 | Verify grading preview shows admin allowance on memorization evaluate | Main | 1. Open memorization evaluate for a student with marks loaded.<br>2. Scroll to the allowance panel. | 1. Panel title matches "Mistake allowance" (or Arabic equivalent).<br>2. Each active type shows admin → computed allowed counts.<br>3. Pages in range and max score are visible. |
| TC-02 | ALFQ-HAL-EVA-GRA-002 | Verify teacher can enable whole-range override and autosave | Main | 1. Turn on override toggle.<br>2. Choose "Whole range (integers)".<br>3. Set memorization allowed to `0`.<br>4. Wait for autosave or tap "Save draft". | 1. Score preview updates after save.<br>2. Reload page shows override still enabled with memorization `0`. |
| TC-03 | ALFQ-HAL-EVA-GRA-003 | Verify locked override shows message only | Edge | 1. In grading settings, lock teacher override for memorization scope.<br>2. Open memorization evaluate. | Panel shows "Teacher allowance override is locked in grading settings." with no editable fields. |
| TC-04 | ALFQ-HAL-EVA-GRA-004 | Verify negative score shows red value and stamp | Main | 1. Mark more mistakes than allowed (e.g. many حفظ marks on one page).<br>2. Observe score line. | 1. Score displays in red.<br>2. Badge shows "Below zero" (or Arabic equivalent). |
| TC-05 | ALFQ-HAL-EVA-GRA-005 | Verify revision evaluate uses same allowance panel and tested-only note | Main | 1. Open revision tab evaluate.<br>2. Load a segment and add marks.<br>3. Open score summary on outcome modal or report. | 1. Allowance panel is present before submit.<br>2. Score summary shows tested-only note for revision.<br>3. Single score line (no dual master/loaded cards). |
