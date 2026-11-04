# Tenant app shell — Frontend Test Cases


## Metadata
- Project: `alforqan`
- Feature: `tenant-shell`
- Layer: `frontend`
- Routes: `Any signed-in tenant route, for example `/{locale}/{tenantSlug}/halaqat` (replace with a real tenant slug).`
- APIs: -
- **Related code:** `src/components/layout/tenant-top-chrome.tsx`, `src/features/alforqan/components/tenant-navigation.tsx`, `src/components/brand/brand-logo.tsx`, `src/components/brand/brand-page-shell.tsx`
- **Related docs:** `docs/technical/tenant/navigation-layout-overhaul.md`, `docs/technical/brand-assets.md`

## Scope

- Signed-in tenant experience: language control, notification bell, and account avatar clustered at inline-end (away from the sidebar brand/header), plus the mobile navigation opener on narrow viewports.
- Collapsible sidebar groups, breadcrumbs, and dashboard stat grid / role bars.
- Arabic (right-to-left) and English layouts for the top controls.
- A way back to the public landing page (`/{locale}`) from any tenant shell page without signing out.
- Below `lg`, a hamburger in the top bar opens the nav sheet (sidebar is hidden).

## Test pages

- Any signed-in tenant route, for example `/{locale}/{tenantSlug}/halaqat` (replace with a real tenant slug).
- Tenant dashboard: `/{locale}/{tenantSlug}` (or dashboard home for the institute).

## Test scenarios

- On desktop, the language control sits next to the notification bell and account menu and does not cover the sidebar brand/header label.
- On a narrow screen in Arabic or English, the language + bell + avatar cluster and the menu opener are all visible and tappable without covering each other.
- Switching locale from the clustered language control still works.
- Sidebar groups start collapsed (except the group for the current page); collapse/expand still works; active link uses emerald styling.
- Desktop breadcrumbs update with the current path.
- Dashboard shows a responsive stat grid and (for admin/supervisor) role breakdown bars.

## Delivery waves

| Wave | Intent | Keys (representative) | Depends on (cross-file) |
| ---- | ------ | --------------------- | ----------------------- |
| 1 | Top bar layout (language next to bell; menu clearance) | `ALFQ-TEN-SHL-001`–`004` | — |
| 2 | Nav groups, breadcrumbs, account menu, dashboard layout | `ALFQ-TEN-SHL-005`–`009`, `ALFQ-TEN-SHL-012` | — |
| 3 | Return to public landing from any tenant page | `ALFQ-TEN-SHL-010`–`011` | — |
| 4 | Mobile/medium nav opener in top chrome | `ALFQ-TEN-SHL-015` | Wave 1 |

Independent of halaqat delivery order. Bell cases complement [notifications](../../halaqat/frontend/halaqat-notifications-test-cases.md) Wave 1.

## Test cases

### Wave 1 — Top bar (language next to bell)

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-01 | ALFQ-TEN-SHL-001 | Verify the language control sits next to the notification bell and does not cover the sidebar brand on desktop | Main | User is signed in to a tenant; browser window is wide enough that the sidebar is shown | 1. Open a tenant page such as the halaqat list in Arabic.<br>2. Look at the top of the page: sidebar brand/header, language control, and circular bell control. | 1. The sidebar brand/header label (for example "Alforqan") is fully readable and not covered by the language control.<br>2. The language control and the notification bell sit next to each other on the opposite side from the sidebar (inline-end), with the account avatar nearby.<br>3. The language control shows "العربية" (or the English label when the site is in English). | - | `src/components/layout/tenant-top-chrome.tsx`, `src/components/layout/language-switcher.tsx` | Locale: "ar" |
| TC-02 | ALFQ-TEN-SHL-002 | Verify the language and bell cluster stays usable in English on a narrow screen | Main | User is signed in to a tenant; site language is English; browser window is narrow enough that the sidebar is not shown | 1. Open the same kind of tenant page.<br>2. Look at the top bar: language control, circular bell, account avatar, and square navigation opener. | 1. The language control is fully visible (label may show as "English").<br>2. The language control sits next to the notification bell and account avatar at inline-end.<br>3. The square navigation opener is inside the top bar at inline-start (not hidden behind the bar).<br>4. Tapping the navigation opener opens the institute navigation side panel. | - | `src/components/layout/tenant-top-chrome.tsx`, `src/features/alforqan/components/tenant-navigation.tsx` | Locale: "en" |
| TC-03 | ALFQ-TEN-SHL-003 | Verify the language and bell cluster does not cover the menu opener on a narrow screen | Regression | User is signed in to a tenant; browser window is narrow enough that the sidebar is not shown | 1. Open a tenant page such as the halaqat list in Arabic or English.<br>2. Find the language control, the circular outline control with a bell icon, the account avatar, and the square outline navigation opener in the top bar. | 1. All controls are fully visible; the language / bell / avatar cluster does not sit on top of the navigation opener.<br>2. The navigation opener is in the top bar (inline-start), not a floating button hidden under the bar.<br>3. Tapping the navigation opener opens the institute navigation side panel.<br>4. Tapping the bell opens the notifications side panel. | ALFQ-TEN-SHL-001 | `src/components/layout/tenant-top-chrome.tsx`, `src/features/alforqan/components/notifications/tenant-notification-bell.tsx`, `src/features/alforqan/components/tenant-navigation.tsx` | Locale: "ar" or "en" |
| TC-04 | ALFQ-TEN-SHL-004 | Verify switching language from the clustered control updates the page locale | Main | User is signed in to a tenant; language control is visible next to the bell | 1. Open a tenant page.<br>2. Open the language control next to the notification bell.<br>3. Choose the other language (English if Arabic, or "العربية" if English). | 1. The page reloads or navigates into the other locale.<br>2. The language control label updates to the new language.<br>3. The control remains next to the notification bell (not over the sidebar brand). | ALFQ-TEN-SHL-001 | `src/components/layout/language-switcher.tsx`, `src/components/layout/tenant-top-chrome.tsx` | Locale: "ar" then "en" (or reverse) |

