# Parents UI overhaul — Frontend Test Cases

## Metadata
- Project: `alforqan`
- Feature: `parents`
- Layer: `frontend`
- Routes: `/{locale}/{tenantSlug}`, `/{locale}/{tenantSlug}/parents`, `/{locale}/{tenantSlug}/parents/{id}`
- APIs: -
- Related code: `src/app/[locale]/(tenant)/[tenantSlug]/dashboard-content.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/parents/parents-list-content.tsx`
- Related docs: `docs/technical/tenant/parents-ui-overhaul.md`, `docs/technical/tenant/parent-dashboard-overhaul.md`

## Scope

- Phase 7.1 + 7.2 emerald / BrandPageShell redesign for parents list, profile, and parent dashboard chrome.
- Does not replace deep parent registration/onboarding functional specs.

## Delivery waves

| Wave | Intent | Keys |
| ---- | ------ | ---- |
| 1 | List cards/filters/bulk | `ALFQ-PRN-UI-001`–`003` |
| 2 | Profile hero/children/contact | `ALFQ-PRN-UI-004`–`006` |
| 3 | Parent dashboard cards/feed | `ALFQ-PRN-UI-007`–`009` |

## Test cases

### Wave 1 — List

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-01 | ALFQ-PRN-UI-001 | Verify parents list grid/list toggle, search, and status filter | Main | User can open parents list | 1. Open parents list.<br>2. Toggle grid and list.<br>3. Search and change status filter. | 1. Grid shows parent cards with avatar initials.<br>2. List shows a table with parent rows.<br>3. Search and filter update visible parents without a crash. | - | `src/app/[locale]/(tenant)/[tenantSlug]/parents/parents-list-content.tsx`, `src/features/alforqan/components/parent-card.tsx` | Locale en/ar |
| TC-02 | ALFQ-PRN-UI-002 | Verify empty parents state and create CTA | Main | No matching parents | 1. Filter or use a tenant with no parents.<br>2. Read empty panel. | 1. Empty title and description are shown.<br>2. "Add Parent" CTA appears when create claim is granted. | - | `src/app/[locale]/(tenant)/[tenantSlug]/parents/parents-list-content.tsx` | Empty list |
| TC-03 | ALFQ-PRN-UI-003 | Verify bulk select and deactivate toolbar | Main | User has manage-user claim; at least one active linked parent | 1. Select one or more parents (grid checkbox or list).<br>2. Use "Deactivate selected" and confirm. | 1. Bulk toolbar shows selected count.<br>2. Confirmed deactivate succeeds and list refreshes. | - | `src/app/[locale]/(tenant)/[tenantSlug]/parents/parents-list-content.tsx`, `src/features/alforqan/components/parent-card.tsx` | Active parents with `userId` |

### Wave 2 — Profile

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-04 | ALFQ-PRN-UI-004 | Verify parent profile hero and information card | Main | Parent exists | 1. Open parent detail.<br>2. Check hero and parent information card. | 1. Hero shows parent initials, name, phone, and status badge.<br>2. Parent information card shows full name, nationality, and active status. | - | `src/app/[locale]/(tenant)/[tenantSlug]/parents/[id]/parent-detail-content.tsx` | Existing parent |
| TC-05 | ALFQ-PRN-UI-005 | Verify children linkage display with student deep links | Main | Parent has one or more linked students | 1. Open parent detail.<br>2. Open linked children section.<br>3. Click a student link. | 1. Linked children cards show student name and student code.<br>2. Relation type is shown when available.<br>3. Clicking a student opens that student detail page. | - | `src/app/[locale]/(tenant)/[tenantSlug]/parents/[id]/parent-detail-content.tsx` | Parent with linked students |
| TC-06 | ALFQ-PRN-UI-006 | Verify contact section and account tabs still work on redesigned profile | Main | Parent has linked user; tester has roles/claims permissions | 1. Open parent detail.<br>2. Check contact info section.<br>3. Open roles/permissions and password tabs. | 1. Contact section shows phone, secondary phone, and address values.<br>2. Roles/permissions tab opens existing user claims UI.<br>3. Password tab opens change password form without errors. | - | `src/app/[locale]/(tenant)/[tenantSlug]/parents/[id]/parent-detail-content.tsx` | Linked parent user |

### Wave 3 — Parent dashboard

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-07 | ALFQ-PRN-UI-007 | Verify parent dashboard children progress cards render with report data | Main | Parent account with one or more linked children and reports | 1. Sign in as parent user.<br>2. Open tenant dashboard `/{tenantSlug}`.<br>3. Review children progress section. | 1. Children cards are shown for linked children.<br>2. Each card shows average score and reports count values.<br>3. Clicking a child card opens reports page filtered for that child. | ALFQ-PAR-REG-001 | `src/features/alforqan/components/parent-dashboard-overview.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/dashboard-content.tsx` | Parent with linked students |
| TC-08 | ALFQ-PRN-UI-008 | Verify parent dashboard recent reports section shows latest report cards | Main | Parent account with family reports | 1. Sign in as parent user.<br>2. Open tenant dashboard.<br>3. Review recent reports section and open one card. | 1. Recent reports section shows latest session/sard cards.<br>2. Each card displays child context and report type badges.<br>3. Opening a card navigates to the matching report detail page. | ALFQ-PRN-UI-007 | `src/features/alforqan/components/parent-dashboard-overview.tsx`, `src/features/alforqan/reports/family-report-card.tsx` | Existing family reports |
| TC-09 | ALFQ-PRN-UI-009 | Verify parent dashboard notifications feed shows unread status and deep links | Main | Parent account with at least one notification | 1. Sign in as parent user.<br>2. Open tenant dashboard.<br>3. Click a notification in the feed. | 1. Notifications list shows latest items with unread highlight and unread count badge.<br>2. Clicking a notification marks it as read when possible.<br>3. Notification deep-links to the matching report/session page when a target exists. | ALFQ-PRN-UI-007 | `src/features/alforqan/components/parent-dashboard-overview.tsx`, `src/features/alforqan/components/notifications/tenant-notification-target.ts`, `src/features/alforqan/components/notifications/tenant-notification-bell.tsx` | Notification data |
