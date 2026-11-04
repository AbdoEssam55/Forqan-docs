# Halaqat UI overhaul — Frontend Test Cases


## Metadata
- Project: `alforqan`
- Feature: `halaqat`
- Layer: `frontend`
- Routes: `/{locale}/{tenantSlug}/halaqat`, `/{locale}/{tenantSlug}/halaqat/{id}`, sessions and session report routes
- APIs: -
- Related code: `src/app/[locale]/(tenant)/[tenantSlug]/halaqat/halaqat-list-content.tsx`, `src/components/brand/brand-page-shell.tsx`
- Related docs: `docs/technical/tenant/halaqat-ui-overhaul.md`, `docs/technical/brand-assets.md`

## Scope

- Emerald redesign of halaqat list (grid/list, filters, empty), detail tabs + FAB, sessions timeline, session detail/attendance, session report export styling.
- Does not replace deep functional coverage in existing halaqat session/evaluation specs.

## Test pages

- `/{locale}/{tenantSlug}/halaqat`
- `/{locale}/{tenantSlug}/halaqat/{id}`
- `/{locale}/{tenantSlug}/halaqat/{id}/sessions`
- Session detail and a session student/evaluation report when available

## Delivery waves

| Wave | Intent | Keys | Depends on |
| ---- | ------ | ---- | ---------- |
| 1 | List + detail shell | `ALFQ-HAL-UI-001`–`004` | — |
| 2 | Sessions + reports chrome | `ALFQ-HAL-UI-005`–`006` | — |

## Test cases

### Wave 1 — List and detail

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-01 | ALFQ-HAL-UI-001 | Verify halaqat list supports grid and list views with search and filters | Main | User can open halaqat list | 1. Open the halaqat list.<br>2. Toggle grid and list view controls.<br>3. Type a search term.<br>4. Change type and status filters. | 1. Grid shows cards; list shows a table.<br>2. Search updates the results after a short delay.<br>3. Filters narrow visible rows/cards without a blank broken page. | - | `src/app/[locale]/(tenant)/[tenantSlug]/halaqat/halaqat-list-content.tsx`, `src/features/alforqan/components/halaqa-card.tsx` | Locale: "en" or "ar" |
| TC-02 | ALFQ-HAL-UI-002 | Verify empty halaqat list shows the empty state and create CTA | Main | Tenant has no matching halaqat (or filters yield none) | 1. Open halaqat list with no results (or filter to none).<br>2. Read the empty panel. | 1. An empty title and description are shown.<br>2. A create / "Add Halaqa" button is visible. | - | `src/app/[locale]/(tenant)/[tenantSlug]/halaqat/halaqat-list-content.tsx` | Empty or filtered-empty list |
| TC-03 | ALFQ-HAL-UI-003 | Verify halaqa detail tabs switch Overview, Students, and Sessions | Main | At least one halaqa exists | 1. Open a halaqa detail page.<br>2. Click Overview, Students, and Sessions tabs.<br>3. On Sessions, open a session if any, or use "View all sessions". | 1. Hero shows the halaqa name with emerald styling.<br>2. Each tab shows its content without a full page reload error.<br>3. Sessions tab shows a timeline or empty message. | - | `src/app/[locale]/(tenant)/[tenantSlug]/halaqat/[id]/halaqa-detail-content.tsx` | Existing halaqa |
| TC-04 | ALFQ-HAL-UI-004 | Verify the floating quick-actions button opens useful shortcuts | Main | On halaqa detail | 1. Click the floating "+" button.<br>2. Use Edit (or Sessions / Enroll shortcut). | 1. Shortcut buttons appear.<br>2. Edit opens the edit page (or enroll focuses Students / sessions link works). | - | `src/app/[locale]/(tenant)/[tenantSlug]/halaqat/[id]/halaqa-detail-content.tsx` | Existing halaqa |

### Wave 2 — Sessions and reports chrome

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-05 | ALFQ-HAL-UI-005 | Verify sessions list timeline and create session dialog styling | Main | User can manage sessions for a halaqa | 1. Open halaqa sessions list.<br>2. If permitted, open "Add session" and cancel. | 1. Sessions appear on a vertical timeline (or empty dashed panel).<br>2. Create dialog opens with the schedule form. | - | `src/app/[locale]/(tenant)/[tenantSlug]/halaqat/[id]/sessions/halaqa-sessions-list-content.tsx` | Manage-session claim |
| TC-06 | ALFQ-HAL-UI-006 | Verify session detail attendance save and report export buttons use emerald styling | Main | Open session in progress (or with attendance); optionally open a student/evaluation report | 1. Open a session detail page.<br>2. Confirm attendance area and evaluate list are readable.<br>3. If a report page is available, confirm PDF download button is prominent. | 1. Session page uses emerald header/accent styling.<br>2. Attendance save (when editable) uses the emerald primary button.<br>3. Report download CTA is emerald (not a plain muted outline only). | - | `src/app/[locale]/(tenant)/[tenantSlug]/halaqat/[id]/sessions/[sessionId]/halaqa-session-detail-content.tsx`, `src/features/alforqan/components/sessions/attendance-sheet.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/reports/session/[sessionId]/student/[studentId]/session-student-report-content.tsx` | Session + optional report |