### Wave 2 — Navigation groups, breadcrumbs, account menu, dashboard

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-05 | ALFQ-TEN-SHL-005 | Verify sidebar groups collapse and expand and show an emerald active link | Main | User is signed in to a tenant; browser is wide enough to show the sidebar; user can see at least one collapsible group (for example People) | 1. Open the tenant dashboard home.<br>2. Find a collapsible group header such as "People" / "الأشخاص".<br>3. Click the group header to expand, then click again to collapse.<br>4. Expand "Learning" / "التعلم" and click a nav item (for example "Halaqat"). | 1. On the dashboard, People / Learning / Finance / System show headers only (child links hidden).<br>2. Expanding shows the group's links; collapsing hides them again.<br>3. The active link uses emerald highlight styling (not a plain gray ghost button only).<br>4. The current page's nav item stays highlighted after navigation. | - | `src/features/alforqan/components/tenant-navigation.tsx`, `src/features/alforqan/config/tenant-nav.ts` | Locale: "en" or "ar" |
| TC-06 | ALFQ-TEN-SHL-006 | Verify desktop breadcrumbs reflect the current tenant path | Main | User is signed in; browser is wide enough that breadcrumbs are visible (md+) | 1. Open the tenant dashboard home.<br>2. Navigate to Halaqat (or Students).<br>3. Read the breadcrumb trail in the top bar. | 1. On home, a single "Dashboard" / "لوحة التحكم" crumb is shown.<br>2. On a deeper page, crumbs show Dashboard then the section name (for example "Halaqat").<br>3. Clicking an earlier crumb navigates back without a permission error. | - | `src/components/layout/tenant-breadcrumbs.tsx`, `src/components/layout/tenant-top-chrome.tsx` | Locale: "en" |
| TC-07 | ALFQ-TEN-SHL-007 | Verify the account avatar menu opens settings and sign out | Main | User is signed in to a tenant | 1. Click the circular account avatar (or name chip) in the top bar.<br>2. Choose "Settings" / "الإعدادات".<br>3. Open the menu again and choose "Sign Out" / "تسجيل الخروج". | 1. The menu shows the account name, "Back to website" / "العودة إلى الموقع", and links for settings and profile.<br>2. Settings opens the tenant settings page.<br>3. Sign out clears the session and lands on the tenant login (or public sign-in) page. | - | `src/components/layout/tenant-user-menu.tsx` | Locale: "en" or "ar" |
| TC-08 | ALFQ-TEN-SHL-008 | Verify the dashboard shows a responsive stat grid and role bars for admins | Main | User is signed in as admin or supervisor with dashboard access | 1. Open the tenant dashboard home.<br>2. Look at the stat cards grid.<br>3. Look for the "By role" / role breakdown section. | 1. Stat cards appear in a responsive grid (icons + counts) and are clickable where linked.<br>2. For admin/supervisor, a "By role" section shows horizontal emerald bars with counts.<br>3. Cards and bars remain readable on a narrow viewport without overlapping the top chrome. | - | `src/app/[locale]/(tenant)/[tenantSlug]/dashboard-content.tsx`, `src/components/ui/stat-card.tsx`, `src/components/brand/brand-page-shell.tsx` | Admin or supervisor account |
| TC-09 | ALFQ-TEN-SHL-009 | Verify the sidebar shows the Alforqan logo when the tenant has no custom logo | Main | User is signed in; tenant branding has no logo URL (or logo is empty) | 1. Open any signed-in tenant page with the desktop sidebar visible.<br>2. Look at the emerald sidebar header. | 1. The Alforqan logo image (`logo1.png`) shows in the sidebar header (not the letters "AQ").<br>2. The tenant name still appears next to the logo. | - | `src/features/alforqan/components/tenant-navigation.tsx`, `src/components/brand/brand-logo.tsx`, `src/lib/brand/assets.ts` | Locale: "en" or "ar" |
| TC-10 | ALFQ-TEN-SHL-012 | Verify collapsible sidebar groups start collapsed except the current page group | Main | User is signed in to a tenant; browser is wide enough to show the sidebar | 1. Open the tenant dashboard home (`/{locale}/{tenantSlug}`).<br>2. Look at People / Learning / Finance / System headers.<br>3. Open a nested page such as Halaqat.<br>4. Look at the sidebar groups again. | 1. On the dashboard, Dashboard stays visible.<br>2. People, Learning, Finance, and System show headers only (no child links).<br>3. On Halaqat, the "Learning" / "التعلم" group is open and "Halaqat" is highlighted.<br>4. People, Finance, and System stay collapsed until clicked. | ALFQ-TEN-SHL-005 | `src/features/alforqan/components/tenant-navigation.tsx`, `src/features/alforqan/config/tenant-nav.ts` | Locale: "en" or "ar" |

