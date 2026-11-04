# Halaqa session evaluation — Frontend Test Cases

## Metadata
- Project: `alforqan`
- Feature: `halaqat`
- Layer: `frontend`
- Routes: ``/{locale}/{tenantSlug}/halaqat/{halaqaId}/sessions/{sessionId}/evaluate/{studentId}``
- APIs: `POST .../evaluations/students/{studentId}/submit`, `POST .../evaluations/students/{studentId}/outcome`
- Related code: `src/app/[locale]/(tenant)/[tenantSlug]/halaqat/[id]/sessions/[sessionId]/evaluate/[studentId]/halaqa-evaluation-content.tsx`, `src/features/alforqan/evaluation/halaqa-evaluation-marking-section.tsx`, `src/features/alforqan/evaluation/session-evaluation-outcome-modal.tsx`, `src/features/alforqan/evaluation/session-evaluation-report-section.tsx`, `src/features/alforqan/evaluation/use-persisted-evaluation-mode.ts`, `src/features/alforqan/evaluation/use-evaluation-ayah-browse.ts`, `src/features/alforqan/evaluation/evaluation-ayah-browse-toolbar.tsx`
- Related docs: `docs/technical/tenant/halaqat-guide.md`, `docs/explanation/tenant/halaqat-explained.md`, `docs/technical/tenant/hifz-direction-progress.md`, `docs/technical/tenant/evaluation-autosave.md`

## Impact and status

**Per-key impact:** The **Related code** column below scopes `tc impact` to range, marking, or submit keys. File metadata is the fallback when a row uses `-`.

| If you changed… | Re-run keys (minimum) |
| ----------------- | --------------------- |
| Range / juz / surah / ayah inputs, load ayahs | `ALFQ-HAL-EVA-001`–`005` |
| Quick / tap / full-ayah / ayah-counter marking | `ALFQ-HAL-EVA-007`–`009`, `ALFQ-HAL-EVA-019`–`022`, `ALFQ-HAL-EVA-024`–`026` |
| Ayah browse (pagination, filter, search) | `ALFQ-HAL-EVA-027`–`030` |
| Save draft / autosave | `ALFQ-HAL-EVA-010`, `ALFQ-HAL-EVA-032`–`035` |
| Submit + outcome modal | `ALFQ-HAL-EVA-011`–`018`, `ALFQ-HAL-EVA-039`–`041` |
| Did not memorize (failed) | `ALFQ-HAL-EVA-039`–`041` |
| Outcome attendance (unmarked student) | `ALFQ-HAL-EVA-036`–`038` |
| From-end memorization submit (sard gate) | `ALFQ-HAL-EVA-023`, `ALFQ-HAL-EVA-031` |
| Submitted read-only UI | `ALFQ-HAL-EVA-006` |

## Scope

- Default range, ayah loading, marking modes (quick, tap, ayah counters, full ayah), draft save, submit, session outcome, and read-only submitted state.

## Test pages

- `/{locale}/{tenantSlug}/halaqat/{halaqaId}/sessions/{sessionId}/evaluate/{studentId}`

## Delivery waves

| Wave | Intent | Keys | Depends on |
| ---- | ------ | ---- | ---------- |
| 1 | Range and ayah loading | `ALFQ-HAL-EVA-001`–`005` | `ALFQ-HAL-SES-003` |
| 2 | Marking modes | `ALFQ-HAL-EVA-007`–`009`, `ALFQ-HAL-EVA-019`–`022`, `ALFQ-HAL-EVA-024`–`026` | `ALFQ-HAL-EVA-004` |
| 2b | Ayah browse while marking | `ALFQ-HAL-EVA-027`–`030` | `ALFQ-HAL-EVA-004` |
| 3 | Read-only, save, submit, outcome | `ALFQ-HAL-EVA-006`, `ALFQ-HAL-EVA-010`–`018` | In-file chain; `ALFQ-HAL-EVA-017` → reports |
| 6 | Outcome attendance if not already marked | `ALFQ-HAL-EVA-036`–`038` | `ALFQ-HAL-EVA-011` |
| 5 | Autosave / refresh restore | `ALFQ-HAL-EVA-032`–`035` | `ALFQ-HAL-EVA-004` |
| 4 | Reports (outcome on report) | See [halaqat-reports-test-cases.md](halaqat-reports-test-cases.md) | `ALFQ-HAL-EVA-017` |

