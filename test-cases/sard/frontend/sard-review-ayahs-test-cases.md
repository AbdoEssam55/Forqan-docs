# Sard review ayahs — Frontend Test Cases

## Metadata

- Project: `alforqan`
- Feature: `sard`
- Layer: `frontend`
- Routes: ``/{locale}/{tenantSlug}/sard/{sardSessionId}/review``
- APIs: `GET /api/.../Sard/sessions/{id}`
- Related code: `src/app/[locale]/(tenant)/[tenantSlug]/sard/[sardSessionId]/review/sard-review-content.tsx`, `src/features/alforqan/evaluation/revision-segment-readonly-viewer.tsx`
- Related docs: _See the project documentation indexes for the owning feature._

## Scope

- After a sard session is evaluated, the review/result page shows Quran ayahs grouped like revision segments: one block per surah that has mistakes, with surah/range header, then mistakes only vs all tested ayahs for that block.
- If there are no mistakes, one block covers the full sard scope.
- Mistake breakdown table remains when mistakes exist.
- No ayah viewer when the session has not been evaluated yet.

## Delivery waves

| Wave | Intent | Keys |
|------|--------|------|
| 1 | Review ayah viewer grouped by surah (revision-style) | `ALFQ-SARD-REV-001`–`003` |
| 2 | Loading copy on review page | `ALFQ-SARD-REV-004` |

## Test cases

### Wave 1 — Review ayah viewer

| ID | Key | Case | Type | Steps | Expected | Related code |
|----|-----|------|------|-------|----------|--------------|
| TC-01 | ALFQ-SARD-REV-001 | Verify evaluated sard review groups mistake ayahs by surah like revision | Main | 1. Open a sard session that is submitted or decided and has mistakes in at least two surahs.<br>2. Open ``/{tenantSlug}/sard/{sardSessionId}/review`` as admin or committee teacher.<br>3. Find the "Evaluation details" section.<br>4. Confirm each surah with mistakes has its own block with a "Segment {n}" title and a range line that includes the surah name (not only `4:3`).<br>5. In one block, confirm "Mistakes only" is selected by default and shows that surah's marked ayahs.<br>6. Click "All tested ayahs" in that block. | 1. Each surah with mistakes is a separate block with a range header that names the surah.<br>2. Buttons "Mistakes only" and "All tested ayahs" are visible in each block (Arabic: "الأخطاء فقط" / "كل الآيات المُختبرة").<br>3. "Mistakes only" shows Quran text only for marked ayahs in that surah.<br>4. "All tested ayahs" shows ayahs for that surah's sard-scope range with browse/pagination as needed.<br>5. The mistakes table still lists each mistake below the viewer. | `src/app/[locale]/(tenant)/[tenantSlug]/sard/[sardSessionId]/review/sard-review-content.tsx`, `src/features/alforqan/evaluation/revision-segment-readonly-viewer.tsx` |
| TC-02 | ALFQ-SARD-REV-002 | Verify evaluated sard with zero mistakes still shows the ayah viewer | Main | 1. Open a sard session that is evaluated with no mistakes recorded.<br>2. Open the sard review page. | 1. The "Evaluation details" section is visible.<br>2. With "Mistakes only" selected, the page shows "No marks in this segment." (Arabic: "لا أخطاء في هذا الجزء.").<br>3. "All tested ayahs" shows ayahs for the full sard scope.<br>4. No mistakes breakdown table appears. | `src/app/[locale]/(tenant)/[tenantSlug]/sard/[sardSessionId]/review/sard-review-content.tsx` |
| TC-03 | ALFQ-SARD-REV-003 | Verify unevaluated sard review does not show the ayah viewer | Main | 1. Open a sard session that is scheduled or in progress and not yet submitted with evaluation.<br>2. Open the review page if routed, or confirm review is only available after evaluation per product rules. | 1. No "Evaluation details" ayah viewer block appears.<br>2. No mistakes table appears. | `src/app/[locale]/(tenant)/[tenantSlug]/sard/[sardSessionId]/review/sard-review-content.tsx` |

### Wave 2 — Loading copy

| ID | Key | Case | Type | Requires | Steps | Expected | Related code |
|----|-----|------|------|----------|-------|----------|--------------|
| TC-04 | ALFQ-SARD-REV-004 | Verify sard review initial load shows localized loading text | Regression | ALFQ-SARD-REV-001 | 1. Hard-refresh the sard review page while the session request is in flight. | 1. A localized loading message appears (not bare "...").<br>2. After load, the review content replaces the loading state. | `src/app/[locale]/(tenant)/[tenantSlug]/sard/[sardSessionId]/review/sard-review-content.tsx` |