### Wave 3 — Back to public landing

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-11 | ALFQ-TEN-SHL-010 | Verify the sidebar "Back to website" link opens the public landing page | Main | User is signed in to a tenant (for example `/ar/alforqan`) | 1. Open the tenant dashboard or any other tenant page (for example Halaqat).<br>2. In the sidebar footer (or the mobile nav sheet), find "Back to website" / "العودة إلى الموقع".<br>3. Click that link. | 1. The control is visible on every tenant page that uses the shell, not only the dashboard.<br>2. The browser opens the locale landing page (`/ar` or `/en`), not the tenant dashboard and not login.<br>3. The landing hero (logo, title, "Get Started") is visible. | - | `src/features/alforqan/components/tenant-navigation.tsx` | Locale: "ar" |
| TC-12 | ALFQ-TEN-SHL-011 | Verify the account menu "Back to website" link opens the public landing page | Main | User is signed in to a tenant | 1. Open any signed-in tenant page.<br>2. Open the account avatar menu in the top bar.<br>3. Click "Back to website" / "العودة إلى الموقع". | 1. The menu item is labeled "Back to website" / "العودة إلى الموقع".<br>2. The browser opens the locale landing page (`/ar` or `/en`).<br>3. The session is not signed out (returning to the tenant URL still shows the dashboard, not login). | ALFQ-TEN-SHL-007 | `src/components/layout/tenant-user-menu.tsx` | Locale: "ar" |

### Wave 4 — Mobile/medium nav opener

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-13 | ALFQ-TEN-SHL-015 | Verify a menu button opens the nav on small and medium screens | Main | User is signed in to a tenant | 1. Resize the window to a phone width (below 768px) on a tenant page such as `/ar/alforqan`.<br>2. Confirm the persistent sidebar is hidden and a square menu button is visible in the top bar.<br>3. Tap the menu button and pick a nav item (for example Halaqat).<br>4. Resize to a tablet/medium width (768px–1023px) and repeat: menu button still visible, sidebar still hidden, tap opens the panel. | 1. Below `lg` (1024px) the left/right sidebar is hidden.<br>2. A square menu opener is visible in the top bar at inline-start (not missing and not covered by the bar).<br>3. Tapping it opens the institute nav sheet with the same groups and links as the desktop sidebar.<br>4. Choosing a link closes the sheet and opens that page.<br>5. The same opener works at both phone width and medium/tablet width. | ALFQ-TEN-SHL-002 | `src/components/layout/tenant-top-chrome.tsx`, `src/features/alforqan/components/tenant-navigation.tsx` | Locale: "ar"; widths: ~390px and ~900px |
