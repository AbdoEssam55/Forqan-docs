# Login Page Redesign Test Cases

## Metadata

- **Feature:** Login Page (Redesigned)
- **Layer:** frontend
- **Product:** Alforqan
- **Version:** 2.0
- **Last Updated:** 2026-08-29
- **Related code:** `src/components/auth/auth-layout.tsx`, `src/components/auth/auth-card.tsx`, `src/components/auth/social-login-buttons.tsx`, `src/lib/auth/remembered-tenant-login.ts`, `src/features/alforqan/components/tenant-login-form.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/auth/login/page.tsx`, `src/components/brand/brand-logo.tsx`, `src/components/brand/brand-background.tsx`, `src/lib/brand/assets.ts`
- **Related docs:** `docs/technical/auth/login-page.md`, `docs/technical/brand-assets.md`
- **Automation:** -

## Scope

Tests for the redesigned login page with split-screen layout, improved styling, and smooth animations matching the landing page design system.

## Test cases

### Layout & Visual Design

| ID | Key | Case | Steps | Expected | Type |
|----|-----|------|-------|----------|------|
| TC-01 | ALFQ-AUTH-LOGIN-001 | Verify split-screen layout on desktop | 1. Open browser at 1280px+ width<br>2. Navigate to `/{locale}/{tenant}/auth/login` | 1. Left panel displays with the `bg2.png` dark teal mosque/lantern scene.<br>2. The Alforqan logo image (`logo1.png`) is visible on the left panel.<br>3. There is no BookOpen icon standing in for the logo.<br>4. Feature highlights list displays (3 items with bullet points).<br>5. Right panel displays with white background and login form.<br>6. Both panels are visible side-by-side. | UI |
| TC-02 | ALFQ-AUTH-LOGIN-002 | Verify Islamic scene background | 1. Navigate to login page on desktop<br>2. Observe left panel background | 1. The `bg2.png` scene fills the left panel.<br>2. A dark overlay keeps the white text readable.<br>3. The scene does not overwhelm the title or feature list. | UI |
| TC-03 | ALFQ-AUTH-LOGIN-003 | Verify the brand logo on mobile when the left panel is hidden | 1. Open the login page at a width below 1024px<br>2. Look above the login card | 1. The left brand panel is hidden.<br>2. `logo1.png` displays above the form card.<br>3. The login form is still usable. | UI |
| TC-04 | ALFQ-AUTH-LOGIN-004 | Verify login form card styling | 1. Navigate to login page<br>2. Observe right panel form | 1. Form is in a white card with rounded corners (2xl).<br>2. Card has subtle shadow.<br>3. Card has border (zinc-200).<br>4. Form has padding (8 units). | UI |
| TC-05 | ALFQ-AUTH-LOGIN-005 | Verify form heading and subtitle | 1. Navigate to login page<br>2. Read form heading | 1. Heading displays inside the AuthCard header.<br>2. Heading is large (3xl), bold, centered.<br>3. Subtitle displays below heading in muted color.<br>4. Text matches translation keys. | UI |

### Form Elements

