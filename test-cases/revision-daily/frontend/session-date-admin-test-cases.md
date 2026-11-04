# Session date admin — Frontend Test Cases

## Metadata

- Layer: frontend
- Related code: `src/features/alforqan/components/sessions/halaqa-session-form.tsx`, `src/features/alforqan/sessions/halaqa-session-detail-content.tsx`
- Related docs: `Alforqan-Backend/docs/features/halaqat.md`
- Automation: -

## Scope

Non-admin cannot change session date on edit; create still shows datetime (API enforces weekday).

## Test cases

### Session form

| ID | Key | Case | Steps | Expected |
| --- | --- | --- | --- | --- |
| TC-01 | ALFQ-REVDAY-DATE-006 | Verify edit form disables scheduled datetime for non-admin | 1. Sign in as halaqa teacher (not Admin).<br>2. Open an existing session edit screen. | 1. The scheduled date/time control is disabled or read-only.<br>2. Notes and other fields remain editable. |
