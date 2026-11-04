# Halaqa grading configuration — Frontend Test Cases

## Metadata
- Project: `alforqan`
- Feature: `halaqat`
- Layer: `frontend`
- Routes: `/{locale}/{tenantSlug}/settings/grading`
- APIs: `GET/POST` grading configs batch upsert
- Related code: `src/app/[locale]/(tenant)/[tenantSlug]/settings/grading/tenant-grading-settings-content.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/settings/grading/grading-settings-helpers.ts`
- Related docs: `docs/technical/tenant/grading-settings.md`, `docs/technical/tenant/halaqat-guide.md`

## Scope

- Per-scope (memorization / revision / sard) allowed-per-page, weight, active, and lock teacher override.
- Institute default vs per-halaqa overrides.
- Student exempt UI removed.

## Test cases

### Settings — per-page allowance

| ID | Key | Case | Type | Steps | Expected | Related code |
|----|-----|------|------|-------|----------|--------------|
| TC-01 | ALFQ-HAL-GRA-001 | Verify grading settings page opens for permitted users | Main | 1. Open Settings.<br>2. Click Grading. | 1. Title shows "Grading configuration" (or Arabic).<br>2. Tabs for memorization, revision, and sard are visible. | `src/app/[locale]/(tenant)/[tenantSlug]/settings/grading/tenant-grading-settings-content.tsx` |
| TC-02 | ALFQ-HAL-GRA-002 | Verify allowed per page and weight save | Main | 1. On memorization tab, set حفظ allowed per page to `2` and weight `1.5`.<br>2. Click "Save weights". | 1. Toast "Weights saved".<br>2. Fields still show allowed `2` and weight `1.5` (not defaults `1` / `0.5` / `0.25`).<br>3. Reload still shows those saved values. | `src/app/[locale]/(tenant)/[tenantSlug]/settings/grading/grading-settings-helpers.ts`, `src/app/[locale]/(tenant)/[tenantSlug]/settings/grading/tenant-grading-settings-content.tsx`, `src/types/halaqa-session.ts` |
| TC-03 | ALFQ-HAL-GRA-003 | Verify sard scope tab saves independently | Main | 1. Open Sard tab.<br>2. Change تجويد allowed per page and weight.<br>3. Click "Save weights". | 1. Sard fields still show the entered values after save (not defaults).<br>2. Reload on Sard still shows those values.<br>3. Memorization tab values are unchanged. | `src/app/[locale]/(tenant)/[tenantSlug]/settings/grading/grading-settings-helpers.ts`, `src/app/[locale]/(tenant)/[tenantSlug]/settings/grading/tenant-grading-settings-content.tsx`, `src/types/halaqa-session.ts` |
| TC-04 | ALFQ-HAL-GRA-004 | Verify lock teacher override toggle saves | Main | 1. Enable "Lock teacher allowance override".<br>2. Save weights. | 1. Success toast.<br>2. Evaluate screen shows locked message (see evaluation grading spec). | `src/app/[locale]/(tenant)/[tenantSlug]/settings/grading/tenant-grading-settings-content.tsx` |
| TC-05 | ALFQ-HAL-GRA-005 | Verify no student exempt section | Regression | 1. Open grading settings. | No "Student overrides" or "Exempt" controls appear. | `src/app/[locale]/(tenant)/[tenantSlug]/settings/grading/tenant-grading-settings-content.tsx` |
| TC-06 | ALFQ-HAL-GRA-006 | Verify users without access see no access message | Edge | 1. Open grading URL without halaqa view claim. | Message states user does not have access. | `src/app/[locale]/(tenant)/[tenantSlug]/settings/grading/tenant-grading-settings-content.tsx` |
| TC-07 | ALFQ-HAL-GRA-007 | Verify revision tab shows saved weights after save | Main | 1. Open Revision tab.<br>2. Set حفظ weight to `2`.<br>3. Click "Save weights". | 1. Revision حفظ weight still shows `2` (not default `1`).<br>2. Reload on Revision tab still shows `2`. | `src/app/[locale]/(tenant)/[tenantSlug]/settings/grading/grading-settings-helpers.ts`, `src/app/[locale]/(tenant)/[tenantSlug]/settings/grading/tenant-grading-settings-content.tsx`, `src/types/halaqa-session.ts` |
