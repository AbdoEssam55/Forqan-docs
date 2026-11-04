# Session evaluation report — Frontend Test Cases


## Metadata
- Project: `alforqan`
- Feature: `halaqat`
- Layer: `frontend`
- Routes: ``/{locale}/{tenantSlug}/reports/evaluation/{evaluationId}``
- APIs: -
- Related code: `src/app/[locale]/(tenant)/[tenantSlug]/reports/evaluation/[evaluationId]/session-evaluation-report-content.tsx`, `src/features/alforqan/evaluation/session-evaluation-report-section.tsx`, `src/features/alforqan/evaluation/session-evaluation-detail-readonly.tsx`, `src/features/alforqan/evaluation/memorization-readonly-ayah-viewer.tsx`, `src/features/alforqan/evaluation/revision-segment-readonly-viewer.tsx`, `src/features/alforqan/evaluation/revision-segment-helpers.ts`
- Related docs: `docs/explanation/tenant/reports-explained.md`

## Scope

- Digital evaluation summary, expandable read-only Quran detail, and PDF download for parents and staff with allowed access.
- Revision evaluations list all tested segments (`ayahsLoaded`) in summary and details, with per-segment ayah view modes.
- Memorization (hifz) evaluations use the same mistakes-only and all-tested ayah toggles in evaluation details.

## Test pages

- `/{locale}/{tenantSlug}/reports/evaluation/{evaluationId}`

## Test scenarios

- Parent linked to student opens report after notification.
- Teacher with halaqa access opens report and downloads PDF.
- Parent or student expands evaluation details to see marked words in ayah context.
- Teacher or parent opens a revision report with multiple tested segments and browses all tested ayahs per segment.
- Teacher, student, or parent opens a memorization (hifz) report and toggles mistakes-only vs all tested ayahs in details.

## Delivery waves

| Wave | Intent | Keys (representative) | Depends on (cross-file) |
| ---- | ------ | --------------------- | ----------------------- |
| 1 | Digital report — summary, mistakes, read-only detail | `ALFQ-HAL-REP-001`–`002`, `ALFQ-HAL-REP-005`–`006` | `ALFQ-HAL-EVA-011` on `ALFQ-HAL-REP-001` |
| 2 | PDF download and parent access edge | `ALFQ-HAL-REP-003`–`004` | In-file: `ALFQ-HAL-REP-001` on TC-04 |
| 3 | Revision report — all tested segments and ayah browse | `ALFQ-HAL-REP-007`–`011` | `ALFQ-HAL-REP-001`, revision evaluation with loaded segments |
| 4 | Memorization report — ayah view modes in details | `ALFQ-HAL-REP-012`–`015` | `ALFQ-HAL-REP-001`, submitted memorization evaluation |

Run after [evaluation](halaqat-evaluation-test-cases.md) (`ALFQ-HAL-EVA-011`).

## Test cases

### Wave 1 — Digital report

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-01 | ALFQ-HAL-REP-001 | Verify report page shows student and score summary | Main | User may view the report; session outcome is recorded | 1. Open the report URL for a known evaluation id. | 1. Section "Summary" appears<br>2. labels "Student", "Range", and "Score" appear with values<br>3. when outcome is recorded, "Session result" shows "Passed" or "Not passed". | ALFQ-HAL-EVA-017 | `src/app/[locale]/(tenant)/[tenantSlug]/reports/evaluation/[evaluationId]/session-evaluation-report-content.tsx` | Evaluation id after outcome recorded |
| TC-02 | ALFQ-HAL-REP-002 | Verify mistakes section shows count, weight, and total deduction by type | Main | Evaluation is submitted and contains at least one mistake | 1. Open the same report page.<br>2. Scroll to "Mistakes". | 1. A table appears with column headers matching "Mistake type", "Number of errors", "Weight per error", and "Total deduction" (or Arabic equivalents "نوع الخطأ", "عدد الأخطاء", "وزن الخطأ الواحد", "الخصم الكلى")<br>2. the "Mistake type" header is right-aligned<br>3. each row shows a type label (e.g. "Tajweed" / "التجويد"), a numeric count, a per-error weight, and a total deduction prefixed with "−". | ALFQ-HAL-REP-001 | `src/features/alforqan/evaluation/mistake-breakdown-table.tsx`, `src/features/alforqan/evaluation/evaluation-mistake-helpers.ts` | Submitted evaluation with graded mistakes |
| TC-03 | ALFQ-HAL-REP-005 | Verify show evaluation details reveals read-only Quran marks | Main | Submitted memorization (hifz) evaluation with at least one word-level mistake | 1. Open the memorization evaluation report.<br>2. In "Summary", choose "Show evaluation details".<br>3. Wait for ayahs to load. | 1. Section "Evaluation details" appears with range labels.<br>2. Uthmani ayah text shows at least one highlighted word.<br>3. Buttons "Mistakes only" / "All tested ayahs" may appear for ayah view; there are no evaluate-page mode tabs and no "Save draft" or "Submit evaluation" buttons. | ALFQ-HAL-REP-001 | `src/features/alforqan/evaluation/session-evaluation-detail-readonly.tsx`, `src/features/alforqan/evaluation/memorization-readonly-ayah-viewer.tsx` | Memorization evaluation with word marks |
| TC-04 | ALFQ-HAL-REP-006 | Verify hide evaluation details collapses the detail section | Main | Evaluation details are expanded | 1. Choose "Hide evaluation details". | 1. The Quran detail section is no longer visible<br>2. summary and mistakes table remain. | ALFQ-HAL-REP-005 | `src/features/alforqan/evaluation/session-evaluation-detail-readonly.tsx` | - |

