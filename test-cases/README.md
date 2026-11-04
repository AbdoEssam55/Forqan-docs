# Alforqan test cases index

Canonical manual test definitions for **Alforqan**. Format: [Testing-feature `test-case-format.md`](O:/Company work/Company projects/ExcpTime/Testing-feature/docs/standards/test-case-format.md). Delivery waves: [Testing-feature `delivery-waves.md`](O:/Company work/Company projects/ExcpTime/Testing-feature/docs/standards/delivery-waves.md).

Run `tc` from this repo root (`testing-feature.project.json`). Marking **`ok`** respects **`Requires`**: the CLI refuses to write until prerequisite keys are **`succeeded`**, unless you pass **`--force`**.

Use **`tc blockers --wave N`** or **`tc ui`** → **Session** → **Wave** filter to focus on one wave. Each spec file has **`## Delivery waves`** and **`### Wave N — …`** headings.

## Halaqat cross-file delivery order

Run files in this order for the main teacher flow. **Grading** is independent.

| Order | File | In-file waves | Gate (succeed first) |
| ----- | ---- | ------------- | -------------------- |
| 1 | [halaqat-sessions](halaqat/frontend/halaqat-sessions-test-cases.md) | Wave 1 list → Wave 2 lifecycle | — |
| 2 | [halaqat-attendance](halaqat/frontend/halaqat-attendance-test-cases.md) | Wave 1 attendance → Wave 2 required marks / late / end-session | `ALFQ-HAL-SES-003`, `ALFQ-HAL-SES-005` (in-file) |
| 3 | [halaqat-evaluation](halaqat/frontend/halaqat-evaluation-test-cases.md) | Waves 1–3 + browse/sticky + autosave + outcome attendance (`ALFQ-HAL-EVA-036`–`038`) | In-file chain to `ALFQ-HAL-EVA-017`. **Per-key Related code** scopes `tc impact`. |
| 4 | [halaqat-reports](halaqat/frontend/halaqat-reports-test-cases.md) | Wave 1 digital → Wave 2 PDF | `ALFQ-HAL-EVA-011` on `ALFQ-HAL-REP-001` |
| 5 | [halaqat-notifications](halaqat/frontend/halaqat-notifications-test-cases.md) | Wave 1 bell → Wave 2 deep link | `ALFQ-HAL-EVA-011` on `ALFQ-HAL-NOT-004` (Wave 2 only) |
| — | [halaqat-grading](halaqat/frontend/halaqat-grading-test-cases.md) | Wave 1 weights → Wave 2 overrides | No session chain |
| — | [halaqat-session-permissions](halaqat/frontend/halaqat-session-permissions-test-cases.md) | Claim-based session manage/evaluate + admin uncheck | `ALFQ-HAL-SES-003` for session detail |

## Halaqat / frontend

- [`halaqat/frontend/halaqat-enrollment-test-cases.md`](halaqat/frontend/halaqat-enrollment-test-cases.md)
- [`halaqat/frontend/halaqat-sessions-test-cases.md`](halaqat/frontend/halaqat-sessions-test-cases.md)
- [`halaqat/frontend/halaqat-attendance-test-cases.md`](halaqat/frontend/halaqat-attendance-test-cases.md)
- [`halaqat/frontend/halaqat-evaluation-test-cases.md`](halaqat/frontend/halaqat-evaluation-test-cases.md)
- [`halaqat/frontend/halaqat-revision-evaluation-test-cases.md`](halaqat/frontend/halaqat-revision-evaluation-test-cases.md) — revision tab + autosave (`ALFQ-HAL-REV-011`) + outcome attendance (`ALFQ-HAL-REV-012`)
- [`halaqat/frontend/halaqat-grading-test-cases.md`](halaqat/frontend/halaqat-grading-test-cases.md)
- [`halaqat/frontend/halaqat-notifications-test-cases.md`](halaqat/frontend/halaqat-notifications-test-cases.md)
- [`halaqat/frontend/halaqat-reports-test-cases.md`](halaqat/frontend/halaqat-reports-test-cases.md)
- [`halaqat/frontend/halaqat-session-permissions-test-cases.md`](halaqat/frontend/halaqat-session-permissions-test-cases.md)
- [`halaqat/frontend/session-start-window-test-cases.md`](halaqat/frontend/session-start-window-test-cases.md) — start window UI (`ALFQ-SESWIN-009`, `014`, `019`, `026`–`032`)
- [`halaqat/frontend/halaqat-sessions-schedule-tab-test-cases.md`](halaqat/frontend/halaqat-sessions-schedule-tab-test-cases.md) — weekday chips, per-day sessions, preview, inline schedule, weekday retarget, weekly date roll (`ALFQ-HAL-SCH-001`–`020`)
- [`halaqat/frontend/halaqat-ui-overhaul-test-cases.md`](halaqat/frontend/halaqat-ui-overhaul-test-cases.md) — Phase 4 list/detail/sessions/report chrome (`ALFQ-HAL-UI-001`–`006`)