| ID | Key | Case | Steps | Expected | Type |
|----|-----|------|-------|----------|------|
| TC-06 | ALFQ-AUTH-LOGIN-006 | Verify email/username input styling | 1. Navigate to login page<br>2. Inspect email input | 1. Input has Mail icon on left side (zinc-400).<br>2. Input height is 12 units.<br>3. Background is zinc-50.<br>4. Border is zinc-200.<br>5. Label is "Email" or translation.<br>6. Placeholder text displays. | UI |
| TC-07 | ALFQ-AUTH-LOGIN-007 | Verify email input focus state | 1. Navigate to login page<br>2. Click/focus email input | 1. Border color changes to emerald-500.<br>2. Focus ring appears (emerald-500/20).<br>3. Transition is smooth. | UI |
| TC-08 | ALFQ-AUTH-LOGIN-008 | Verify password input styling | 1. Navigate to login page<br>2. Inspect password input | 1. Input has Lock icon on left side.<br>2. Input has Eye/EyeOff toggle button on right side.<br>3. Input height is 12 units.<br>4. Background is zinc-50.<br>5. Label is "Password" with "Forgot password?" link on same line.<br>6. Placeholder displays. | UI |
| TC-09 | ALFQ-AUTH-LOGIN-009 | Verify password visibility toggle | 1. Navigate to login page<br>2. Type password in password field<br>3. Click Eye icon | 1. Password changes from dots/bullets to plain text.<br>2. Eye icon changes to EyeOff icon.<br>3. Click again, password hides and icon changes back.<br>4. Toggle works smoothly. | Functional |
| TC-10 | ALFQ-AUTH-LOGIN-010 | Verify "Forgot password" link position | 1. Navigate to login page<br>2. Locate "Forgot password" link | Link displays inline with password label (right-aligned), not below submit button. | UI |
| TC-11 | ALFQ-AUTH-LOGIN-011 | Verify submit button styling | 1. Navigate to login page<br>2. Inspect submit button | 1. Button spans full width.<br>2. Button has emerald-to-teal gradient background.<br>3. Button height is 12 units.<br>4. Button has shadow (emerald-500/25).<br>5. Text is white, medium weight. | UI |
| TC-12 | ALFQ-AUTH-LOGIN-012 | Verify submit button hover effect | 1. Navigate to login page<br>2. Hover over submit button | 1. Gradient darkens slightly.<br>2. Shadow intensity increases.<br>3. Transition is smooth (300ms). | UI |
| TC-39 | ALFQ-AUTH-LOGIN-039 | Verify "Remember me" checkbox is visible | 1. Navigate to login page<br>2. Locate the checkbox between password and submit | 1. Checkbox label shows "Remember me" / "تذكرني".<br>2. Checkbox is unchecked by default when no saved username exists.<br>3. Checkbox sits above the Sign In button. | UI |
| TC-40 | ALFQ-AUTH-LOGIN-040 | Verify "Remember me" restores username after sign-out | 1. Navigate to login page<br>2. Enter username/email and password<br>3. Check "Remember me"<br>4. Sign in successfully<br>5. Sign out and reopen login | 1. Username/email field is prefilled.<br>2. "Remember me" is checked.<br>3. Password field stays empty. | Functional |

### Animations

| ID | Key | Case | Steps | Expected | Type |
|----|-----|------|-------|----------|------|
| TC-13 | ALFQ-AUTH-LOGIN-013 | Verify page load animations | 1. Navigate to login page<br>2. Observe animation sequence | 1. Left panel slides in from left with fade-in (0.8s).<br>2. Right panel slides in from right with fade-in (0.8s, delay 0.2s).<br>3. Decorations (pattern, orbs) fade in (1.5s).<br>4. Form card fades in and slides up (0.6s, delay 0.3s).<br>5. Animations are smooth and professional. | Animation |
| TC-14 | ALFQ-AUTH-LOGIN-014 | Verify loading state animation | 1. Navigate to login page<br>2. Enter valid credentials<br>3. Click submit<br>4. Observe button | 1. Spinner icon appears next to button text.<br>2. Spinner rotates continuously.<br>3. Button is disabled during loading.<br>4. Input fields are disabled during loading. | Animation |

### Validation & Errors