### Wave 2 — PDF

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-05 | ALFQ-HAL-REP-003 | Verify download PDF triggers success feedback and full report content | Main | User may view own reports or halaqat per account | 1. Choose "Download PDF". | 1. A success toast appears with wording matching "PDF download started" (or Arabic equivalent) or the browser begins a file download named like "evaluation-{id}.pdf"<br>2. the PDF uses the site language (Arabic labels when the UI is Arabic, English when English) and includes the same sections as the on-screen report: "Summary" / "الملخص", mistakes table with type/count/weight/deduction columns, and "Evaluation details" / "تفاصيل التقييم" with Uthmani ayah text and colored word marks where applicable. | ALFQ-HAL-REP-001 | `src/app/[locale]/(tenant)/[tenantSlug]/reports/evaluation/[evaluationId]/session-evaluation-report-content.tsx` | - |
| TC-06 | ALFQ-HAL-REP-004 | Verify parent without link to student cannot open report | Edge | Parent account not linked to evaluated student | 1. Sign in as that parent.<br>2. Open another family's evaluation URL. | 1. A permission or error state appears<br>2. summary content for that child is not shown. | - | `src/app/[locale]/(tenant)/[tenantSlug]/reports/evaluation/[evaluationId]/page.tsx` | Foreign evaluation id |

### Wave 3 — Revision report segments and ayah browse

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-07 | ALFQ-HAL-REP-007 | Verify revision summary lists all tested segments even when some have zero mistakes | Main | Submitted revision evaluation with at least two `ayahsLoaded` segments; one segment has mistakes and one has none | 1. Open the revision evaluation report.<br>2. Read the summary card above "Show evaluation details". | 1. Label "Segments tested" / "الأجزاء المُختبرة" shows the loaded segment count.<br>2. Both tested segments appear with one compact range per segment (surah name and ayah from–to, without mushaf numbers in parentheses).<br>3. when mistake ayahs fall inside the segment range, no extra marked-ayah suffix is shown after the range.<br>4. the zero-mistake segment is listed even though it has no marked ayahs. | ALFQ-HAL-REP-001 | `src/features/alforqan/evaluation/session-evaluation-report-section.tsx`, `src/features/alforqan/evaluation/revision-segment-helpers.ts` | Revision evaluation with 2+ loaded segments |
| TC-08 | ALFQ-HAL-REP-008 | Verify evaluation details include zero-mistake tested segment with no-mistake message | Main | Same revision evaluation as TC-07 | 1. Choose "Show evaluation details".<br>2. Find the segment with zero mistakes. | 1. The zero-mistake segment header appears with its full tested range.<br>2. In default "Mistakes only" / "الأخطاء فقط" mode, message "No marks in this segment." / "لا أخطاء في هذا الجزء." appears.<br>3. no ayah list is shown for that segment in mistakes-only mode. | ALFQ-HAL-REP-007 | `src/features/alforqan/evaluation/session-evaluation-detail-readonly.tsx`, `src/features/alforqan/evaluation/revision-segment-readonly-viewer.tsx` | Segment with `ayahsLoaded` and zero marks |
| TC-09 | ALFQ-HAL-REP-009 | Verify per-segment ayah view defaults to mistakes only | Main | Revision evaluation with at least one segment that has word-level mistakes | 1. Expand evaluation details.<br>2. Inspect a segment that has mistakes before changing any toggle. | 1. Button "Mistakes only" / "الأخطاء فقط" is selected by default.<br>2. only ayahs with mistakes are shown in the Quran viewer for that segment. | ALFQ-HAL-REP-008 | `src/features/alforqan/evaluation/revision-segment-readonly-viewer.tsx` | Segment with marked ayahs |
| TC-10 | ALFQ-HAL-REP-010 | Verify all tested ayahs mode shows the full segment range | Main | Revision evaluation with a loaded segment spanning many ayahs (e.g. 2:1–2:60) and mistakes on only a few ayahs | 1. Expand evaluation details.<br>2. For that segment, choose "All tested ayahs" / "كل الآيات المُختبرة".<br>3. Wait for ayahs to load. | 1. Ayahs from the full tested range appear, not only marked ayahs.<br>2. marked words remain highlighted where mistakes exist.<br>3. unmarked ayahs in the range appear without highlights. | ALFQ-HAL-REP-009 | `src/features/alforqan/evaluation/revision-segment-readonly-viewer.tsx`, `src/features/alforqan/evaluation/load-quran-ayahs-in-range.ts` | Large single-surah segment |
| TC-11 | ALFQ-HAL-REP-011 | Verify all tested ayahs mode supports pagination, ayah filter, and text search | Main | Same large segment as TC-10; all-tested mode active | 1. In all-tested mode, change page size and go to page 2.<br>2. Set ayah min/max to a narrow range within the segment.<br>3. Enter a distinctive substring from an ayah in the search box. | 1. Pagination controls update the visible ayah page.<br>2. ayah min/max filters limit visible ayahs to the chosen range.<br>3. search narrows results to matching ayah text or shows "No ayahs match your filters." / "لا توجد آيات تطابق التصفية." when nothing matches. | ALFQ-HAL-REP-010 | `src/features/alforqan/evaluation/evaluation-ayah-browse-toolbar.tsx`, `src/features/alforqan/evaluation/use-evaluation-ayah-browse.ts` | Segment with 20+ ayahs |