## Halaqat / backend

| File | Notes |
| ---- | ----- |
| [`halaqat/backend/halaqat-session-permissions-test-cases.md`](halaqat/backend/halaqat-session-permissions-test-cases.md) | Session manage/evaluate API gates and claim save/JWT fidelity |
| [`halaqat/backend/halaqat-attendance-test-cases.md`](halaqat/backend/halaqat-attendance-test-cases.md) | Late duration rules (xUnit) |
| [`halaqat/backend/halaqat-session-title-test-cases.md`](halaqat/backend/halaqat-session-title-test-cases.md) | Auto-generated bilingual session titles (xUnit) |
| [`halaqat/backend/halaqat-grading-score-test-cases.md`](halaqat/backend/halaqat-grading-score-test-cases.md) | Per-page mushaf grading math + config resolve (xUnit) |
| [`halaqat/backend/session-start-window-test-cases.md`](halaqat/backend/session-start-window-test-cases.md) | Session start window helper, auto-create, start gate, teacher list, notify stamp, weekly date roll (`ALFQ-SESWIN-001`–`025`, `033`–`035`) |
| [`halaqat/backend/halaqat-sessions-schedule-test-cases.md`](halaqat/backend/halaqat-sessions-schedule-test-cases.md) | Plan one current session per saved weekday (`ALFQ-HAL-SCH-013`–`015`) |

## Sard / frontend

| File | Waves |
| ---- | ----- |
| [`sard/frontend/sard-checkpoints-test-cases.md`](sard/frontend/sard-checkpoints-test-cases.md) | Wave 1 plan settings → Wave 2 committee/sessions → Wave 3 evaluation → Wave 4 gate → Wave 5 decision/points/notifications → Wave 6 certificate (in-file `Requires` chain from `ALFQ-SARD-PLAN-001`) |
| [`sard/frontend/sard-session-scope-test-cases.md`](sard/frontend/sard-session-scope-test-cases.md) | Wave 1 per-session respect-checkpoints checkbox → Wave 2 create-session start (`ALFQ-SARD-PLAN-008`–`009`, `ALFQ-SARD-SES-008`–`012`) |
| [`sard/frontend/sard-reports-test-cases.md`](sard/frontend/sard-reports-test-cases.md) | Teacher assignments, daily stop points, admin sard reports, teacher profile sard totals |
| [`sard/frontend/sard-review-ayahs-test-cases.md`](sard/frontend/sard-review-ayahs-test-cases.md) | Wave 1 review ayah viewer (mistakes only / all tested) → Wave 2 loading copy (`ALFQ-SARD-REV-001`–`004`) |

## Sard / backend

