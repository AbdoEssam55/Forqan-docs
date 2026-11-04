# Design system showcase — Frontend Test Cases

## Metadata
- Project: `alforqan`
- Feature: `ui`
- Layer: `frontend`
- Routes: `/{locale}/{tenantSlug}/settings/design-system`
- Related code: `src/app/[locale]/(tenant)/[tenantSlug]/settings/design-system/tenant-design-system-content.tsx`, `src/components/brand/settings-shell.tsx`
- Related docs: `docs/technical/tenant/design-system-showcase.md`

## Scope

- Phase 15 live design-system page (colors, type, icons, motion, components) behind ViewAdmins. Host portal is out of scope.

## Delivery waves

| Wave | Intent | Keys |
| ---- | ------ | ---- |
| 1 | Open + tabs | `ALFQ-DS-UI-001`–`002` |
| 2 | Palette / type / icons | `ALFQ-DS-UI-003`–`005` |
| 3 | Motion + components | `ALFQ-DS-UI-006`–`007` |

## Test cases

### Wave 1 — Open + tabs

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-01 | ALFQ-DS-UI-001 | Verify an admin can open the design system from settings | Main | Tenant user with ViewAdmins | 1. Open `/{tenantSlug}/settings`.<br>2. Click "Design system" / "نظام التصميم" in the settings sidebar. | 1. The design system page opens.<br>2. The page title is "Design system" / "نظام التصميم".<br>3. "Design system" is the active (emerald) sidebar item. | - | `src/components/brand/settings-shell.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/settings/design-system/tenant-design-system-content.tsx` | Admin account |
| TC-02 | ALFQ-DS-UI-002 | Verify the page has Colors, Typography, Icons, Motion, and Components tabs | Main | Same as TC-01 | 1. Open the design system page.<br>2. Read the tab row. | 1. Tabs labeled "Colors", "Typography", "Icons", "Motion", and "Components" are visible (or Arabic equivalents).<br>2. The Colors tab is selected by default. | ALFQ-DS-UI-001 | `src/app/[locale]/(tenant)/[tenantSlug]/settings/design-system/tenant-design-system-content.tsx` | - |

### Wave 2 — Palette / type / icons

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-03 | ALFQ-DS-UI-003 | Verify the Colors tab shows emerald/teal swatches and hex values | Main | Design system page open | 1. Stay on "Colors".<br>2. Read the swatch grid. | 1. Swatches include emerald-600 and teal-600.<br>2. Each swatch shows a token name and a hex value (for example "#059669").<br>3. The brand logo mark is visible. | ALFQ-DS-UI-002 | `src/app/[locale]/(tenant)/[tenantSlug]/settings/design-system/tenant-design-system-content.tsx`, `src/components/brand/brand-logo.tsx` | - |
| TC-04 | ALFQ-DS-UI-004 | Verify the Typography tab shows the type scale | Main | Design system page open | 1. Click "Typography".<br>2. Read the sample lines. | 1. Lines are labeled from "text-xs" up through larger heading sizes.<br>2. An Arabic sample line is visible. | ALFQ-DS-UI-002 | `src/app/[locale]/(tenant)/[tenantSlug]/settings/design-system/tenant-design-system-content.tsx` | - |
| TC-05 | ALFQ-DS-UI-005 | Verify the Icons tab lists product Lucide icons | Main | Design system page open | 1. Click "Icons".<br>2. Read the icon grid. | 1. Named icons such as Home, Users, and Settings are visible.<br>2. Icons use an emerald tint. | ALFQ-DS-UI-002 | `src/app/[locale]/(tenant)/[tenantSlug]/settings/design-system/tenant-design-system-content.tsx` | - |

### Wave 3 — Motion + components

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-06 | ALFQ-DS-UI-006 | Verify the Motion tab card lifts on hover | Main | Design system page open | 1. Click "Motion".<br>2. Hover the sample card.<br>3. Click "Ripple button". | 1. The card lifts and the border shifts toward emerald.<br>2. The button uses the emerald save chrome. | ALFQ-DS-UI-002 | `src/app/[locale]/(tenant)/[tenantSlug]/settings/design-system/tenant-design-system-content.tsx`, `src/lib/brand/card.ts` | - |
| TC-07 | ALFQ-DS-UI-007 | Verify the Components tab shows buttons, badges, empty state, and confirm | Main | Design system page open | 1. Click "Components".<br>2. Click "Open confirm" / "فتح التأكيد". | 1. Save, Cancel, Edit, and Delete buttons are visible.<br>2. Success / Warning / Info badges are visible.<br>3. A compact empty state is visible.<br>4. Confirm dialog opens with the deactivate title. | ALFQ-DS-UI-002 | `src/app/[locale]/(tenant)/[tenantSlug]/settings/design-system/tenant-design-system-content.tsx`, `src/components/brand/confirm-dialog.tsx`, `src/components/ui/empty-state.tsx` | - |
