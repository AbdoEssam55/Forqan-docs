# Tables UI overhaul — Frontend Test Cases

## Metadata
- Project: `alforqan`
- Feature: `ui`
- Layer: `frontend`
- Routes: `/{locale}/{tenantSlug}/students`, `/{locale}/{tenantSlug}/teachers`, `/{locale}/{tenantSlug}/parents`, `/{locale}/{tenantSlug}/halaqat`
- Related code: `src/lib/brand/table.ts`, `src/components/ui/data-table.tsx`, `src/components/ui/table.tsx`, `src/components/ui/table-pagination.tsx`
- Related docs: `docs/technical/tenant/tables-ui-overhaul.md`

## Scope

- Phase 11.1 DataTable chrome: header, hover, sort, empty, loading bar, pagination, row actions.
- List pages that already use `DataTable` pick up chrome automatically.

## Delivery waves

| Wave | Intent | Keys |
| ---- | ------ | ---- |
| 1 | Header + row chrome | `ALFQ-TBL-UI-001`–`002` |
| 2 | Sort + actions | `ALFQ-TBL-UI-003`–`004` |
| 3 | Empty + pagination | `ALFQ-TBL-UI-005`–`006` |

## Test cases

### Wave 1 — Header + row chrome

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-01 | ALFQ-TBL-UI-001 | Verify students list table has emerald header band | Main | User can view students; list has rows | 1. Open `/{tenantSlug}/students` in list/table view.<br>2. Look at the column header row. | 1. Header row uses a light emerald/green tinted band (not plain gray only).<br>2. Header labels look uppercase / semi-bold compared to body cells. | - | `src/components/ui/data-table.tsx`, `src/lib/brand/table.ts`, `src/app/[locale]/(tenant)/[tenantSlug]/students/students-list-content.tsx` | Students exist |
| TC-02 | ALFQ-TBL-UI-002 | Verify row hover uses emerald tint | Main | Students list with at least one row | 1. Move the pointer over a data row (not the header). | 1. The row background soft-highlights with an emerald/teal tint.<br>2. The highlight clears when the pointer leaves. | ALFQ-TBL-UI-001 | `src/components/ui/table.tsx`, `src/lib/brand/table.ts` | - |

### Wave 2 — Sort + actions

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-03 | ALFQ-TBL-UI-003 | Verify sortable column toggles sort direction | Main | Students list; a sortable column (e. and name) | 1. Click a sortable column header once.<br>2. Click the same header again. | 1. First click shows an up or down sort arrow on that column.<br>2. Second click flips the arrow direction.<br>3. Row order changes with the sort. | ALFQ-TBL-UI-001 | `src/components/ui/data-table.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/students/students-list-content.tsx` | Multiple students |
| TC-04 | ALFQ-TBL-UI-004 | Verify row actions menu does not navigate the row | Main | Students list with row actions | 1. Open the actions control on a row.<br>2. Do not choose an item yet; close the menu. | 1. Actions menu opens.<br>2. Opening the menu does not navigate to the student profile. | ALFQ-TBL-UI-001 | `src/components/ui/data-table.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/students/students-list-content.tsx` | - |

### Wave 3 — Empty + pagination

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-05 | ALFQ-TBL-UI-005 | Verify empty table shows icon and hint | Main | Search/filter that matches no students | 1. Enter a search that returns zero students.<br>2. Read the table body. | 1. An empty illustration/icon appears in the table.<br>2. Empty title (e.g. "No students found") is visible.<br>3. A muted hint about filters/search is visible. | ALFQ-TBL-UI-001 | `src/components/ui/data-table.tsx` | No matching students |
| TC-06 | ALFQ-TBL-UI-006 | Verify pagination controls keep branded page buttons | Main | Enough students for 2+ pages | 1. Scroll to the pagination under the table.<br>2. Go to page 2. | 1. Current page button uses primary/gradient fill.<br>2. Other page buttons use outline chrome.<br>3. Page 2 loads a different set of rows. | ALFQ-TBL-UI-001 | `src/components/ui/table-pagination.tsx`, `src/components/ui/data-table.tsx` | 11+ students |
