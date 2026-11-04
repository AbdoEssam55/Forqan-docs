# Feedback & status UI overhaul — Frontend Test Cases

## Metadata
- Project: `alforqan`
- Feature: `ui`
- Layer: `frontend`
- Routes: `/{locale}/{tenantSlug}/students`, `/{locale}/{tenantSlug}/teachers`, `/{locale}/{tenantSlug}/halaqat`
- Related code: `src/components/ui/sonner.tsx`, `src/features/alforqan/components/notifications/tenant-notification-bell.tsx`, `src/components/ui/empty-state.tsx`
- Related docs: `docs/technical/tenant/feedback-status-ui-overhaul.md`

## Scope

- Phase 12 toast chrome, notification center, badges, alert banners, shimmer/spinner/progress, shared empty states.

## Delivery waves

| Wave | Intent | Keys |
| ---- | ------ | ---- |
| 1 | Toasts + notification center | `ALFQ-FEED-UI-001`–`003` |
| 2 | Loading chrome | `ALFQ-FEED-UI-004`–`005` |
| 3 | Empty states | `ALFQ-FEED-UI-006`–`007` |

## Test cases

### Wave 1 — Notifications

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-01 | ALFQ-FEED-UI-001 | Verify success toast uses emerald chrome and a close control | Main | User can deactivate a student | 1. Open `/{tenantSlug}/students`.<br>2. Deactivate one student and confirm. | 1. A success toast appears (not a browser alert).<br>2. The toast uses an emerald/green tinted background or border.<br>3. A close control is visible on the toast. | - | `src/components/ui/sonner.tsx`, `src/providers/toast-provider.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/students/students-list-content.tsx` | At least one student |
| TC-02 | ALFQ-FEED-UI-002 | Verify the notification bell shows an unread count badge | Main | Tenant user with at least one unread notification | 1. Open any tenant page with the header bell.<br>2. Look at the bell button. | 1. A round emerald count badge sits on the outer corner of the bell circle.<br>2. The full badge and all count digits are visible (not cut off by the circular button).<br>3. The badge shows the unread count (or "99+" if over 99). | - | `src/features/alforqan/components/notifications/tenant-notification-bell.tsx`, `src/components/ui/badge.tsx` | Unread notifications |
| TC-03 | ALFQ-FEED-UI-003 | Verify the notification panel shows an unread banner and list chrome | Main | Same as TC-02 | 1. Click the bell.<br>2. Read the slide-over panel. | 1. An info alert banner shows the unread count.<br>2. Unread rows have a stronger emerald highlight than read rows.<br>3. "Mark all read" is enabled. | ALFQ-FEED-UI-002 | `src/features/alforqan/components/notifications/tenant-notification-bell.tsx`, `src/components/brand/alert-banner.tsx` | Unread notifications |

### Wave 2 — Loading

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-04 | ALFQ-FEED-UI-004 | Verify students grid loading uses shimmer skeletons | Main | User can view students | 1. Open `/{tenantSlug}/students`.<br>2. Hard-refresh (or throttle network) so the grid loading state is visible. | 1. Card-shaped skeleton blocks appear before the list.<br>2. The blocks show a moving light shimmer, not a static gray box only. | - | `src/components/ui/skeleton.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/students/students-list-content.tsx`, `src/styles/globals.css` | - |
| TC-05 | ALFQ-FEED-UI-005 | Verify a thin emerald progress bar appears while loading | Main | User can open the notification panel or a students list | 1. Open the notification bell while the list is still loading, or switch student list pages with a slow network.<br>2. Watch the top of the panel/table. | 1. A thin emerald/teal bar animates while data is loading.<br>2. The notification panel shows a spinner with a loading label on first load. | ALFQ-FEED-UI-002 | `src/components/ui/progress.tsx`, `src/components/feedback/loading-spinner.tsx`, `src/features/alforqan/components/notifications/tenant-notification-bell.tsx`, `src/components/ui/data-table.tsx` | - |

### Wave 3 — Empty states

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-06 | ALFQ-FEED-UI-006 | Verify the students grid empty state shows icon, message, and create CTA | Main | User can create students; list is empty (or search matches none in grid view) | 1. Open `/{tenantSlug}/students` in grid view with no matching students.<br>2. Read the empty card. | 1. A dashed card shows a large icon in an emerald square.<br>2. An empty title and description are visible.<br>3. A create CTA button is visible if the user can create students. | - | `src/components/ui/empty-state.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/students/students-list-content.tsx` | Empty or filtered-empty list |
| TC-07 | ALFQ-FEED-UI-007 | Verify table and notification empty states use the shared empty layout | Main | User can open students list view and the notification panel | 1. Switch students to list/table view and search so zero rows match.<br>2. Open the notification bell with no notifications. | 1. The table empty cell shows a circular icon, title, and hint (not a blank table).<br>2. The notification panel empty state shows a bell icon, title, and description. | ALFQ-FEED-UI-006 | `src/components/ui/empty-state.tsx`, `src/components/ui/data-table.tsx`, `src/features/alforqan/components/notifications/tenant-notification-bell.tsx` | Zero matching students; empty inbox |
