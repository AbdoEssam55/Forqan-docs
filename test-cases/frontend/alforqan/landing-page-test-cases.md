# Landing Page Test Cases

## Metadata

- **Feature:** Landing Page
- **Layer:** frontend
- **Product:** Alforqan
- **Version:** 1.0
- **Last Updated:** 2026-09-04
- **Related code:** `src/app/[locale]/(public)/page.tsx`, `src/components/landing/hero-section.tsx`, `src/components/landing/role-cards.tsx`, `src/components/landing/features-section.tsx`, `src/components/landing/cta-section.tsx`, `src/components/landing/landing-footer.tsx`, `src/components/brand/brand-logo.tsx`, `src/components/brand/brand-background.tsx`, `src/lib/brand/assets.ts`
- **Related docs:** `docs/technical/landing-page.md`, `docs/technical/brand-assets.md`, `docs/explanation/landing-page-design.md`
- **Automation:** -

## Scope

This test suite covers the new landing page for the Alforqan platform, including hero section, role cards, features, CTAs, and footer. Tests verify visual appearance, animations, localization, responsiveness, and navigation.

## Test cases

### Hero Section

| ID | Key | Case | Steps | Expected | Type | Related code |
|----|-----|------|-------|----------|------|--------------|
| TC-01 | ALFQ-LANDING-HERO-001 | Verify hero section displays correctly on page load | 1. Navigate to the root URL (e.g., `http://localhost:3000/en`)<br>2. Observe the hero section | 1. The Alforqan logo image (`logo1.png`) displays in the center (square lockup: gold arch, Arabic "الفرقان", open book).<br>2. There is no BookOpen icon standing in for the logo.<br>3. Hero title "Empower Your Qur'an Journey" displays in bold, light text at a comfortable size (not oversized).<br>4. Subtitle and description text are visible on the dark scene background (`bg2.png`).<br>5. Two CTA buttons display: "Get Started" (green gradient) and "Learn More" (black background, white text, white outline). | UI | `src/components/landing/hero-section.tsx`, `src/components/brand/brand-logo.tsx`, `src/components/brand/brand-background.tsx` |
| TC-02 | ALFQ-LANDING-HERO-002 | Verify hero animations on page load | 1. Navigate to the root URL<br>2. Observe the animation sequence | 1. Hero title fades in and slides up smoothly.<br>2. Subtitle fades in after title with slight delay.<br>3. Description fades in after subtitle.<br>4. CTA buttons fade in last.<br>5. The `bg2.png` scene background is visible behind the text. | Animation | `src/components/landing/hero-section.tsx`, `src/components/brand/brand-background.tsx` |
| TC-03 | ALFQ-LANDING-HERO-003 | Verify "Get Started" CTA navigation | 1. Navigate to the root URL (e.g. `/en` or `/ar`)<br>2. Click the "Get Started" button | 1. Browser navigates to `/{locale}/{tenantSlug}/auth/login`.<br>2. Default tenant slug is "alforqan" (or `NEXT_PUBLIC_DEFAULT_TENANT_SLUG`).<br>3. English example: `/en/alforqan/auth/login`.<br>4. Tenant login page loads (not a 404). | Functional | `src/components/landing/hero-section.tsx`, `src/lib/config/tenant-paths.ts` |
| TC-04 | ALFQ-LANDING-HERO-004 | Verify "Learn More" CTA scrolls to features | 1. Navigate to the root URL<br>2. Click the "Learn More" button | Page smoothly scrolls to the features section (#features anchor). | Functional | `src/components/landing/hero-section.tsx` |
| TC-05 | ALFQ-LANDING-HERO-005 | Verify trust indicators display | 1. Navigate to the root URL<br>2. Scroll to view trust indicators below CTAs | 1. Three trust indicators display with green pulsing dots.<br>2. Text shows: "Trusted by 1000+ institutions", "50,000+ students", "Available in 20+ countries". | UI | `src/components/landing/hero-section.tsx` |

### Role Cards Section

| ID | Key | Case | Steps | Expected | Type | Related code |
|----|-----|------|-------|----------|------|--------------|
| TC-06 | ALFQ-LANDING-ROLES-001 | Verify role cards display correctly | 1. Navigate to the root URL<br>2. Scroll to the "Who Are You?" section | 1. Section heading "Who Are You?" displays in light text.<br>2. Four role cards display in a grid: Student, Teacher, Parent, Admin.<br>3. Each card shows an icon, title, description, and a black action button with white text.<br>4. Cards have distinct gradient colors.<br>5. The `dark-sky.avif` night sky fills the section behind the cards (`object-cover`, no empty bands).<br>6. The sky stays black — no teal or green overlay. | UI | `src/components/landing/role-cards.tsx`, `src/app/[locale]/(public)/page.tsx`, `src/components/brand/brand-background.tsx`, `src/lib/brand/assets.ts` |
| TC-07 | ALFQ-LANDING-ROLES-002 | Verify role card scroll animations | 1. Navigate to the root URL<br>2. Slowly scroll down to the role cards section | 1. Cards fade in and slide up as they enter the viewport.<br>2. Cards animate in sequence with slight stagger (left to right). | Animation | `src/components/landing/role-cards.tsx` |
| TC-08 | ALFQ-LANDING-ROLES-003 | Verify role card hover effects | 1. Navigate to the root URL<br>2. Scroll to role cards<br>3. Hover over each card | 1. Card lifts up slightly on hover.<br>2. Shadow increases.<br>3. Gradient overlay becomes visible. | UI | `src/components/landing/role-cards.tsx` |
| TC-09 | ALFQ-LANDING-ROLES-004 | Verify Student role card navigation | 1. Navigate to the root URL<br>2. Scroll to role cards<br>3. Click "Start Learning" button on Student card | 1. Browser navigates to `/{locale}/{tenantSlug}/auth/login`.<br>2. Default tenant slug is "alforqan" (or `NEXT_PUBLIC_DEFAULT_TENANT_SLUG`).<br>3. Tenant login page loads (not a 404). | Functional | `src/components/landing/role-cards.tsx`, `src/lib/config/tenant-paths.ts` |
| TC-10 | ALFQ-LANDING-ROLES-005 | Verify Teacher role card navigation | 1. Navigate to the root URL<br>2. Scroll to role cards<br>3. Click "Start Teaching" button on Teacher card | 1. Browser navigates to `/{locale}/{tenantSlug}/auth/login`.<br>2. Default tenant slug is "alforqan" (or `NEXT_PUBLIC_DEFAULT_TENANT_SLUG`).<br>3. Tenant login page loads (not a 404). | Functional | `src/components/landing/role-cards.tsx`, `src/lib/config/tenant-paths.ts` |
| TC-11 | ALFQ-LANDING-ROLES-006 | Verify Parent role card navigation | 1. Navigate to the root URL<br>2. Scroll to role cards<br>3. Click "View Progress" button on Parent card | 1. Browser navigates to `/{locale}/{tenantSlug}/auth/login`.<br>2. Default tenant slug is "alforqan" (or `NEXT_PUBLIC_DEFAULT_TENANT_SLUG`).<br>3. Tenant login page loads (not a 404). | Functional | `src/components/landing/role-cards.tsx`, `src/lib/config/tenant-paths.ts` |
| TC-12 | ALFQ-LANDING-ROLES-007 | Verify Admin role card navigation | 1. Navigate to the root URL<br>2. Scroll to role cards<br>3. Click "Manage Platform" button on Admin card | 1. Browser navigates to `/{locale}/{tenantSlug}/auth/login`.<br>2. Default tenant slug is "alforqan" (or `NEXT_PUBLIC_DEFAULT_TENANT_SLUG`).<br>3. Tenant login page loads (not a 404). | Functional | `src/components/landing/role-cards.tsx`, `src/lib/config/tenant-paths.ts` |
| TC-41 | ALFQ-LANDING-ROLES-008 | Verify the night sky stays fixed while scrolling through roles | 1. Navigate to the root URL<br>2. Scroll until the "Who Are You?" section fills most of the viewport<br>3. Keep scrolling slowly through the role cards | 1. The `dark-sky.avif` stars stay pinned to the viewport (they do not scroll away with the cards).<br>2. Role cards, heading, and subtitle move up over the same star field.<br>3. The sky stays black — no teal or green overlay. | Animation | `src/app/[locale]/(public)/page.tsx`, `src/components/brand/brand-background.tsx`, `src/components/landing/role-cards.tsx` |

### Features Section

| ID | Key | Case | Steps | Expected | Type |
|----|-----|------|-------|----------|------|
| TC-13 | ALFQ-LANDING-FEATURES-001 | Verify features section displays correctly | 1. Navigate to the root URL<br>2. Scroll to the "Everything You Need to Excel" section | 1. Section heading displays.<br>2. Six feature cards display in a grid.<br>3. Each card has an icon, title, and description.<br>4. Cards have distinct colors (emerald, blue, purple, orange, teal, yellow). | UI |
| TC-14 | ALFQ-LANDING-FEATURES-002 | Verify features content | 1. Navigate to the root URL<br>2. Scroll to features section<br>3. Read each card | 1. "Memorization Tracking" card with BookMarked icon (emerald).<br>2. "Revision Management" card with RefreshCw icon (blue).<br>3. "Halaqa Organization" card with Users icon (purple).<br>4. "Detailed Reports" card with FileText icon (orange).<br>5. "Narration Assessment" card with Mic icon (teal).<br>6. "Student Rankings" card with Trophy icon (yellow). | UI |
| TC-15 | ALFQ-LANDING-FEATURES-003 | Verify feature cards scroll animations | 1. Navigate to the root URL<br>2. Slowly scroll to features section | Cards fade in and slide up as they enter viewport, with slight stagger between cards. | Animation |
| TC-16 | ALFQ-LANDING-FEATURES-004 | Verify feature card hover effects | 1. Navigate to the root URL<br>2. Scroll to features<br>3. Hover over cards | 1. Card lifts up on hover.<br>2. Shadow increases. | UI |

### CTA Section

| ID | Key | Case | Steps | Expected | Type | Related code |
|----|-----|------|-------|----------|------|--------------|
| TC-17 | ALFQ-LANDING-CTA-001 | Verify CTA section displays correctly | 1. Navigate to the root URL<br>2. Scroll to the CTA section (dark sky background) | 1. The `dark-sky.avif` night sky fills the section behind the copy (`object-cover`, no empty bands).<br>2. The sky stays black — no teal or green overlay.<br>3. Heading "Ready to Transform Your Qur'an Learning?" displays in white.<br>4. Subtitle displays.<br>5. Two buttons: "Start Free Trial" (white bg, green text) and "Schedule Demo" (black bg, white text, white outline). | UI | `src/components/landing/cta-section.tsx`, `src/app/[locale]/(public)/page.tsx`, `src/components/brand/brand-background.tsx`, `src/lib/brand/assets.ts` |
| TC-18 | ALFQ-LANDING-CTA-002 | Verify CTA section scroll animation | 1. Navigate to the root URL<br>2. Slowly scroll to CTA section | Section fades in and slides up when entering viewport. | Animation | `src/components/landing/cta-section.tsx` |
| TC-42 | ALFQ-LANDING-CTA-005 | Verify the night sky stays fixed while scrolling through the CTA | 1. Navigate to the root URL<br>2. Scroll until the CTA section is in view<br>3. Keep scrolling slowly toward the footer | 1. The `dark-sky.avif` stars stay pinned to the viewport while CTA copy moves.<br>2. The same star field is visible behind roles and CTA (features and footer still cover it).<br>3. The sky stays black — no teal or green overlay. | Animation | `src/app/[locale]/(public)/page.tsx`, `src/components/brand/brand-background.tsx`, `src/components/landing/cta-section.tsx` |
| TC-19 | ALFQ-LANDING-CTA-003 | Verify "Start Free Trial" button navigation | 1. Navigate to the root URL<br>2. Scroll to CTA section<br>3. Click "Start Free Trial" button | 1. Browser navigates to `/{locale}/{tenantSlug}/auth/login`.<br>2. Default tenant slug is "alforqan" (or `NEXT_PUBLIC_DEFAULT_TENANT_SLUG`).<br>3. Tenant login page loads (not a 404). | Functional | `src/components/landing/cta-section.tsx`, `src/lib/config/tenant-paths.ts` |
| TC-20 | ALFQ-LANDING-CTA-004 | Verify "Schedule Demo" button navigation | 1. Navigate to the root URL<br>2. Scroll to CTA section<br>3. Click "Schedule Demo" button | Button navigates to `/contact`. | Functional | `src/components/landing/cta-section.tsx` |

### Footer

| ID | Key | Case | Steps | Expected | Type | Related code |
|----|-----|------|-------|----------|------|--------------|
| TC-21 | ALFQ-LANDING-FOOTER-001 | Verify footer displays correctly | 1. Navigate to the root URL<br>2. Scroll to the footer | 1. The `cover.png` brand cover fills the footer background (`object-cover`).<br>2. A dark layout overlay sits over the cover so footer text and links stay readable.<br>3. The Alforqan logo image (`logo1.png`) displays in the brand column.<br>4. Four columns: Brand, Quick Links, Legal, Follow Us.<br>5. Copyright notice at bottom. | UI | `src/components/landing/landing-footer.tsx`, `src/components/brand/brand-logo.tsx`, `src/components/brand/brand-background.tsx` |
| TC-22 | ALFQ-LANDING-FOOTER-002 | Verify footer links | 1. Navigate to the root URL<br>2. Scroll to footer<br>3. Hover over each link | 1. Quick Links: About, Features, Pricing, Contact (hover: emerald-400 color).<br>2. Legal: Privacy Policy, Terms of Service. | UI | - |
| TC-23 | ALFQ-LANDING-FOOTER-003 | Verify social media icons | 1. Navigate to the root URL<br>2. Scroll to footer<br>3. Hover over social icons | 1. Four social icons display: Twitter, Facebook, Instagram, GitHub.<br>2. Each icon is in a rounded square (zinc-800 bg).<br>3. On hover, background changes to emerald-600. | UI | - |
| TC-24 | ALFQ-LANDING-FOOTER-004 | Verify footer tagline and copyright | 1. Navigate to the root URL<br>2. Scroll to footer | 1. Tagline: "Empowering Qur'an memorization worldwide".<br>2. Copyright: "© 2026 Alforqan. All rights reserved." (with current year). | UI | - |

### Localization (Arabic)

| ID | Key | Case | Steps | Expected | Type |
|----|-----|------|-------|----------|------|
| TC-25 | ALFQ-LANDING-I18N-001 | Verify Arabic landing page content | 1. Navigate to `/ar`<br>2. Observe all sections | 1. All text displays in Arabic.<br>2. Hero title: "انطلق في رحلتك مع القرآن الكريم".<br>3. Role cards in Arabic.<br>4. Features in Arabic.<br>5. CTA in Arabic.<br>6. Footer in Arabic. | Localization |
| TC-26 | ALFQ-LANDING-I18N-002 | Verify Arabic layout (RTL) | 1. Navigate to `/ar`<br>2. Observe layout direction | 1. Text aligns right.<br>2. Icons and buttons flip horizontally.<br>3. Grid layouts mirror correctly.<br>4. Animations work in RTL context. | Localization |
| TC-27 | ALFQ-LANDING-I18N-003 | Verify Arabic typography | 1. Navigate to `/ar`<br>2. Observe fonts | 1. The logo image already includes Arabic "الفرقان" lockup text.<br>2. Body text uses Cairo font.<br>3. Text is readable and properly sized. | Localization |

### Responsive Design

| ID | Key | Case | Steps | Expected | Type | Related code |
|----|-----|------|-------|----------|------|--------------|
| TC-28 | ALFQ-LANDING-RESPONSIVE-001 | Verify mobile layout (375px width) | 1. Set viewport to 375px width<br>2. Navigate to root URL<br>3. Scroll through entire page | 1. Hero text scales down appropriately.<br>2. Role cards display in single column.<br>3. Features display in single column.<br>4. Footer stacks vertically.<br>5. All content is readable without horizontal scroll.<br>6. Roles and CTA `dark-sky.avif` covers the full section with no empty bands.<br>7. Hero `bg2.png` shows the mosque arch (not a flat teal fill).<br>8. "Learn More" and "Schedule Demo" stay black with white text. | Responsive | `src/components/landing/hero-section.tsx`, `src/components/landing/cta-section.tsx`, `src/components/brand/brand-background.tsx` |
| TC-29 | ALFQ-LANDING-RESPONSIVE-002 | Verify tablet layout (768px width) | 1. Set viewport to 768px width<br>2. Navigate to root URL<br>3. Scroll through entire page | 1. Hero text is medium size.<br>2. Role cards display in 2 columns.<br>3. Features display in 2 columns.<br>4. Footer remains horizontal. | Responsive | - |
| TC-30 | ALFQ-LANDING-RESPONSIVE-003 | Verify desktop layout (1280px width) | 1. Set viewport to 1280px width<br>2. Navigate to root URL<br>3. Scroll through entire page | 1. Hero text is largest size.<br>2. Role cards display in 4 columns.<br>3. Features display in 3 columns.<br>4. All sections centered with max-width constraint.<br>5. Roles and CTA `dark-sky.avif` covers the full section width (`object-cover`). | Responsive | - |
| TC-31 | ALFQ-LANDING-RESPONSIVE-004 | Verify touch interactions on mobile | 1. Use mobile device or emulator<br>2. Navigate to root URL<br>3. Tap all interactive elements | 1. All buttons are easily tappable (min 44x44px).<br>2. Hover effects don't interfere with tap.<br>3. Smooth scroll works on touch.<br>4. No unintended zoom on double-tap. | Responsive | - |
| TC-43 | ALFQ-LANDING-MOBILE-001 | Verify background images show on a phone | 1. Set viewport to 390px width (or open `/ar` on a phone)<br>2. View the hero<br>3. Scroll through roles, CTA, and footer | 1. Hero shows the `bg2.png` mosque arch (not a flat teal or green fill).<br>2. Roles and CTA show the `dark-sky.avif` star field (not a solid green block).<br>3. Footer shows `cover.png` under the dark overlay (not a solid teal fill). | UI | `src/components/brand/brand-background.tsx`, `src/components/landing/hero-section.tsx`, `src/components/landing/landing-footer.tsx`, `src/app/[locale]/(public)/page.tsx` |
| TC-44 | ALFQ-LANDING-MOBILE-002 | Verify black landing buttons keep white text on a phone | 1. Set viewport to 390px width (or open `/ar` on a phone)<br>2. Look at hero "Learn More"<br>3. Scroll to role card actions and CTA "Schedule Demo" | 1. "Learn More" has a black background, white text, and a white border.<br>2. Role card actions stay black with white text (they do not turn white).<br>3. "Schedule Demo" has a black background, white text, and a white border. | UI | `src/components/landing/hero-section.tsx`, `src/components/landing/role-cards.tsx`, `src/components/landing/cta-section.tsx` |

### Dark Mode

| ID | Key | Case | Steps | Expected | Type | Related code |
|----|-----|------|-------|----------|------|--------------|
| TC-32 | ALFQ-LANDING-DARKMODE-001 | Verify dark mode appearance | 1. Enable system dark mode<br>2. Navigate to root URL<br>3. Observe all sections | 1. Hero uses `bg2.png`; roles and CTA use `dark-sky.avif`.<br>2. Footer stays dark with the `cover.png` background.<br>3. Text remains readable with proper contrast (light text on dark sections).<br>4. Role and feature cards keep readable card surfaces.<br>5. Logo images remain visible on all sections. | UI | `src/components/landing/hero-section.tsx`, `src/components/landing/role-cards.tsx`, `src/components/landing/cta-section.tsx`, `src/components/brand/brand-background.tsx`, `src/components/landing/landing-footer.tsx` |
| TC-33 | ALFQ-LANDING-DARKMODE-002 | Verify dark mode animations | 1. Enable dark mode<br>2. Navigate to root URL<br>3. Observe animations | All animations work correctly in dark mode without visual glitches. | Animation | - |

### Performance

| ID | Key | Case | Steps | Expected | Type |
|----|-----|------|-------|----------|------|
| TC-34 | ALFQ-LANDING-PERF-001 | Verify page load performance | 1. Open DevTools Network tab<br>2. Navigate to root URL<br>3. Observe load time | 1. Page loads within 3 seconds on 3G connection.<br>2. Hero section visible within 1 second.<br>3. No console errors.<br>4. GSAP library loads correctly. | Performance |
| TC-35 | ALFQ-LANDING-PERF-002 | Verify scroll performance | 1. Navigate to root URL<br>2. Open DevTools Performance tab<br>3. Record while scrolling entire page | 1. Scroll remains smooth (60fps).<br>2. ScrollTrigger animations don't cause jank.<br>3. No layout thrashing. | Performance |

### Accessibility

| ID | Key | Case | Steps | Expected | Type |
|----|-----|------|-------|----------|------|
| TC-36 | ALFQ-LANDING-A11Y-001 | Verify keyboard navigation | 1. Navigate to root URL<br>2. Use Tab key to navigate through all interactive elements | 1. Tab order follows visual order.<br>2. All buttons and links are reachable.<br>3. Focus states are visible.<br>4. No keyboard trap. | Accessibility |
| TC-37 | ALFQ-LANDING-A11Y-002 | Verify heading hierarchy | 1. Navigate to root URL<br>2. Use screen reader or inspect heading structure | 1. Single h1: Hero title.<br>2. Section headings are h2.<br>3. Card titles are h3.<br>4. No skipped heading levels. | Accessibility |
| TC-38 | ALFQ-LANDING-A11Y-003 | Verify color contrast | 1. Navigate to root URL<br>2. Use browser accessibility tools to check contrast | 1. All text meets WCAG AA contrast ratio (4.5:1 for normal text).<br>2. Dark text on light backgrounds: high contrast.<br>3. Light text on dark backgrounds: high contrast.<br>4. CTA buttons: sufficient contrast. | Accessibility |

### Brand assets

| ID | Key | Case | Steps | Expected | Type | Related code |
|----|-----|------|-------|----------|------|--------------|
| TC-39 | ALFQ-LANDING-BRAND-001 | Verify every landing section uses the brand logo or scene background | 1. Navigate to the root URL<br>2. Scroll through hero, roles, features, CTA, and footer | 1. Hero shows the `bg2.png` mosque/lantern scene.<br>2. Roles and CTA show `dark-sky.avif` filling each section (`object-cover`), pinned to the viewport while those sections scroll.<br>3. Footer shows `cover.png` filling the section (`object-cover`) under a dark layout overlay.<br>4. Features section shows a light wash of `bg2.png` behind the cards.<br>5. Hero and footer show `logo1.png` (not a BookOpen icon). | UI | `src/components/brand/brand-logo.tsx`, `src/components/brand/brand-background.tsx`, `src/lib/brand/assets.ts`, `src/app/[locale]/(public)/page.tsx`, `src/components/landing/hero-section.tsx`, `src/components/landing/role-cards.tsx`, `src/components/landing/features-section.tsx`, `src/components/landing/cta-section.tsx`, `src/components/landing/landing-footer.tsx` |

### Typography

| ID | Key | Case | Steps | Expected | Type | Related code |
|----|-----|------|-------|----------|------|--------------|
| TC-40 | ALFQ-LANDING-TYPE-001 | Verify landing type is comfortable to read | 1. Navigate to the root URL on a desktop viewport (1280px)<br>2. Read the hero title, subtitle, and description<br>3. Scroll to roles, features, and CTA headings | 1. Hero title is smaller than a full-viewport shout: about 3xl on mobile and 5xl on large screens, not 7xl.<br>2. Subtitle and description are smaller than the title and easy to read in one glance.<br>3. Section headings (roles, features, CTA) are clearly smaller than the hero title.<br>4. Card titles and body text do not dominate the cards.<br>5. CTA buttons use the default large size, not extra-tall `text-lg` padding. | UI | `src/components/landing/hero-section.tsx`, `src/components/landing/role-cards.tsx`, `src/components/landing/features-section.tsx`, `src/components/landing/cta-section.tsx` |