## Test cases

### Wave 1 — Range and ayah loading

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-01 | ALFQ-HAL-EVA-001 | Verify default range fields match student profile | Main | Student profile has juz "1", reached surah "Al-Baqarah (2)", last ayah "20" | 1. Open the evaluate page for that student. | 1. "From juz" shows "Juz 1".<br>2. "From surah" shows "Al-Baqarah (2)".<br>3. "From ayah" shows "20".<br>4. "To surah" defaults to "Al-Baqarah (2)". | ALFQ-HAL-SES-003 | `src/features/alforqan/evaluation/load-quran-ayahs-in-range.ts`, `src/features/alforqan/evaluation/evaluation-quran-range.ts` | Profile: juz "1"<br>surah "Al-Baqarah (2)"<br>last ayah "20" |
| TC-02 | ALFQ-HAL-EVA-002 | Verify surah fields use searchable list with name and number | Main | Ayahs not yet loaded | 1. Open the evaluate page.<br>2. Click "From surah" dropdown. | 1. A searchable list appears.<br>2. Each option shows the surah name with a number in parentheses (for example "Al-Baqarah (2)"). | ALFQ-HAL-SES-003 | `src/components/ui/searchable-select.tsx`, `src/features/alforqan/components/surah-ayah-number-input.tsx` | - |
| TC-03 | ALFQ-HAL-EVA-003 | Verify changing surah to another juz updates the juz field | Main | "From juz" shows "Juz 1";<br>"From surah" shows "Al-Baqarah (2)" | 1. Open "From surah".<br>2. Select a surah from a different juz (for example "An-Nisa (4)" in Juz 4). | "From juz" updates to match the selected surah (for example "Juz 4"). | ALFQ-HAL-SES-003 | `src/features/alforqan/evaluation/evaluation-quran-range.ts` | - |
| TC-04 | ALFQ-HAL-EVA-004 | Verify loading ayahs shows success and displays text | Main | Range set: "From surah" "Al-Baqarah (2)", "From ayah" "20", "To surah" "Al-Baqarah (2)", "To ayah" "22" | 1. Click "Load ayahs for range". | 1. A success toast appears mentioning the loaded count (for example "3 ayahs loaded").<br>2. Ayah text becomes visible on the page. | ALFQ-HAL-SES-003 | `src/features/alforqan/evaluation/load-quran-ayahs-in-range.ts` | - |
| TC-05 | ALFQ-HAL-EVA-005 | Verify evaluation opens with neutral defaults when student has no reached surah | Edge | Student profile has no reached surah set | 1. Open the evaluate page for that student. | 1. "From juz" shows the not-set option (for example "Not set").<br>2. "From surah" and "From ayah" allow choosing a range.<br>3. User can set "To surah", "To ayah", and click "Load ayahs for range" successfully. | ALFQ-HAL-SES-003 | `src/features/alforqan/evaluation/evaluation-quran-range.ts` | Student with no reached surah |
| TC-06 | ALFQ-HAL-EVA-006 | Verify submitted evaluation disables editing | Main | An evaluation is already submitted and outcome recorded for this student | 1. Open the evaluate page for that student. | 1. Range inputs are disabled.<br>2. Marking controls are disabled.<br>3. A "View report" link appears. | ALFQ-HAL-EVA-017 | `src/app/[locale]/(tenant)/[tenantSlug]/halaqat/[id]/sessions/[sessionId]/evaluate/[studentId]/halaqa-evaluation-content.tsx` | Student with completed submit + outcome |

