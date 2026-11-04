# Ranking + honor board — Frontend Test Cases

## Metadata

- Project: `alforqan`
- Feature: `reports`
- Layer: `frontend`
- Routes: `/{locale}/{tenantSlug}`, `/{locale}/{tenantSlug}/reports/ranking`
- APIs: `GET /API/V01/Ranking/HonorBoard`, `POST /API/V01/Ranking/Search`
- Related code: `src/features/alforqan/reports/honor-board-section.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/dashboard-content.tsx`, `src/features/alforqan/components/parent-dashboard-overview.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/reports/ranking/ranking-content.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/reports/ranking/layout.tsx`
- Related docs: `docs/technical/tenant/dashboard-and-ranking-guide.md`
- Automation: `-`

## Scope

- Staff ranking is one filterable top leaderboard (type, period, halaqa, session).
- URL params drive the Search API. Session is disabled until a halaqa is picked.
- Ranks 1–3 show medals. No worst/bottom control.
- Parents/students cannot open the ranking page. Hidden students still appear with a badge.
- Tenant dashboard shows two honor boards (this week + all time) for staff, student, and parent when Reports is on.
- Honor charts list opted-in students only (full names, no codes). Own / child rows highlight. Private rank line if outside top 10.

## Test pages

- `/{locale}/{tenantSlug}`
- `/{locale}/{tenantSlug}/reports/ranking`

## Test scenarios

- One board with type and period filters written to the URL.
- Session dropdown stays disabled until a halaqa is selected.
- Medals on 1–3; no bottom list.
- Parent hits no-permission, no table.
- Opted-out students still show with the hidden badge.
- Dashboard: This week + All time charts; highlight own/child; hidden names off charts; no worst list.

## Delivery waves

| Wave | Intent | Keys (representative) | Depends on (cross-file) |
| ---- | ------ | --------------------- | ----------------------- |
| 1 | Staff ranking page | `ALFQ-RANK-UI-001`–`005` | Search API Wave 2 (`ALFQ-RANK-API-010`) |
| 2 | Dashboard honor boards | `ALFQ-RANK-DASH-001`–`005` | HonorBoard API Wave 3 (`ALFQ-RANK-API-020`) |

## Test cases

### Wave 1 — Staff ranking page

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-01 | ALFQ-RANK-UI-001 | Verify staff ranking is one board with type + period + halaqa filters and URL params | Main | Staff JWT with a session-reports claim; Reports module on | 1. Open `/{tenantSlug}/reports/ranking`.<br>2. Confirm there is a single leaderboard (not two stacked ranking cards).<br>3. Choose type tab "Hifz" / "حفظ".<br>4. Expand filters with "Show filters" / "إظهار التصفية".<br>5. Choose period "This week" / "هذا الأسبوع".<br>6. Choose a halaqa from "Halaqa" / "الحلقة". | 1. One ranking table or empty message is shown; titles "Halaqa Ranking" / "ترتيب الحلقة" and "Global Ranking" / "الترتيب العام" do not appear.<br>2. The URL contains `type=hifz`.<br>3. The URL contains `preset=week`.<br>4. The URL contains `halaqaId` for the chosen halaqa.<br>5. Type tabs include "All" / "الكل", "Hifz" / "حفظ", "Revision" / "مراجعة", and "Sard" / "سرد". | - | `src/app/[locale]/(tenant)/[tenantSlug]/reports/ranking/ranking-content.tsx` | Staff account |
| TC-02 | ALFQ-RANK-UI-002 | Verify session dropdown disabled until halaqa; picking a session updates the board | Main | Staff on ranking; at least one halaqa with sessions | 1. Expand filters with "Show filters" / "إظهار التصفية".<br>2. Leave halaqa on "All" / "الكل".<br>3. Read the session dropdown labeled "Session" / "الجلسة".<br>4. Choose a halaqa.<br>5. Choose a session from the session dropdown. | 1. Before a halaqa is chosen, the session dropdown is disabled and hint "Pick a halaqa first" / "اختر حلقة أولاً" is shown.<br>2. After a halaqa is chosen, the session dropdown is enabled.<br>3. Choosing a session writes `sessionId` in the URL.<br>4. The ranking list reloads for that meeting. | ALFQ-RANK-UI-001 | `src/app/[locale]/(tenant)/[tenantSlug]/reports/ranking/ranking-content.tsx` | Halaqa with sessions |
| TC-03 | ALFQ-RANK-UI-003 | Verify ranks 1–3 show medals; no worst/bottom control | Main | Staff ranking page 1 has at least three qualifying students | 1. Open `/reports/ranking` on page 1 with results.<br>2. Read the podium and table rank cells.<br>3. Scan the page for a worst or bottom toggle. | 1. Rank 1 shows a trophy icon.<br>2. Rank 2 shows a medal icon.<br>3. Rank 3 shows an award icon.<br>4. No control to show worst/bottom students appears. | ALFQ-RANK-UI-001 | `src/app/[locale]/(tenant)/[tenantSlug]/reports/ranking/ranking-content.tsx` | Three+ ranked students |
| TC-04 | ALFQ-RANK-UI-004 | Verify parent cannot open `/reports/ranking` (no-permission, no table) | Main | Parent signed in without staff report claims; Reports module on | 1. Open `/{locale}/{tenantSlug}/reports/ranking` as the parent. | 1. Title "No permission" / "لا توجد صلاحية" is shown.<br>2. The ranking table and type tabs do not appear. | - | `src/app/[locale]/(tenant)/[tenantSlug]/reports/ranking/layout.tsx` | Parent account |
| TC-05 | ALFQ-RANK-UI-005 | Verify student with `showOnLeaderboard` off still appears on staff ranking with hidden badge | Main | Staff JWT; a qualifying student has honor board off | 1. Open `/reports/ranking` with filters that include that student.<br>2. Find the student row. | 1. The student is listed with their full name.<br>2. A muted badge matches "Hidden from honor board" / "مخفي من لوحة التميز". | ALFQ-RANK-UI-001 | `src/app/[locale]/(tenant)/[tenantSlug]/reports/ranking/ranking-content.tsx` | Student with `ShowOnLeaderboard` false |

