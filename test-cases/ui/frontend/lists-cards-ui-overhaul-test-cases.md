# Lists & cards UI overhaul — Frontend Test Cases

## Metadata
- Project: `alforqan`
- Feature: `ui`
- Layer: `frontend`
- Routes: `/{locale}/{tenantSlug}/students`, `/{locale}/{tenantSlug}/teachers`, `/{locale}/{tenantSlug}/parents`, `/{locale}/{tenantSlug}/halaqat`
- Related code: `src/lib/brand/card.ts`, `src/components/ui/entity-card.tsx`, `src/components/ui/card.tsx`, `src/features/alforqan/components/student-card.tsx`
- Related docs: `docs/technical/tenant/lists-cards-ui-overhaul.md`

## Scope

- Phase 11.2 card variants, entity card shell, card grid layout (no masonry).
- Grid view on students/teachers/parents/halaqat lists.

## Delivery waves

| Wave | Intent | Keys |
| ---- | ------ | ---- |
| 1 | Grid + card chrome | `ALFQ-CARD-UI-001`–`002` |
| 2 | Selection + hover | `ALFQ-CARD-UI-003`–`004` |
| 3 | Cross-list consistency | `ALFQ-CARD-UI-005` |

## Test cases

### Wave 1 — Grid + card chrome

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-01 | ALFQ-CARD-UI-001 | Verify students grid uses responsive card layout | Main | User can view students; grid view | 1. Open `/{tenantSlug}/students`.<br>2. Switch to grid view if needed.<br>3. Resize or check desktop vs narrow width. | 1. Cards appear in a multi-column grid on wide screens (2–3 columns).<br>2. Cards stack to one column on a narrow viewport. | - | `src/components/ui/entity-card.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/students/students-list-content.tsx` | Students exist |
| TC-02 | ALFQ-CARD-UI-002 | Verify student cards use rounded interactive chrome | Main | Grid view with students | 1. Look at a student card.<br>2. Hover the card. | 1. Card has rounded corners and a light border/shadow.<br>2. Hover lifts the card slightly and shifts the border toward emerald. | ALFQ-CARD-UI-001 | `src/features/alforqan/components/student-card.tsx`, `src/lib/brand/card.ts` | - |

### Wave 2 — Selection + hover

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-03 | ALFQ-CARD-UI-003 | Verify selected student card shows emerald ring | Main | User can manage students; grid view | 1. Check the checkbox on a student card. | 1. The card border turns emerald.<br>2. A soft emerald ring appears around the card. | ALFQ-CARD-UI-002 | `src/features/alforqan/components/student-card.tsx`, `src/components/ui/entity-card.tsx` | Manage-user claim |
| TC-04 | ALFQ-CARD-UI-004 | Verify loading grid shows skeleton cards | Main | Slow network or hard refresh on students | 1. Reload students while watching the first paint. | 1. Skeleton placeholders appear in the same card grid layout before data loads. | ALFQ-CARD-UI-001 | `src/app/[locale]/(tenant)/[tenantSlug]/students/students-list-content.tsx`, `src/components/ui/entity-card.tsx` | - |

### Wave 3 — Cross-list

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-05 | ALFQ-CARD-UI-005 | Verify teachers grid cards match the same shell chrome | Main | Teachers list with rows; grid view | 1. Open `/{tenantSlug}/teachers` in grid view.<br>2. Hover a teacher card. | 1. Cards use the same rounded interactive shell as students.<br>2. Hover shows the same emerald border lift behavior. | ALFQ-CARD-UI-002 | `src/features/alforqan/components/teacher-card.tsx`, `src/components/ui/entity-card.tsx` | Teachers exist |