| File | Waves |
| ---- | ----- |
| [`sard/backend/sard-gate-bootstrap-test-cases.md`](sard/backend/sard-gate-bootstrap-test-cases.md) | Startup pass backfill, draft/submit gate parity, empty submit body, camelCase snapshots, from-end القرآن كامل → Al-Fatiha 1:7, from-end hifz progress (`ALFQ-SARD-BE-HIFZ-001`–`009`) |
| [`sard/backend/sard-session-scope-test-cases.md`](sard/backend/sard-session-scope-test-cases.md) | Create-session start range: previous checkpoint default, earlier checkpoint, mushaf start, free ayah, rejects (`ALFQ-SARD-BE-SCOPE-001`–`007`) |
| [`sard/backend/sard-session-access-test-cases.md`](sard/backend/sard-session-access-test-cases.md) | Evaluate access (admin/committee/assigned) and delete only before submit (`ALFQ-SARD-BE-ACCESS-001`–`004`) |
| [`sard/backend/sard-session-list-scope-test-cases.md`](sard/backend/sard-session-list-scope-test-cases.md) | List/get-by-id scoped to assigned teacher unless Manage sard (`ALFQ-SARD-BE-LIST-001`–`003`) |

## Homework / frontend

| File | Waves |
| ---- | ----- |
| [`homework/frontend/homework-teacher-test-cases.md`](homework/frontend/homework-teacher-test-cases.md) | Wave 4 teacher UI (`ALFQ-HW-UI-001`–`010`); Wave 5 hifz window (`ALFQ-HW-UI-011`–`024`, incl. from-end last ayah → previous surah ayah 1) |

## Homework / backend

| File | Notes |
| ---- | ----- |
| [`homework/backend/homework-api-test-cases.md`](homework/backend/homework-api-test-cases.md) | CRUD, ranges, status, from-end start cap (`ALFQ-HW-API-030`) |
| [`homework/backend/homework-evaluation-lock-test-cases.md`](homework/backend/homework-evaluation-lock-test-cases.md) | Eval lock + replace |
| [`homework/backend/homework-schema-api-test-cases.md`](homework/backend/homework-schema-api-test-cases.md) | Schema / list filters |

## Reports / frontend

| File | Waves |
| ---- | ----- |
| [`reports/frontend/family-reports-test-cases.md`](reports/frontend/family-reports-test-cases.md) | Wave 1 nav/hub (incl. parent claim guard `ALFQ-FAM-REP-022`) → Wave 3 timeline → Wave 4 filters (halaqa dropdown) → Wave 5 calendar → Wave 6 polish |
| [`reports/frontend/staff-session-reports-test-cases.md`](reports/frontend/staff-session-reports-test-cases.md) | Wave 1 hub/shell → Wave 2 toolbar (incl. session ignores period `ALFQ-SSR-UI-019`) → Wave 3 revision/sard tabs → Wave 4 KPI/export → Wave 5 mistakes column → Wave 6 honor-board export (`ALFQ-SSR-UI-035`–`039`) → Wave 7 late duration (`ALFQ-SSR-UI-040`) |
| [`reports/frontend/ranking-honor-board-test-cases.md`](reports/frontend/ranking-honor-board-test-cases.md) | Wave 1 staff ranking page (`ALFQ-RANK-UI-001`–`005`) → Wave 2 dashboard honor boards (`ALFQ-RANK-DASH-001`–`005`) |
| [`reports/frontend/reports-ui-overhaul-test-cases.md`](reports/frontend/reports-ui-overhaul-test-cases.md) | Phase 8 hub/memorization/sessions calendar/ranking chrome (`ALFQ-REP-UI-001`–`010`) |

## UI / frontend

