# Charts & graphs UI overhaul — Frontend Test Cases

## Metadata
- Project: `alforqan`
- Feature: `ui`
- Layer: `frontend`
- Routes: `/{locale}/{tenantSlug}/reports/memorization`, `/{locale}/{tenantSlug}/reports/ranking`, `/{locale}/{tenantSlug}/dashboard`
- Related code: `src/lib/brand/chart.ts`, `src/components/ui/chart.tsx`, `src/features/alforqan/reports/report-stat-bar.tsx`
- Related docs: `docs/technical/tenant/charts-ui-overhaul.md`

## Scope

- Phase 11.3 chart palette, bar tooltips, split-bar legend, responsive inline bars on ranking/dashboard.

## Delivery waves

| Wave | Intent | Keys |
| ---- | ------ | ---- |
| 1 | Report bars + tooltip/legend | `ALFQ-CHART-UI-001`–`002` |
| 2 | Ranking + dashboard inline bars | `ALFQ-CHART-UI-003`–`004` |

## Test cases

### Wave 1 — Report bars

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-01 | ALFQ-CHART-UI-001 | Verify memorization summary bars show value and hover tooltip | Main | User can open memorization reports with stats | 1. Open `/{tenantSlug}/reports/memorization`.<br>2. Find the summary progress bars.<br>3. Hover a bar track. | 1. Each bar shows a label and numeric value.<br>2. The fill uses an emerald-to-teal gradient on a muted track.<br>3. Hover shows a tooltip with the label and value. | - | `src/components/ui/chart.tsx`, `src/features/alforqan/reports/report-stat-bar.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/reports/memorization/memorization-reports-list-content.tsx` | Reports with scores |
| TC-02 | ALFQ-CHART-UI-002 | Verify split bar shows legend under the track | Main | Memorization list shows pass/fail or split counts | 1. On memorization reports, find a two-tone split bar.<br>2. Look under the bar. | 1. The bar has two colored segments.<br>2. A legend row under the bar shows a swatch and label for each side. | ALFQ-CHART-UI-001 | `src/components/ui/chart.tsx`, `src/features/alforqan/reports/report-stat-bar.tsx` | Split counts present |

### Wave 2 — Inline surfaces

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-03 | ALFQ-CHART-UI-003 | Verify ranking progress column uses chart inline bars | Main | Ranking report has rows | 1. Open `/{tenantSlug}/reports/ranking`.<br>2. Look at the progress column. | 1. Each row shows a compact horizontal bar.<br>2. Bar width matches relative score (higher score → wider fill). | ALFQ-CHART-UI-001 | `src/components/ui/chart.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/reports/ranking/ranking-content.tsx` | Ranking data |
| TC-04 | ALFQ-CHART-UI-004 | Verify dashboard role counts use the same bar chrome | Main | Admin dashboard with users-by-role | 1. Open `/{tenantSlug}/dashboard` as admin.<br>2. Find the by-role bars. | 1. Each role row has a muted track and emerald gradient fill.<br>2. Fill width scales with the role count. | ALFQ-CHART-UI-001 | `src/components/ui/chart.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/dashboard-content.tsx` | Dashboard stats |