### Wave 2 — Marking modes (quick and ayah counters)

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-07 | ALFQ-HAL-EVA-007 | Verify quick mode toggles a word mark with the selected type | Main | Ayahs loaded; mode "Quick buttons" | 1. Choose mistake type "Tajweed".<br>2. Tap one word in the ayah text. | 1. The word shows a highlight.<br>2. "Marks recorded" increases by one. | ALFQ-HAL-EVA-004 | `src/features/alforqan/components/quran-viewer`, `src/features/alforqan/evaluation/evaluation-mistake-helpers.ts` | - |
| TC-08 | ALFQ-HAL-EVA-008 | Verify ayah counter mode changes totals | Main | Ayahs loaded; mode "Ayah counters" | 1. Open "Ayah counters".<br>2. Increase "Mem" count for the first ayah once. | 1. The counter shows "1" for that ayah.<br>2. "Marks recorded" includes that count. | ALFQ-HAL-EVA-004 | `src/features/alforqan/components/quran-viewer` | - |
| TC-09 | ALFQ-HAL-EVA-009 | Verify switching modes keeps word marks when returning from ayah mode | Edge | User added at least one word mark then switched to ayah mode and back | 1. Add a word mark in quick mode.<br>2. Switch to "Ayah counters", then back to "Quick buttons". | The same word still shows the highlight after returning. | ALFQ-HAL-EVA-004 | `src/features/alforqan/components/quran-viewer` | - |

### Wave 3 — Save draft and submit

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-10 | ALFQ-HAL-EVA-010 | Verify save draft shows confirmation | Main | Marks or notes changed | 1. Click "Save draft". | 1. Status text shows "All marks saved" / "تم حفظ جميع العلامات" (or "Saving…" then saved).<br>2. Reloading the page still shows the same marks and notes. | ALFQ-HAL-EVA-004 | `src/app/[locale]/(tenant)/[tenantSlug]/halaqat/[id]/sessions/[sessionId]/evaluate/[studentId]/halaqa-evaluation-content.tsx`, `src/features/alforqan/evaluation/use-evaluation-autosave.ts`, `src/features/alforqan/evaluation/evaluation-save-status-bar.tsx` | Notes: "Good effort" |
| TC-11 | ALFQ-HAL-EVA-011 | Verify submit opens outcome modal with result summary | Main | Evaluation not yet submitted; ayahs loaded | 1. Click "Submit evaluation".<br>2. Expand "Show score breakdown" under the marks count. | 1. A modal opens showing score summary (score, range, mistake count).<br>2. "Show score breakdown" lists memorization, tajweed, and tashkeel each with mark count and points deducted (−pts).<br>3. "Session result" offers "Passed" and "Failed".<br>4. No parent notification is sent yet.<br>5. If this student has no saved attendance, "Attendance for this student" / "حضور هذا الطالب" is shown with Present, Absent, and Late (not Excused). | ALFQ-HAL-EVA-010 | `src/features/alforqan/evaluation/session-evaluation-outcome-modal.tsx`, `src/features/alforqan/evaluation/evaluation-mistake-helpers.ts` | - |
| TC-12 | ALFQ-HAL-EVA-012 | Verify failed outcome submits without updating student progress | Main | Outcome modal open after submit | 1. If "Attendance for this student" is shown, choose "Present" / "حاضر".<br>2. Choose "Failed".<br>3. Confirm the modal. | 1. Success confirmation appears.<br>2. Student profile reached surah/ayah unchanged.<br>3. "View report" appears.<br>4. Report shows "Not passed". | ALFQ-HAL-EVA-011 | `src/features/alforqan/evaluation/session-evaluation-outcome-modal.tsx` | Profile snapshot before test |
| TC-13 | ALFQ-HAL-EVA-013 | Verify passed with revision only does not update progress | Main | Outcome modal open; student has known reached surah/ayah | 1. Choose "Passed".<br>2. Choose not to update memorization (revision only).<br>3. Confirm. | 1. Report shows "Passed" and line "Memorization progress: not updated".<br>2. Student profile reached surah/ayah unchanged. | ALFQ-HAL-EVA-011 | `src/features/alforqan/evaluation/session-evaluation-outcome-modal.tsx` | - |
| TC-14 | ALFQ-HAL-EVA-014 | Verify passed with progress update defaults to evaluation end | Main | Outcome modal open; range ends at "Al-Baqarah (2)" ayah "55" | 1. Choose "Passed".<br>2. Choose to update memorization.<br>3. Confirm without changing reached fields. | 1. Student profile shows reached surah "Al-Baqarah (2)" and last ayah "55".<br>2. "From juz" on a later evaluation reflects the updated juz. | ALFQ-HAL-EVA-011 | `src/features/alforqan/evaluation/session-evaluation-outcome-modal.tsx` | To ayah "55" |
| TC-15 | ALFQ-HAL-EVA-015 | Verify teacher can certify a lower ayah within the range | Main | Range ends at ayah "55"; profile below "52" | 1. Choose "Passed" and update memorization.<br>2. Set reached ayah to "52".<br>3. Confirm any lower-progress prompt.<br>4. Confirm. | Student profile last ayah is "52", not "55". | ALFQ-HAL-EVA-011 | `src/features/alforqan/evaluation/session-evaluation-outcome-modal.tsx` | - |
| TC-16 | ALFQ-HAL-EVA-016 | Verify certified position outside evaluation range is blocked | Edge | Outcome modal open | 1. Choose "Passed" and update memorization.<br>2. Set reached ayah beyond the evaluation "To ayah".<br>3. Try to confirm. | 1. An error explains the position must stay within the evaluation range<br>2. profile is not updated. | ALFQ-HAL-EVA-011 | `src/features/alforqan/evaluation/session-evaluation-outcome-modal.tsx` | - |
| TC-17 | ALFQ-HAL-EVA-017 | Verify outcome completion enables report and notifications | Main | Outcome recorded (any result) | 1. Open "View report".<br>2. As linked parent or student, check notifications. | 1. Report page loads with session result visible.<br>2. Parent and student each have a notification about the evaluation. | ALFQ-HAL-EVA-012 | `src/features/alforqan/evaluation/session-evaluation-outcome-modal.tsx` | - |
| TC-18 | ALFQ-HAL-EVA-018 | Verify optional study stage updates only when progress is updated | Main | Outcome modal open; institute has multiple study stages | 1. Choose "Passed", update memorization, pick a different study stage.<br>2. Confirm. | Student profile study stage matches the selection. | ALFQ-HAL-EVA-011 | `src/features/alforqan/evaluation/session-evaluation-outcome-modal.tsx` | - |