| File | Waves |
| ---- | ----- |
| [`ui/frontend/modals-ui-overhaul-test-cases.md`](ui/frontend/modals-ui-overhaul-test-cases.md) | Phase 9 confirm/delete/bulk dialogs + toasts (`ALFQ-MOD-UI-001`–`007`) |
| [`ui/frontend/forms-ui-overhaul-test-cases.md`](ui/frontend/forms-ui-overhaul-test-cases.md) | Phase 10.1–10.2 field chrome + validation feedback (`ALFQ-FORM-UI-001`–`009`) |
| [`ui/frontend/tables-ui-overhaul-test-cases.md`](ui/frontend/tables-ui-overhaul-test-cases.md) | Phase 11.1 DataTable header/hover/sort/empty/pagination (`ALFQ-TBL-UI-001`–`006`) |
| [`ui/frontend/lists-cards-ui-overhaul-test-cases.md`](ui/frontend/lists-cards-ui-overhaul-test-cases.md) | Phase 11.2 card shell / grid / selection (`ALFQ-CARD-UI-001`–`005`) |
| [`ui/frontend/charts-ui-overhaul-test-cases.md`](ui/frontend/charts-ui-overhaul-test-cases.md) | Phase 11.3 chart bars / tooltips / legends (`ALFQ-CHART-UI-001`–`004`) |
| [`ui/frontend/feedback-status-ui-overhaul-test-cases.md`](ui/frontend/feedback-status-ui-overhaul-test-cases.md) | Phase 12 toasts / notification center / loading / empty states (`ALFQ-FEED-UI-001`–`007`) |
| [`ui/frontend/settings-admin-ui-overhaul-test-cases.md`](ui/frontend/settings-admin-ui-overhaul-test-cases.md) | Phase 13 settings sidebar / sections / switches / users hub / permissions (`ALFQ-SET-UI-001`–`007`) |
| [`ui/frontend/polish-ui-overhaul-test-cases.md`](ui/frontend/polish-ui-overhaul-test-cases.md) | Phase 14 hover / ripple / 44px / dark / skip-link (`ALFQ-POLISH-UI-001`–`009`) |
| [`ui/frontend/design-system-showcase-test-cases.md`](ui/frontend/design-system-showcase-test-cases.md) | Phase 15 design-system tabs (`ALFQ-DS-UI-001`–`007`) |

## Reports / backend

| File | Waves |
| ---- | ----- |
| [`reports/backend/family-reports-test-cases.md`](reports/backend/family-reports-test-cases.md) | Wave 1d parent/student report claims → Wave 2 list/children API → Wave 4 advanced filters → Wave 4c halaqa filter |
| [`reports/backend/staff-session-reports-test-cases.md`](reports/backend/staff-session-reports-test-cases.md) | Wave 1 hifz list API (incl. sessionId skips period `ALFQ-SSR-API-006`) → Wave 2 revision/sard/summary → Wave 3 export → Wave 4 mistake counts → Wave 5 honor-board formats → Wave 6 late minutes (`ALFQ-SSR-API-029`) |
| [`reports/backend/ranking-honor-board-test-cases.md`](reports/backend/ranking-honor-board-test-cases.md) | Wave 1 scoring helper (`ALFQ-RANK-API-001`–`005`) → Wave 2 Search API (`ALFQ-RANK-API-010`–`014`) → Wave 3 HonorBoard (`ALFQ-RANK-API-020`–`024`) |

## Students / frontend

| File | Waves |
| ---- | ----- |
| [`students/frontend/students-edit-test-cases.md`](students/frontend/students-edit-test-cases.md) | Wave 1 save → Wave 2 tab persistence (`ALFQ-STU-EDT-002` requires `ALFQ-STU-EDT-001`) → Wave 5 honor-board opt-out (`ALFQ-STU-EDT-007`) |
| [`students/frontend/student-registration-onboarding-test-cases.md`](students/frontend/student-registration-onboarding-test-cases.md) | Wave 1 admin create → Wave 2 first-login gates → Wave 6 simple password policy |
| [`students/frontend/students-ui-overhaul-test-cases.md`](students/frontend/students-ui-overhaul-test-cases.md) | Phase 5 list/profile/forms chrome (`ALFQ-STU-UI-001`–`006`) |
| [`students/frontend/students-list-filters-test-cases.md`](students/frontend/students-list-filters-test-cases.md) | List columns + server filters (`ALFQ-STU-LST-001`–`007`) |
| [`students/backend/students-list-filters-test-cases.md`](students/backend/students-list-filters-test-cases.md) | GET Students/List filters + DTO enrich (`ALFQ-STU-LST-BE-001`–`005`) |
| [`students/frontend/hifz-direction-test-cases.md`](students/frontend/hifz-direction-test-cases.md) | From-end juz progress and juz/surah pickers (`ALFQ-STU-HIFZ-001`–`003`) |

