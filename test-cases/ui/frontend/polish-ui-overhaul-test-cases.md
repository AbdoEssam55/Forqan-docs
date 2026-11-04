# Polish & refinement UI overhaul — Frontend Test Cases

## Metadata
- Project: `alforqan`
- Feature: `ui`
- Layer: `frontend`
- Routes: `/{locale}/{tenantSlug}/students`, `/{locale}/{tenantSlug}/settings`, `/{locale}/{tenantSlug}/auth/login`
- Related code: `src/lib/brand/motion.ts`, `src/components/ui/button.tsx`, `src/components/brand/skip-to-content.tsx`
- Related docs: `docs/technical/tenant/polish-ui-overhaul.md`

## Scope

- Phase 14 micro-interactions, 44px touch targets, dark contrast, skip link and focus rings. Host portal is out of scope.

## Delivery waves

| Wave | Intent | Keys |
| ---- | ------ | ---- |
| 1 | Micro-interactions | `ALFQ-POLISH-UI-001`–`004` |
| 2 | Responsive / touch | `ALFQ-POLISH-UI-005`–`006` |
| 3 | Dark mode | `ALFQ-POLISH-UI-007` |
| 4 | Accessibility | `ALFQ-POLISH-UI-008`–`009` |

## Test cases

### Wave 1 — Micro-interactions

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-01 | ALFQ-POLISH-UI-001 | Verify a primary button lifts on hover and shows an emerald focus ring | Main | Login page or any tenant page with a primary button | 1. Open `/{tenantSlug}/auth/login`.<br>2. Hover "Sign In".<br>3. Tab until "Sign In" is focused. | 1. On hover the button moves up slightly and the shadow strengthens.<br>2. The icon (if any) scales up a little.<br>3. Keyboard focus shows an emerald ring (not a generic browser outline only). | - | `src/components/ui/button.tsx`, `src/lib/brand/motion.ts` | - |
| TC-02 | ALFQ-POLISH-UI-002 | Verify clicking a button plays a short ripple | Main | Same as TC-01 | 1. Click "Sign In" (or any Button) and watch the surface. | A brief light ripple appears on the button, then fades. | ALFQ-POLISH-UI-001 | `src/components/ui/button.tsx`, `src/styles/globals.css` | - |
| TC-03 | ALFQ-POLISH-UI-003 | Verify student grid cards lift on hover | Main | User can view students; grid view | 1. Open `/{tenantSlug}/students` in grid view.<br>2. Hover a student card. | 1. The card lifts (moves up) and the shadow grows.<br>2. The border shifts toward emerald. | - | `src/lib/brand/card.ts`, `src/components/ui/entity-card.tsx`, `src/features/alforqan/components/student-card.tsx` | Students exist |
| TC-04 | ALFQ-POLISH-UI-004 | Verify sidebar nav icons scale on hover | Main | Signed in; desktop sidebar visible | 1. Open a tenant page wide enough to show the sidebar.<br>2. Hover a nav item such as "Students". | 1. The nav icon scales up slightly.<br>2. The row stays at least 44px tall. | - | `src/features/alforqan/components/tenant-navigation.tsx` | Signed-in tenant user |

### Wave 2 — Responsive

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-05 | ALFQ-POLISH-UI-005 | Verify the mobile menu opener is at least 44px | Main | Signed in; viewport below 1024px | 1. Open a tenant page at phone width.<br>2. Measure the square menu button in the top bar.<br>3. Tap it. | 1. The menu button is at least 44×44 CSS pixels.<br>2. Tapping it opens the institute nav sheet. | - | `src/features/alforqan/components/tenant-navigation.tsx`, `src/components/layout/tenant-top-chrome.tsx`, `src/components/ui/button.tsx` | Signed-in tenant user |
| TC-06 | ALFQ-POLISH-UI-006 | Verify settings nav stays tappable on a narrow screen | Main | User can open Settings | 1. Open `/{tenantSlug}/settings` at phone width.<br>2. Look at Account / Grading / Sard plan.<br>3. Tap "Grading" if the claim allows. | 1. Settings section links sit in a horizontal row that can scroll if needed.<br>2. Each link is at least 44px tall.<br>3. Grading opens if the user has the claim. | ALFQ-POLISH-UI-005 | `src/components/brand/settings-shell.tsx`, `src/lib/brand/settings.ts` | Tenant admin |

### Wave 3 — Dark mode

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-07 | ALFQ-POLISH-UI-007 | Verify dark mode uses zinc surfaces and readable muted text | Main | User can toggle dark mode (or OS dark) | 1. Open a tenant page (students or settings) in dark mode.<br>2. Read body text, muted hints, and card borders. | 1. Page background is near-black zinc, not a blue-gray slate wash only.<br>2. Muted labels stay readable (light gray on dark).<br>3. Cards keep a faint emerald border. | - | `src/styles/globals.css`, `src/lib/brand/card.ts`, `src/components/brand/brand-page-shell.tsx` | Dark theme |

### Wave 4 — Accessibility

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-08 | ALFQ-POLISH-UI-008 | Verify Skip to content moves focus into the main pane | Main | Signed in to a tenant | 1. Open any signed-in tenant page.<br>2. Press Tab until "Skip to content" / "تخطى إلى المحتوى" is focused.<br>3. Press Enter. | 1. The skip control becomes visible when focused.<br>2. Activating it moves to the main content (`#tenant-main`), not the sidebar. | - | `src/components/brand/skip-to-content.tsx`, `src/components/auth/tenant-auth-guard.tsx` | Signed-in tenant user |
| TC-09 | ALFQ-POLISH-UI-009 | Verify the current sidebar page is marked for assistive tech | Main | Signed in; desktop sidebar | 1. Open Students.<br>2. Inspect the Students nav link (or use a screen reader). | 1. The Students item looks active (emerald highlight).<br>2. The link exposes current-page state (`aria-current="page"`). | ALFQ-POLISH-UI-004 | `src/features/alforqan/components/tenant-navigation.tsx` | Signed-in tenant user |