### Wave 2 — Marking modes (tap and full ayah)

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-19 | ALFQ-HAL-EVA-019 | Verify tap mode opens a word menu with three mistake types | Main | Ayahs loaded; mode "Tap to flag" | 1. Tap one unmarked word in the ayah text. | 1. A small menu appears with "Memorization", "Tajweed", and "Tashkeel".<br>2. "No mistake" is not shown. | ALFQ-HAL-EVA-004 | `src/features/alforqan/evaluation/word-mistake-picker.tsx`, `src/features/alforqan/components/quran-viewer` | - |
| TC-20 | ALFQ-HAL-EVA-020 | Verify tap mode applies and replaces a single word mark | Main | Ayahs loaded; mode "Tap to flag" | 1. Tap one word.<br>2. Choose "Tajweed".<br>3. Tap the same word again to reopen the menu.<br>4. Choose "Memorization". | 1. After step 2 the word is highlighted and "Marks recorded" increases by one.<br>2. After step 4 the word stays highlighted once with the memorization style (not two marks). | ALFQ-HAL-EVA-004 | `src/features/alforqan/evaluation/word-mistake-picker.tsx`, `src/features/alforqan/evaluation/evaluation-mistake-helpers.ts` | - |
| TC-21 | ALFQ-HAL-EVA-021 | Verify full ayah button marks every word with all three types | Main | Ayahs loaded; mode "Tap to flag" or "Quick buttons" | 1. Click "Full ayah" on the first loaded ayah. | 1. Every word in that ayah shows a highlight.<br>2. The button label shows the active full-ayah state (for example "Full ayah ✓").<br>3. "Marks recorded" increases by three times the number of words in that ayah. | ALFQ-HAL-EVA-004 | `src/features/alforqan/evaluation/evaluation-mistake-helpers.ts`, `src/features/alforqan/components/quran-viewer` | Ayah with at least two words |
| TC-22 | ALFQ-HAL-EVA-022 | Verify full ayah clears when the button is clicked again | Main | One ayah marked with "Full ayah" | 1. Click "Full ayah" again on that ayah. | 1. Word highlights on that ayah are removed.<br>2. "Marks recorded" drops by three times the word count for that ayah. | ALFQ-HAL-EVA-021 | `src/features/alforqan/evaluation/evaluation-mistake-helpers.ts` | - |