## Student actions

| File | Waves |
| ---- | ----- |
| [`student-actions/backend/student-actions-match-test-cases.md`](student-actions/backend/student-actions-match-test-cases.md) | Wave 1 match API (`ALFQ-ACT-BE-001`–`005`) |
| [`student-actions/frontend/student-actions-page-test-cases.md`](student-actions/frontend/student-actions-page-test-cases.md) | Wave 1 page + Apply (`ALFQ-ACT-UI-001`–`005`) |
| [`student-actions/backend/student-actions-commands-test-cases.md`](student-actions/backend/student-actions-commands-test-cases.md) | Wave 1 notify/deactivate API (`ALFQ-ACT-BE-010`–`014`) |
| [`student-actions/frontend/student-actions-commands-test-cases.md`](student-actions/frontend/student-actions-commands-test-cases.md) | Wave 1 notify/deactivate UI (`ALFQ-ACT-UI-010`–`014`) |

## Users / activation

| Spec | Notes |
| ---- | ----- |
| [`users/backend/user-activation-test-cases.md`](users/backend/user-activation-test-cases.md) | Core deactivate/activate sync (`ALFQ-USER-ACT-BE-001`–`006`) |
| [`users/backend/user-activation-live-halaqa-test-cases.md`](users/backend/user-activation-live-halaqa-test-cases.md) | Live halaqa & sessions (`ALFQ-USER-ACT-BE-010`–`015`) |
| [`users/backend/user-activation-live-pickers-test-cases.md`](users/backend/user-activation-live-pickers-test-cases.md) | Sard, homework, reports pickers (`ALFQ-USER-ACT-BE-020`–`024`) |
| [`users/backend/user-activation-counts-test-cases.md`](users/backend/user-activation-counts-test-cases.md) | Dashboard & list counts (`ALFQ-USER-ACT-BE-030`–`033`) |
| [`users/frontend/user-activation-test-cases.md`](users/frontend/user-activation-test-cases.md) | Activate UI & filters (`ALFQ-USER-ACT-FE-001`–`006`) |

## Parents / frontend

| File | Waves |
| ---- | ----- |
| [`parents/frontend/parent-registration-onboarding-test-cases.md`](parents/frontend/parent-registration-onboarding-test-cases.md) | Admin min-create wizard → first-login gates → simple password |
| [`parents/frontend/parents-ui-overhaul-test-cases.md`](parents/frontend/parents-ui-overhaul-test-cases.md) | Phase 7.1 list/profile + Phase 7.2 parent dashboard (`ALFQ-PRN-UI-001`–`009`) |

## Teachers / frontend

| File | Waves |
| ---- | ----- |
| [`teachers/frontend/teacher-registration-onboarding-test-cases.md`](teachers/frontend/teacher-registration-onboarding-test-cases.md) | Wave 1 admin create → Wave 2 first-login gates → Wave 3 validation parity |
| [`teachers/frontend/teachers-ui-overhaul-test-cases.md`](teachers/frontend/teachers-ui-overhaul-test-cases.md) | Phase 6 list/profile/forms chrome (`ALFQ-TCH-UI-001`–`006`) |

## Tenant shell / frontend

| File | Waves |
| ---- | ----- |
| [`tenant-shell/frontend/tenant-shell-test-cases.md`](tenant-shell/frontend/tenant-shell-test-cases.md) | Wave 1 top bar (`ALFQ-TEN-SHL-001`–`004`); Wave 2 nav/breadcrumbs/dashboard (`ALFQ-TEN-SHL-005`–`009`, `ALFQ-TEN-SHL-012`); Wave 3 back to landing (`ALFQ-TEN-SHL-010`–`011`); Wave 4 mobile/medium menu (`ALFQ-TEN-SHL-015`) |

