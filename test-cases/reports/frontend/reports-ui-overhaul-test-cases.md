# Reports UI overhaul — Frontend Test Cases

## Metadata
- Project: `alforqan`
- Feature: `reports`
- Layer: `frontend`
- Routes: `/{locale}/{tenantSlug}/reports`, `/{locale}/{tenantSlug}/reports/memorization`, `/{locale}/{tenantSlug}/reports/sessions`, `/{locale}/{tenantSlug}/reports/ranking`
- Related code: `src/app/[locale]/(tenant)/[tenantSlug]/reports/reports-hub-content.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/reports/memorization/memorization-reports-list-content.tsx`, `src/features/alforqan/reports/staff-session-reports-page.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/reports/ranking/ranking-content.tsx`
- Related docs: `docs/technical/tenant/reports-ui-overhaul.md`, `docs/technical/tenant/staff-session-reports-guide.md`, `docs/technical/tenant/reports-memorization-guide.md`, `docs/technical/tenant/dashboard-and-ranking-guide.md`

## Scope

- Phase 8 emerald / BrandPageShell redesign for reports hub, memorization CRUD list, session attendance calendar, and ranking leaderboard.
- Does not replace staff session filter/tab functional specs (`ALFQ-SSR-UI-*`).

## Delivery waves

| Wave | Intent | Keys |
| ---- | ------ | ---- |
| 1 | Memorization filters/charts/export | `ALFQ-REP-UI-001`–`003` |
| 2 | Sessions KPI charts + attendance calendar | `ALFQ-REP-UI-004`–`006` |
| 3 | Ranking medals/progress/period | `ALFQ-REP-UI-007`–`009` |
| 4 | Reports hub chrome | `ALFQ-REP-UI-010` |

## Test cases

### Wave 1 — Memorization list

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-01 | ALFQ-REP-UI-001 | Verify memorization list filters student teacher type and dates | Main | Staff can open memorization reports; existing reports | 1. Open `/reports/memorization`.<br>2. Click "Show filters" / "إظهار التصفية".<br>3. Set student, teacher, type, and date range. | 1. Filter controls are hidden until "Show filters" is clicked.<br>2. Table rows match the selected student, teacher, type, and dates.<br>3. Empty filters show the unfiltered list again after "Clear filters" / "مسح التصفية". | - | `src/app/[locale]/(tenant)/[tenantSlug]/reports/memorization/memorization-reports-list-content.tsx` | Existing memorization reports |
| TC-02 | ALFQ-REP-UI-002 | Verify memorization score bars and type charts on the current page | Main | Memorization list has mixed memorization and revision scores | 1. Open `/reports/memorization` with at least two scored rows.<br>2. Read the chart strip and score column. | 1. Average score bar is visible.<br>2. Memorization vs revision split counts match the current page.<br>3. Each score cell shows a progress bar. | ALFQ-REP-UI-001 | `src/app/[locale]/(tenant)/[tenantSlug]/reports/memorization/memorization-reports-list-content.tsx`, `src/features/alforqan/reports/report-stat-bar.tsx` | Mixed scores |
| TC-03 | ALFQ-REP-UI-003 | Verify memorization CSV export of the current page | Main | Memorization list has at least one row | 1. Open `/reports/memorization`.<br>2. Click "Export CSV" / "تصدير CSV". | 1. Browser downloads a `.csv` file.<br>2. File includes student, teacher, date, type, and score columns for the visible page. | ALFQ-REP-UI-001 | `src/app/[locale]/(tenant)/[tenantSlug]/reports/memorization/memorization-reports-list-content.tsx` | Exportable rows |