### Wave 3 — From-end hifz submit (sard gate)

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-23 | ALFQ-HAL-EVA-023 | Verify from-end student can submit memorization evaluation without false sard block | Main | Student `HifzDirection` is "From the end" (2); pending sard checkpoint is after the student's reached position; evaluation range is within allowed boundary (for example Al-Ma'idah 5:120) | 1. Open memorization evaluate page.<br>2. Set range and load ayahs.<br>3. Click "Save draft".<br>4. Click "Submit evaluation". | 1. Draft save succeeds.<br>2. Submit succeeds and opens the outcome modal (no sard gate error dialog with checkpoint label such as "27 جزء").<br>3. Score summary shows the saved range (not cleared to empty). | ALFQ-HAL-EVA-010 | `src/lib/api/tenant-client.ts`, `src/app/[locale]/(tenant)/[tenantSlug]/halaqat/[id]/sessions/[sessionId]/evaluate/[studentId]/halaqa-evaluation-content.tsx` | From-end student |
| TC-31 | ALFQ-HAL-EVA-031 | Verify from-end default memorization range starts at An-Nas | Main | From-end student with no reached surah, or reached An-Nas 114:1 | 1. Open memorization evaluate with no saved draft.<br>2. Open the "From juz" dropdown. | 1. "From surah" shows An-Nas (114), not Al-Fatiha (1).<br>2. "To" is two ayahs ahead in An-Nas (114:3), not Al-Falaq (113).<br>3. Juz options start at 30, not 1.<br>4. "Load ayahs for range" loads text. | ALFQ-HAL-SES-003 | `src/lib/utils/hifz-progress.ts`, `src/app/[locale]/(tenant)/[tenantSlug]/halaqat/[id]/sessions/[sessionId]/evaluate/[studentId]/halaqa-evaluation-content.tsx` | From-end; no reached or 114:1 |

### Wave 2 — Persisted mode and sticky marking chrome

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-24 | ALFQ-HAL-EVA-024 | Verify first visit opens tap mode by default | Main | Teacher has no saved marking mode in browser storage for this account | 1. Clear site data or use a fresh browser profile.<br>2. Open memorization evaluate and load ayahs. | 1. Tab "Tap to flag" / "لمس للتعليم" is selected.<br>2. Hint text matches tap-mode instructions. | ALFQ-HAL-EVA-004 | `src/features/alforqan/evaluation/use-persisted-evaluation-mode.ts`, `src/features/alforqan/evaluation/halaqa-evaluation-marking-section.tsx` | - |
| TC-25 | ALFQ-HAL-EVA-025 | Verify last-used marking mode persists after reload | Main | Ayahs loaded on memorization tab | 1. Switch to "Quick buttons" / "أزرار سريعة".<br>2. Reload the evaluate page.<br>3. Load the same ayah range again. | 1. After reload, "Quick buttons" / "أزرار سريعة" remains selected.<br>2. الحفظ / التجويد / التشكيل buttons appear under the mode tabs. | ALFQ-HAL-EVA-024 | `src/features/alforqan/evaluation/use-persisted-evaluation-mode.ts` | - |
| TC-26 | ALFQ-HAL-EVA-026 | Verify recording block stays visible while scrolling the mushaf | Main | Long range loaded (20+ ayahs); evaluation not submitted | 1. Scroll far down through ayah text.<br>2. Without scrolling back up, switch to "Ayah counters" / "عدادات لكل آية". | 1. Title "Recording method" / "طريقة التسجيل", all three mode tabs, and the active hint remain visible at the top.<br>2. Mode switch succeeds without scrolling back to the top. | ALFQ-HAL-EVA-004 | `src/features/alforqan/evaluation/halaqa-evaluation-marking-section.tsx` | Range with 20+ ayahs |