## UI overhaul / frontend

| File | Waves |
| ---- | ----- |
| [`frontend/alforqan/auth-login-redesign-test-cases.md`](frontend/alforqan/auth-login-redesign-test-cases.md) | Split-screen login redesign, remember me, social login visibility |
| [`frontend/alforqan/auth-pages-overhaul-test-cases.md`](frontend/alforqan/auth-pages-overhaul-test-cases.md) | Phase 2 registration, password reset, onboarding auth UI |
| [`frontend/alforqan/landing-page-test-cases.md`](frontend/alforqan/landing-page-test-cases.md) | Landing page hero/roles/features/CTA/footer |

## Platform host / frontend (client demo — Phase 1)

Record in this order for sales demos (`tc ui` → Session, optional microphone). Keys use prefix `PLAT-*`.

| Order | File | In-file waves | Gate (succeed first) |
| ----- | ---- | ------------- | -------------------- |
| 1 | [host-auth](platform-host/frontend/host-auth-test-cases.md) | Wave 1 login/sign-out | — |
| 2 | [host-dashboard](platform-host/frontend/host-dashboard-test-cases.md) | Wave 1 overview | `PLAT-HOST-AUTH-001` |
| 3 | [tenants](platform-host/frontend/tenants-test-cases.md) | Wave 1 list/detail → Wave 2 create wizard | `PLAT-HOST-AUTH-001` |
| 4 | [tenant-modules](platform-host/frontend/tenant-modules-test-cases.md) | Wave 1 toggles → Wave 2 bulk | `PLAT-HOST-TEN-003` |
| 5 | [plans](platform-host/frontend/plans-test-cases.md) | Wave 1 module plans | `PLAT-HOST-AUTH-001` |
| 6 | [tenant-branding (host)](platform-host/frontend/tenant-branding-test-cases.md) | Wave 1 configure branding | `PLAT-HOST-TEN-003` |
| 7 | [tenant-auth](platform-tenant/frontend/tenant-auth-test-cases.md) | Wave 1 institute login | — |
| — | [login-lockout-rate-limit](platform-tenant/backend/login-lockout-rate-limit-test-cases.md) | Wave 1 rate limit / lockout (2 min) | — |
| — | [tenant-request-host-db](platform-tenant/backend/tenant-request-host-db-test-cases.md) | JWT status without extra Host DB lookup; LastActivity throttle; client abort (`ALFQ-TEN-REQ-001`–`003`) | — |
| 8 | [tenant-shell](platform-tenant/frontend/tenant-shell-test-cases.md) | Wave 1 layout/settings/locale | `PLAT-TEN-AUTH-001` |
| 9 | [tenant-module-gating](platform-tenant/frontend/tenant-module-gating-test-cases.md) | Wave 1 Reports nav → Wave 2 Payments block | `PLAT-TEN-AUTH-001`, host module keys |
| 10 | [tenant-branding (tenant)](platform-tenant/frontend/tenant-branding-test-cases.md) | Wave 1 visible branding | `PLAT-TEN-AUTH-001`, `PLAT-HOST-BRD-002` |

Platform specs cover multi-tenant **platform** behavior only (host portal, modules, branding, tenant shell). Alforqan product specs (halaqat, sard, students) are separate.

## Platform sales deep dive (Phase 1.5 — record with mic + docs)

After Phase 1 UI demos. Narration-heavy; open docs and repo folders while recording.