### Wave 2 — Session attendance

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-04 | ALFQ-REP-UI-004 | Verify session reports KPI bars and emerald export button | Main | Staff on `/reports/sessions` with hifz rows this month and `ExportReportsClaim` | 1. Open `/reports/sessions`.<br>2. Read KPI strip and Export. | 1. KPI shows total, average, and passed vs absent as bars (not plain number-only cards only).<br>2. Export button is emerald-styled.<br>3. Clicking Export opens the export dialog. | ALFQ-SSR-UI-002 | `src/features/alforqan/reports/staff-session-reports-page.tsx`, `src/features/alforqan/reports/report-stat-bar.tsx` | Hifz data |
| TC-05 | ALFQ-REP-UI-005 | Verify attendance calendar view lists students for a selected day | Main | Staff with hifz attendance this month | 1. Open `/reports/sessions` on Hifz.<br>2. Click "Calendar" / "تقويم".<br>3. Click a day that has sessions. | 1. URL includes `view=calendar`.<br>2. Calendar cells show attendance dots.<br>3. Selected day lists student names with Present/Absent/Late/Excused labels.<br>4. Clicking a student still opens the session report. | ALFQ-REP-UI-004 | `src/features/alforqan/reports/staff-session-reports-page.tsx`, `src/features/alforqan/reports/staff-attendance-calendar.tsx` | Attendance this month |
| TC-06 | ALFQ-REP-UI-006 | Verify sard tab hides calendar toggle and stays on the list | Main | Staff with sard tab | 1. Open `/reports/sessions`.<br>2. Switch to Sard. | 1. List / Calendar toggle is not shown.<br>2. Sard DataTable remains visible. | ALFQ-SSR-UI-023 | `src/features/alforqan/reports/staff-session-reports-page.tsx` | Sard data |

### Wave 3 — Ranking

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-07 | ALFQ-REP-UI-007 | Verify ranking medals and progress bars for top three | Main | Ranking data with at least three students on page 1 | 1. Open `/reports/ranking`.<br>2. Confirm results are on page 1. | 1. Rank 1 shows a trophy icon on the podium and in the table.<br>2. Ranks 2 and 3 show medal/award icons.<br>3. Each table row shows average % with a progress bar. | - | `src/app/[locale]/(tenant)/[tenantSlug]/reports/ranking/ranking-content.tsx` | Ranking rows |
| TC-08 | ALFQ-REP-UI-008 | Verify ranking period preset is written to the URL and reloads one board | Main | Ranking data exists across dates | 1. Open `/reports/ranking`.<br>2. Expand filters with "Show filters" / "إظهار التصفية".<br>3. Choose "This week" / "هذا الأسبوع". | 1. The URL contains `preset=week`.<br>2. One ranking board reloads (not two stacked cards).<br>3. Titles "Halaqa Ranking" / "ترتيب الحلقة" and "Global Ranking" / "الترتيب العام" do not appear. | ALFQ-REP-UI-007 | `src/app/[locale]/(tenant)/[tenantSlug]/reports/ranking/ranking-content.tsx` | Dated ranking |
| TC-09 | ALFQ-REP-UI-009 | Verify ranking table headers are localized with no hardcoded English | Main | Arabic locale | 1. Open `/reports/ranking` in `ar`.<br>2. Expand filters if the table is visible. | 1. Headers show Arabic labels for rank, student, halaqa, average, and slots ("الترتيب", "الطالب", "الحلقة", "المتوسط", "الخانات").<br>2. Type tabs include "الكل", "حفظ", "مراجعة", "سرد".<br>3. Filter show label is "إظهار التصفية", not "Show filters". | ALFQ-REP-UI-007 | `src/app/[locale]/(tenant)/[tenantSlug]/reports/ranking/ranking-content.tsx` | Locale ar |

### Wave 4 — Hub

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-10 | ALFQ-REP-UI-010 | Verify staff reports hub tiles link to sessions sard teacher and ranking only | Main | Staff reports hub user | 1. Open `/{tenantSlug}/reports`. | 1. Three staff tiles appear in this order: Session reports, Sard teacher reports, Ranking.<br>2. First tile goes to `/reports/sessions`.<br>3. Ranking tile goes to `/reports/ranking`.<br>4. Memorization and narration (khatm) tiles are absent. | ALFQ-SSR-UI-001 | `src/app/[locale]/(tenant)/[tenantSlug]/reports/reports-hub-content.tsx` | Staff account |