### Wave 2b — Ayah browse while marking (memorization)

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-27 | ALFQ-HAL-EVA-027 | Verify page size and pagination on long memorization range | Main | Range loaded with 20+ ayahs (e.g. Al-Baqarah 2:1–2:30) | 1. Set "Ayahs per page" to "10".<br>2. Click "Next" to page 2. | 1. At most 10 ayahs show per page.<br>2. Pagination shows page 2 and different ayah numbers than page 1. | ALFQ-HAL-EVA-004 | `src/features/alforqan/evaluation/use-evaluation-ayah-browse.ts`, `src/features/alforqan/evaluation/evaluation-ayah-browse-toolbar.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/halaqat/[id]/sessions/[sessionId]/evaluate/[studentId]/halaqa-evaluation-content.tsx` | 20+ ayahs |
| TC-28 | ALFQ-HAL-EVA-028 | Verify ayah min/max filter and text search | Main | Same long range as TC-27 | 1. Set "From ayah" and "To ayah" to a narrow range within the loaded segment.<br>2. Enter a distinctive substring from an ayah in "Search ayah text".<br>3. Enter text that matches no ayah. | 1. Visible ayahs are limited to the chosen ayah number range.<br>2. Search narrows results to matching ayah text.<br>3. Message "No ayahs match your filters." / "لا توجد آيات تطابق التصفية." appears when nothing matches. | ALFQ-HAL-EVA-027 | `src/features/alforqan/evaluation/evaluation-ayah-browse-utils.ts`, `src/features/alforqan/evaluation/evaluation-ayah-browse-toolbar.tsx` | - |
| TC-29 | ALFQ-HAL-EVA-029 | Verify marks survive ayah browse page change | Main | Long range loaded; at least one word marked on page 1 | 1. Mark a word on page 1.<br>2. Go to page 2 and back to page 1. | 1. The mark on page 1 is still highlighted.<br>2. "Marks recorded" count is unchanged. | ALFQ-HAL-EVA-027 | `src/features/alforqan/evaluation/halaqa-evaluation-marking-section.tsx`, `src/features/alforqan/evaluation/evaluation-mistake-helpers.ts` | - |
| TC-30 | ALFQ-HAL-EVA-030 | Verify browse toolbar sticky only on large screens | Edge | Long range loaded; wide desktop viewport (≥1024px) | 1. On desktop width, scroll far down the mushaf.<br>2. Resize to a narrow phone width and scroll again. | 1. On desktop, browse controls (page size, search, ayah filter) stay visible under the mode tabs while scrolling.<br>2. On narrow width, browse controls scroll away with the mushaf (only the mode block stays sticky). | ALFQ-HAL-EVA-026 | `src/features/alforqan/evaluation/halaqa-evaluation-marking-section.tsx` | Desktop + mobile widths |

### Wave 5 — Autosave and refresh restore

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-32 | ALFQ-HAL-EVA-032 | Verify marks survive refresh without Save draft | Main | Ayahs loaded; evaluation not submitted | 1. Mark at least one word.<br>2. Do not click "Save draft".<br>3. Reload the page. | 1. A banner "Marks restored" / "تم استعادة العلامات" appears, or status shows "All marks saved".<br>2. The same word is still highlighted.<br>3. "Marks recorded" matches the count before reload. | ALFQ-HAL-EVA-007 | `src/lib/evaluation-draft-storage.ts`, `src/features/alforqan/evaluation/use-evaluation-autosave.ts`, `src/app/[locale]/(tenant)/[tenantSlug]/halaqat/[id]/sessions/[sessionId]/evaluate/[studentId]/halaqa-evaluation-content.tsx` | - |
| TC-33 | ALFQ-HAL-EVA-033 | Verify offline marks stay on this device | Edge | Ayahs loaded; browser set offline (DevTools) | 1. Go offline.<br>2. Mark a word.<br>3. Reload while still offline. | 1. Status shows "Offline — marks kept on this device" / "غير متصل — العلامات محفوظة على هذا الجهاز" or the restore banner.<br>2. The mark is still visible after reload. | ALFQ-HAL-EVA-032 | `src/lib/evaluation-draft-storage.ts`, `src/features/alforqan/evaluation/evaluation-save-status-bar.tsx` | Offline |
| TC-34 | ALFQ-HAL-EVA-034 | Verify save status chip while marking | Main | Ayahs loaded | 1. Mark a word.<br>2. Wait about two seconds with network on. | 1. Status shows "Saving…" / "جارٍ الحفظ…" then "All marks saved" / "تم حفظ جميع العلامات".<br>2. No success toast is required. | ALFQ-HAL-EVA-004 | `src/features/alforqan/evaluation/evaluation-save-status-bar.tsx` | - |
| TC-35 | ALFQ-HAL-EVA-035 | Verify submitted evaluation does not restore a local draft | Edge | Evaluation already submitted | 1. Open the submitted evaluate page. | 1. No "Marks restored" banner.<br>2. Marks match the submitted server copy.<br>3. Save draft is hidden. | ALFQ-HAL-EVA-006 | `src/app/[locale]/(tenant)/[tenantSlug]/halaqat/[id]/sessions/[sessionId]/evaluate/[studentId]/halaqa-evaluation-content.tsx` | Submitted evaluation |