### Wave 2 — Dashboard honor boards

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-10 | ALFQ-RANK-DASH-001 | Verify staff dashboard shows This week and All time boards with up to 10 full names | Main | Staff signed in; Reports module on; HonorBoard API available | 1. Open `/{locale}/{tenantSlug}` as staff.<br>2. Find the section titled "Honor board" / "لوحة التميز".<br>3. Read the two cards "This week" / "هذا الأسبوع" and "All time" / "طوال الوقت".<br>4. Count names on each chart.<br>5. Confirm "Full ranking" / "الترتيب الكامل" is present. | 1. Both This week and All time cards are visible side by side on desktop (stacked on mobile).<br>2. Each chart lists at most 10 students.<br>3. Each listed row shows a full name (not a student code alone).<br>4. "Full ranking" / "الترتيب الكامل" links to `/reports/ranking`.<br>5. No error toast appears when a chart is empty. | - | `src/features/alforqan/reports/honor-board-section.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/dashboard-content.tsx` | Staff account |
| TC-11 | ALFQ-RANK-DASH-002 | Verify parent dashboard highlights own child and shows private rank if outside top 10 | Main | Parent signed in with a linked child; Reports module on; child qualifies but is outside top 10 this week | 1. Open `/{locale}/{tenantSlug}` as the parent.<br>2. Read the This week honor-board card.<br>3. If the child is in the top 10, find their row.<br>4. If the child is not in the top 10, read the line under the chart. | 1. Parent does not see "Full ranking" / "الترتيب الكامل".<br>2. When the child is in the chart, that row uses the highlight background (primary tint / ring).<br>3. When the child is outside the chart and qualified, a line matches "{name} is #{rank} This week ({percent})" / "{name} في المرتبة #{rank} هذا الأسبوع ({percent})".<br>4. The child's full name is used (no student code). | - | `src/features/alforqan/reports/honor-board-section.tsx`, `src/features/alforqan/components/parent-dashboard-overview.tsx` | Parent with linked child |
| TC-12 | ALFQ-RANK-DASH-003 | Verify student dashboard highlights own row | Main | Student signed in; Reports module on; student is opted in and appears on at least one board | 1. Open `/{locale}/{tenantSlug}` as the student.<br>2. Find own name on This week or All time.<br>3. Confirm there is no "Full ranking" / "الترتيب الكامل" link. | 1. Own row uses the highlight background (primary tint / ring).<br>2. Own full name is shown (no student code).<br>3. "Full ranking" / "الترتيب الكامل" is not shown. | - | `src/features/alforqan/reports/honor-board-section.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/dashboard-content.tsx` | Student on honor board |
| TC-13 | ALFQ-RANK-DASH-004 | Verify a student with honor board off is not listed on the charts | Main | Staff (or parent of that student) on dashboard; Reports module on; that student has `ShowOnLeaderboard` false and enough slots | 1. Open `/{locale}/{tenantSlug}`.<br>2. Scan This week and All time name lists for that student. | 1. The opted-out student is not listed on either honor-board chart.<br>2. If signed in as that student or their parent, a private rank or "Keep going — 3 sessions needed to appear." / "استمر — يلزم 3 جلسات للظهور." line may still show under the chart. | ALFQ-RANK-DASH-001 | `src/features/alforqan/reports/honor-board-section.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/dashboard-content.tsx` | Student with `ShowOnLeaderboard` false |
| TC-14 | ALFQ-RANK-DASH-005 | Verify no “worst” or bottom students on the dashboard | Main | Any logged-in tenant role on dashboard; Reports module on | 1. Open `/{locale}/{tenantSlug}`.<br>2. Scan the honor-board section for a worst or bottom toggle or list. | 1. Only top honor-board charts are shown (This week and All time).<br>2. No control or list labeled worst/bottom appears. | ALFQ-RANK-DASH-001 | `src/features/alforqan/reports/honor-board-section.tsx`, `src/app/[locale]/(tenant)/[tenantSlug]/dashboard-content.tsx` | Any tenant account |