| ID | Key | Case | Steps | Expected | Type |
|----|-----|------|-------|----------|------|
| TC-15 | ALFQ-AUTH-LOGIN-015 | Verify empty form submission | 1. Navigate to login page<br>2. Click submit without entering data | 1. Inline error message appears under email input.<br>2. Inline error message appears under password input.<br>3. Error text is red.<br>4. Input borders turn red.<br>5. Form does not submit. | Validation |
| TC-16 | ALFQ-AUTH-LOGIN-016 | Verify password too short error | 1. Navigate to login page<br>2. Enter email<br>3. Enter password with < 6 characters<br>4. Try to submit | 1. Inline error appears under password: "Password must be at least 6 characters".<br>2. Password input border is red.<br>3. Form does not submit. | Validation |
| TC-17 | ALFQ-AUTH-LOGIN-017 | Verify invalid credentials error | 1. Navigate to login page<br>2. Enter invalid username/password<br>3. Submit form | 1. Toast notification appears with error message.<br>2. Toast is dismissible.<br>3. Form remains filled (data not cleared).<br>4. User can retry. | Functional |
| TC-18 | ALFQ-AUTH-LOGIN-018 | Verify error input border style | 1. Navigate to login page<br>2. Trigger validation error on email | 1. Email input border changes to red-500.<br>2. Focus ring (when focused) is red-500/20.<br>3. Error state is visually distinct from normal state. | UI |

### Functionality

| ID | Key | Case | Steps | Expected | Type |
|----|-----|------|-------|----------|------|
| TC-19 | ALFQ-AUTH-LOGIN-019 | Verify successful login | 1. Navigate to login page<br>2. Enter valid tenant credentials<br>3. Click submit | 1. Loading spinner appears.<br>2. On success, user is redirected to dashboard or return URL.<br>3. Auth tokens are saved.<br>4. No errors display. | Functional |
| TC-20 | ALFQ-AUTH-LOGIN-020 | Verify "Forgot password" link navigation | 1. Navigate to login page<br>2. Click "Forgot password?" link | Link navigates to `/{tenant}/auth/forgot-password` or correct route. | Functional |
| TC-21 | ALFQ-AUTH-LOGIN-021 | Verify return URL handling | 1. Navigate to protected page (triggers redirect to login with ?returnUrl=...)<br>2. Enter credentials and submit | After successful login, user is redirected to the original protected page (returnUrl). | Functional |
| TC-22 | ALFQ-AUTH-LOGIN-022 | Verify form autofill compatibility | 1. Navigate to login page<br>2. Use browser password manager to autofill | 1. Username/email autofills correctly.<br>2. Password autofills correctly.<br>3. Autofilled inputs have proper styling.<br>4. Form can be submitted after autofill. | Functional |

### Responsive Design

| ID | Key | Case | Steps | Expected | Type |
|----|-----|------|-------|----------|------|
| TC-23 | ALFQ-AUTH-LOGIN-023 | Verify mobile layout (375px) | 1. Set viewport to 375px width<br>2. Navigate to login page | 1. Left panel is hidden.<br>2. Right panel takes full width.<br>3. Form card is centered.<br>4. All inputs and buttons are touch-friendly (height 12).<br>5. Text is readable.<br>6. No horizontal scroll. | Responsive |
| TC-24 | ALFQ-AUTH-LOGIN-024 | Verify tablet layout (768px) | 1. Set viewport to 768px width<br>2. Navigate to login page | 1. Left panel is hidden.<br>2. Right panel takes full width.<br>3. Form is centered with appropriate max-width.<br>4. Layout is clean and usable. | Responsive |
| TC-25 | ALFQ-AUTH-LOGIN-025 | Verify desktop layout (1024px) | 1. Set viewport to 1024px width<br>2. Navigate to login page | 1. Left panel is visible (40% width).<br>2. Right panel visible (60% width).<br>3. Both panels side-by-side.<br>4. Content fits without overflow. | Responsive |
| TC-26 | ALFQ-AUTH-LOGIN-026 | Verify large desktop layout (1920px) | 1. Set viewport to 1920px width<br>2. Navigate to login page | 1. Split-screen layout maintained.<br>2. Content centered appropriately.<br>3. No excessive whitespace.<br>4. Left panel remains at reasonable width. | Responsive |

### Localization (Arabic)