### Wave 6 — Outcome attendance

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-36 | ALFQ-HAL-EVA-036 | Verify outcome requires Present, Late, or Absent when the student is unmarked | Main | Student has no saved attendance for this session; outcome modal open | 1. Choose "Failed" without picking attendance.<br>2. Confirm. | 1. Confirm does not save the result.<br>2. Error matches "Mark whether the student was present, late, or absent." / "حدد إن كان الطالب حاضراً أو متأخراً أو غائباً." | ALFQ-HAL-EVA-011 | `src/features/alforqan/evaluation/session-evaluation-outcome-modal.tsx`, `src/features/alforqan/components/sessions/session-attendance-mark-fields.tsx` | Unmarked student |
| TC-37 | ALFQ-HAL-EVA-037 | Verify outcome hides attendance when the student is already marked | Main | Same student already saved on the attendance sheet as "Present"; outcome modal open | 1. Open the outcome modal. | 1. "Attendance for this student" / "حضور هذا الطالب" is not shown.<br>2. Passed/Failed can be saved without picking attendance again. | ALFQ-HAL-ATT-002, ALFQ-HAL-EVA-011 | `src/features/alforqan/evaluation/session-evaluation-outcome-modal.tsx`, `src/features/alforqan/components/sessions/use-session-student-attendance.ts` | Already marked Present |
| TC-38 | ALFQ-HAL-EVA-038 | Verify Late on the outcome modal requires a duration | Main | Student unmarked; outcome modal open | 1. Choose "Late" / "متأخر" under attendance.<br>2. Leave duration empty.<br>3. Choose "Failed" and confirm. | 1. Result is not saved.<br>2. Error matches "Enter how late, from 1 minute up to 24 hours." / "أدخل مدة التأخر من دقيقة واحدة وحتى 24 ساعة." | ALFQ-HAL-EVA-036 | `src/features/alforqan/evaluation/session-evaluation-outcome-modal.tsx`, `src/features/alforqan/components/sessions/late-minutes.ts` | Late, empty duration |

### Wave 7 — Did not memorize (failed)

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-39 | ALFQ-HAL-EVA-039 | Verify did not memorize hides marking and keeps range | Main | Teacher on memorization evaluate; valid range set | 1. Choose "Did not memorize (failed)" / "لم يحفظ (راسب)".<br>2. Confirm in the dialog.<br>3. Save draft. | 1. Message "Marked as failed without memorization for the assigned range." / "مُعلَّم كراسب دون حفظ للنطاق المحدد." appears.<br>2. Ayah marking and "Load ayahs for range" are hidden.<br>3. Range fields stay visible but disabled.<br>4. After reload, the same failed-without-memorization state is restored. | ALFQ-HAL-EVA-001 | `src/app/[locale]/(tenant)/[tenantSlug]/halaqat/[id]/sessions/[sessionId]/evaluate/[studentId]/halaqa-evaluation-content.tsx` | - |
| TC-40 | ALFQ-HAL-EVA-040 | Verify did not memorize submits with zero score and auto-fail outcome | Main | Failed-without-memorization mark active | 1. Submit evaluation.<br>2. If attendance is required, choose "Present" / "حاضر".<br>3. Choose "Send report" / "إرسال التقرير" (no Passed/Retry buttons). | 1. Score shows 0%.<br>2. Session result is "Retry" / "إعادة".<br>3. Student memorization progress is not updated. | ALFQ-HAL-EVA-039 | `src/app/[locale]/(tenant)/[tenantSlug]/halaqat/[id]/sessions/[sessionId]/evaluate/[studentId]/halaqa-evaluation-content.tsx`, `src/features/alforqan/evaluation/session-evaluation-outcome-modal.tsx` | - |
| TC-41 | ALFQ-HAL-EVA-041 | Verify report shows memorization not performed | Main | Outcome recorded for did-not-memorize case | 1. Open the session student report or evaluation report. | 1. Assigned range is shown.<br>2. Text matches "Memorization was not performed." / "لم يُحفظ الطالب." | ALFQ-HAL-EVA-040 | `src/features/alforqan/evaluation/session-evaluation-report-section.tsx` | - |