| Order | File | Keys | Gate |
| ----- | ---- | ---- | ---- |
| 11 | [architecture overview](platform-architecture/frontend/architecture-overview-test-cases.md) | `PLAT-ARCH-001`–`006` | Phase 1 host/tenant demos |
| 12 | [deployment model](platform-architecture/frontend/deployment-model-test-cases.md) | `PLAT-DEP-001`–`006` | `PLAT-ARCH-002` |
| 13 | [tenant onboarding](platform-onboarding/frontend/tenant-onboarding-test-cases.md) | `PLAT-ONB-001`–`005` | `PLAT-HOST-TEN-004`, `PLAT-TEN-AUTH-001` |

### Session recording folders

Clips are stored under `test-runs/current/evidence/<spec-file-path>/<n>-<Key>/` (one feature folder per `*-test-cases.md` file). After upgrading `tc`, run:

```bash
cd "O:/My work/Alforqan website/TestCases"
tc evidence renumber
```

That migrates legacy flat folders (`55-PLAT-HOST-AUTH-001`) into the nested layout.

## Revision daily (check-in)

| File | Layer |
| ---- | ----- |
| [`revision-daily/backend/halaqa-schedule-test-cases.md`](revision-daily/backend/halaqa-schedule-test-cases.md) | Halaqa schedule + timezone |
| [`revision-daily/backend/session-date-admin-test-cases.md`](revision-daily/backend/session-date-admin-test-cases.md) | Admin session date |
| [`revision-daily/frontend/session-date-admin-test-cases.md`](revision-daily/frontend/session-date-admin-test-cases.md) | Session date UI |
| [`revision-daily/backend/student-timezone-test-cases.md`](revision-daily/backend/student-timezone-test-cases.md) | Student timezone |
| [`revision-daily/frontend/student-timezone-test-cases.md`](revision-daily/frontend/student-timezone-test-cases.md) | Student form timezone |
| [`revision-daily/backend/daily-revision-schema-test-cases.md`](revision-daily/backend/daily-revision-schema-test-cases.md) | Log table / enum |
| [`revision-daily/backend/daily-revision-window-test-cases.md`](revision-daily/backend/daily-revision-window-test-cases.md) | Window helper |
| [`revision-daily/backend/daily-revision-coverage-test-cases.md`](revision-daily/backend/daily-revision-coverage-test-cases.md) | Coverage helper |
| [`revision-daily/backend/daily-revision-submit-test-cases.md`](revision-daily/backend/daily-revision-submit-test-cases.md) | PUT submit |
| [`revision-daily/backend/daily-revision-audio-test-cases.md`](revision-daily/backend/daily-revision-audio-test-cases.md) | Audio limits |
| [`revision-daily/frontend/daily-revision-record-test-cases.md`](revision-daily/frontend/daily-revision-record-test-cases.md) | Recorder |
| [`revision-daily/backend/daily-revision-report-test-cases.md`](revision-daily/backend/daily-revision-report-test-cases.md) | GET report |
| [`revision-daily/backend/daily-revision-correction-test-cases.md`](revision-daily/backend/daily-revision-correction-test-cases.md) | POST correct |
| [`revision-daily/frontend/daily-revision-correction-test-cases.md`](revision-daily/frontend/daily-revision-correction-test-cases.md) | Correction UI |
| [`revision-daily/frontend/daily-revision-submit-test-cases.md`](revision-daily/frontend/daily-revision-submit-test-cases.md) | Assignment panel |
| [`revision-daily/backend/daily-revision-audio-cleanup-test-cases.md`](revision-daily/backend/daily-revision-audio-cleanup-test-cases.md) | Outcome + 7-day cleanup |
| [`revision-daily/frontend/daily-revision-evaluation-view-test-cases.md`](revision-daily/frontend/daily-revision-evaluation-view-test-cases.md) | Teacher eval + report |
| [`revision-daily/frontend/halaqa-schedule-test-cases.md`](revision-daily/frontend/halaqa-schedule-test-cases.md) | Schedule UI |

## Backend

| File | Waves |
| ---- | ----- |
| [`api/backend/cors-test-cases.md`](api/backend/cors-test-cases.md) | Wave 1 preflight + credentialed login + 500 vs CORS |