### Wave 4 — Memorization ayah view modes

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-12 | ALFQ-HAL-REP-012 | Verify memorization details default to mistakes only | Main | Submitted memorization evaluation with word-level mistakes on a range spanning more ayahs than marked | 1. Open the memorization evaluation report.<br>2. Choose "Show evaluation details".<br>3. Before changing toggles, inspect the Quran viewer. | 1. Button "Mistakes only" / "الأخطاء فقط" is selected by default.<br>2. Only ayahs with mistakes appear in the viewer, not the full tested range. | ALFQ-HAL-REP-005 | `src/features/alforqan/evaluation/memorization-readonly-ayah-viewer.tsx` | Hifz eval with sparse marks |
| TC-13 | ALFQ-HAL-REP-013 | Verify memorization mistakes-only mode with zero mistakes shows no ayah list | Main | Submitted memorization evaluation with zero mistakes | 1. Open the memorization evaluation report.<br>2. Choose "Show evaluation details". | 1. "Mistakes only" / "الأخطاء فقط" is selected by default.<br>2. Message "No mistakes recorded" / "لا توجد أخطاء مسجّلة" appears.<br>3. No Quran ayah list is shown in mistakes-only mode. | ALFQ-HAL-REP-012 | `src/features/alforqan/evaluation/memorization-readonly-ayah-viewer.tsx` | Perfect hifz evaluation |
| TC-14 | ALFQ-HAL-REP-014 | Verify memorization all tested ayahs mode shows full range | Main | Same memorization evaluation as TC-12 | 1. With details expanded, choose "All tested ayahs" / "كل الآيات المُختبرة".<br>2. Wait for ayahs to load if needed. | 1. Ayahs from the full memorization range appear, not only marked ayahs.<br>2. Marked words remain highlighted where mistakes exist.<br>3. Unmarked ayahs in the range appear without word highlights. | ALFQ-HAL-REP-012 | `src/features/alforqan/evaluation/memorization-readonly-ayah-viewer.tsx`, `src/features/alforqan/evaluation/load-quran-ayahs-in-range.ts` | Hifz range wider than mistake ayahs |
| TC-15 | ALFQ-HAL-REP-015 | Verify memorization all tested ayahs browse supports pagination, ayah filter, and search | Main | Same evaluation as TC-14; all-tested mode active; range has many ayahs | 1. Change page size and go to page 2.<br>2. Set ayah min/max to a narrow range within the memorization range.<br>3. Enter a distinctive substring from an ayah in the search box. | 1. Pagination controls update the visible ayah page.<br>2. Ayah min/max filters limit visible ayahs to the chosen range.<br>3. Search narrows results to matching ayah text or shows "No ayahs match your filters." / "لا توجد آيات تطابق التصفية." when nothing matches. | ALFQ-HAL-REP-014 | `src/features/alforqan/evaluation/evaluation-ayah-browse-toolbar.tsx`, `src/features/alforqan/evaluation/use-evaluation-ayah-browse.ts`, `src/features/alforqan/evaluation/memorization-readonly-ayah-viewer.tsx` | Hifz range with 20+ ayahs |