| ID | Key | Case | Steps | Expected | Type |
|----|-----|------|-------|----------|------|
| TC-27 | ALFQ-AUTH-LOGIN-027 | Verify Arabic language content | 1. Navigate to `/ar/{tenant}/auth/login` | 1. All text displays in Arabic.<br>2. Heading, labels, placeholders, button text in Arabic.<br>3. "Forgot password?" link in Arabic.<br>4. Brand title "الفرقان" displays correctly. | Localization |
| TC-28 | ALFQ-AUTH-LOGIN-028 | Verify RTL layout | 1. Navigate to Arabic login page | 1. Text aligns right.<br>2. Form elements remain functional.<br>3. Icons remain in logical positions (Mail left, Eye right).<br>4. Layout mirrors correctly. | Localization |
| TC-29 | ALFQ-AUTH-LOGIN-029 | Verify Arabic typography | 1. Navigate to Arabic login page<br>2. Inspect fonts | 1. Body text uses Cairo font.<br>2. The logo image shows the Arabic "الفرقان" lockup.<br>3. Text is readable and properly sized. | Localization |

### Dark Mode

| ID | Key | Case | Steps | Expected | Type |
|----|-----|------|-------|----------|------|
| TC-30 | ALFQ-AUTH-LOGIN-030 | Verify dark mode appearance | 1. Enable system dark mode<br>2. Navigate to login page | 1. Form card background is dark (zinc-900).<br>2. Input backgrounds are dark (zinc-800).<br>3. Borders are zinc-700.<br>4. Text is readable with proper contrast.<br>5. Left panel gradient remains vibrant. | UI |
| TC-31 | ALFQ-AUTH-LOGIN-031 | Verify dark mode animations | 1. Enable dark mode<br>2. Navigate to login page<br>3. Observe animations | All animations work correctly in dark mode without visual glitches. | Animation |

### Accessibility

| ID | Key | Case | Steps | Expected | Type |
|----|-----|------|-------|----------|------|
| TC-32 | ALFQ-AUTH-LOGIN-032 | Verify keyboard navigation | 1. Navigate to login page<br>2. Use Tab key to navigate | 1. Tab order: email → password → forgot password → remember me → submit.<br>2. All interactive elements are reachable.<br>3. Focus states are visible (emerald ring).<br>4. No keyboard trap. | Accessibility |
| TC-33 | ALFQ-AUTH-LOGIN-033 | Verify form submission with Enter key | 1. Navigate to login page<br>2. Enter credentials<br>3. Press Enter key | Form submits without clicking button. | Accessibility |
| TC-34 | ALFQ-AUTH-LOGIN-034 | Verify password toggle button accessibility | 1. Navigate to login page<br>2. Tab to password toggle button<br>3. Press Enter or Space | 1. Button is keyboard accessible.<br>2. Has aria-label ("Show password" / "Hide password").<br>3. Activates on Enter/Space key. | Accessibility |
| TC-35 | ALFQ-AUTH-LOGIN-035 | Verify screen reader compatibility | 1. Enable screen reader<br>2. Navigate to login page<br>3. Navigate through form | 1. Labels are announced.<br>2. Input types are announced.<br>3. Error messages are announced.<br>4. Button states are announced (loading, disabled). | Accessibility |
| TC-36 | ALFQ-AUTH-LOGIN-036 | Verify color contrast | 1. Navigate to login page<br>2. Use browser accessibility tools | 1. Text contrast meets WCAG AA (4.5:1+).<br>2. Input labels readable.<br>3. Error text readable.<br>4. Link text readable. | Accessibility |

### Performance

| ID | Key | Case | Steps | Expected | Type |
|----|-----|------|-------|----------|------|
| TC-37 | ALFQ-AUTH-LOGIN-037 | Verify page load performance | 1. Open DevTools Network tab<br>2. Navigate to login page<br>3. Measure load time | 1. Page loads in < 2 seconds on 3G.<br>2. Form is interactive in < 1 second.<br>3. No console errors. | Performance |
| TC-38 | ALFQ-AUTH-LOGIN-038 | Verify animation performance | 1. Open DevTools Performance tab<br>2. Navigate to login page<br>3. Record animations | 1. Animations maintain 60fps.<br>2. No jank or stuttering.<br>3. GSAP animations are smooth. | Performance |
