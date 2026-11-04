# Ranking + honor board — Backend Test Cases

## Metadata

- Project: `alforqan`
- Feature: `reports`
- Layer: `backend`
- Routes: `/API/V01/Ranking/Search`, `/API/V01/Ranking/Halaqa/{halaqaId}`, `/API/V01/Ranking/Global`, `/API/V01/Ranking/HonorBoard`
- APIs: `POST /API/V01/Ranking/Search`, `POST /API/V01/Ranking/Halaqa/{halaqaId}`, `POST /API/V01/Ranking/Global`, `GET /API/V01/Ranking/HonorBoard`
- Related code: `Alforqan/Alforqan.Core/Features/Ranking/Helpers/RankingScoreHelper.cs`, `Alforqan/Alforqan.Core/Features/Ranking/Helpers/RankingSlotQueryHelper.cs`, `Alforqan/Alforqan.Core/Features/Ranking/Helpers/RankingHonorBoardHelper.cs`, `Alforqan/Alforqan.Core/Features/Ranking/Queries/Handlers/GetRankingSearchQueryHandler.cs`, `Alforqan/Alforqan.Core/Features/Ranking/Queries/Handlers/GetHonorBoardQueryHandler.cs`, `Alforqan/Alforqan.Api/Controllers/Ranking/RankingController.cs`
- Related docs: `docs/features/ranking.md`, `docs/features/dashboard.md`
- Automation: `Tests/Alforqan.Core.Tests/Features/Ranking/RankingScoreHelperTests.cs`, `Tests/Alforqan.Core.Tests/Features/Ranking/RankingSlotQueryHelperTests.cs`, `Tests/Alforqan.Core.Tests/Features/Ranking/RankingHonorBoardHelperTests.cs`

## Scope

- Pure ranking aggregator: slot averages, min slots, sort, and opted-in display ranks.
- Staff `POST Ranking/Search` live slots (hifz, revision, sard, absences). No teacher scope. Hidden students stay on Search.
- Old Halaqa/Global endpoints wrap Search.
- HonorBoard: any authenticated tenant user; opted-in top 10 plus private myEntries. Hidden students omitted from Items.

## Test pages

- `/API/V01/Ranking/Search`
- `/API/V01/Ranking/HonorBoard`

## Test scenarios

- Mixed hifz, revision, and sard slots average equally.
- Absence is 0% in the overall average only.
- Fewer than 3 slots are excluded unless a session filter sets min to 1.
- Sort uses average, then slot count, then recency.
- Opted-in take keeps true rank and consecutive display ranks.
- Parents/students cannot call Search.
- Teachers see the whole tenant.
- `type=hifz` does not include sard-only students.
- A single session ranks with absences as 0% and min slots 1.
- Hidden students still appear on Search.
- Parents can call HonorBoard but not Search.
- HonorBoard Items omit hidden students; display ranks stay consecutive.
- myEntries includes a hidden child with true rank.
- Week and all-time use different date windows.
- HonorBoard take is capped at 10.

## Delivery waves

| Wave | Intent | Keys (representative) | Depends on (cross-file) |
| ---- | ------ | --------------------- | ----------------------- |
| 1 | Scoring helper | `ALFQ-RANK-API-001`–`005` | — |
| 2 | Search API | `ALFQ-RANK-API-010`–`014` | — |
| 3 | Honor board API | `ALFQ-RANK-API-020`–`024` | — |

## Test cases

### Wave 1 — Scoring helper

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-01 | ALFQ-RANK-API-001 | Verify equal average of mixed hifz/revision/sard slots | Main | Three slots for one student: hifz 90, revision 80, sard 70 | 1. Call `RankingScoreHelper.Aggregate` with min slots 3. | 1. The student qualifies.<br>2. `averagePercent` is 80.<br>3. Type averages are 90 / 80 / 70. | - | `Alforqan/Alforqan.Core/Features/Ranking/Helpers/RankingScoreHelper.cs` | hifz 90, revision 80, sard 70 |
| TC-02 | ALFQ-RANK-API-002 | Verify absence slot counts as 0% in the overall average but not in type averages | Main | Three slots: hifz 100, revision 80, absence 0 | 1. Call `RankingScoreHelper.Aggregate` with min slots 3. | 1. `averagePercent` is 60.<br>2. `hifzAverage` is 100.<br>3. `revisionAverage` is 80.<br>4. `sardAverage` is null. | - | `Alforqan/Alforqan.Core/Features/Ranking/Helpers/RankingScoreHelper.cs` | hifz 100, revision 80, absence 0 |
| TC-03 | ALFQ-RANK-API-003 | Verify students with fewer than 3 slots are excluded; `sessionId` min is 1 | Main | One student with 2 slots; one student with 1 slot | 1. Aggregate with `MinSlots(null)`.<br>2. Aggregate with `MinSlots(sessionId)`. | 1. `MinSlots(null)` is 3 and both students are dropped.<br>2. `MinSlots(sessionId)` is 1 and both students qualify. | - | `Alforqan/Alforqan.Core/Features/Ranking/Helpers/RankingScoreHelper.cs` | 2-slot and 1-slot students |
| TC-04 | ALFQ-RANK-API-004 | Verify sort: higher average first; tie → more slots; then more recent | Main | Four qualifying students with mixed averages, slot counts, and dates | 1. Call `RankingScoreHelper.Aggregate` with min slots 3. | 1. Highest average is rank 1.<br>2. Same average: more slots ranks higher.<br>3. Same average and slot count: more recent `lastOccurredOn` ranks higher. | - | `Alforqan/Alforqan.Core/Features/Ranking/Helpers/RankingScoreHelper.cs` | Four named students |
| TC-05 | ALFQ-RANK-API-005 | Verify opted-in take keeps true `Rank` and assigns consecutive display ranks | Main | Five ranked students; ranks 1 and 3 are hidden | 1. Call `TakeTopOptedIn` with take 2 on the opted-in set. | 1. First item `displayRank` is 1 and true `rank` is 2.<br>2. Second item `displayRank` is 2 and true `rank` is 4.<br>3. Hidden students are omitted. | ALFQ-RANK-API-004 | `Alforqan/Alforqan.Core/Features/Ranking/Helpers/RankingScoreHelper.cs` | Two hidden, three opted-in |

### Wave 2 — Search API

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-10 | ALFQ-RANK-API-010 | Verify parent/student JWT cannot call `POST Ranking/Search` (403) | Main | Parent or student JWT; Reports module on | 1. `POST /API/V01/Ranking/Search` with `type` `all`. | 1. HTTP 403.<br>2. No ranking items returned. | - | `Alforqan/Alforqan.Api/Controllers/Ranking/RankingController.cs` | Parent or student JWT |
| TC-11 | ALFQ-RANK-API-011 | Verify teacher Search returns students from **other** teachers' halaqat (whole tenant) | Main | Teacher JWT with a staff reports claim; students scored in another teacher's halaqa | 1. `POST /Ranking/Search` with `type` `all` and no `halaqaId`. | 1. Response succeeds.<br>2. At least one student from another teacher's halaqa is listed. | ALFQ-RANK-API-010 | `Alforqan/Alforqan.Core/Features/Ranking/Queries/Handlers/GetRankingSearchQueryHandler.cs`, `Alforqan/Alforqan.Core/Features/Ranking/Helpers/RankingSlotQueryHelper.cs` | Teacher + two halaqat |
| TC-12 | ALFQ-RANK-API-012 | Verify `type=hifz` excludes sard-only students | Main | One student with only sard slots; one student with hifz slots | 1. `POST /Ranking/Search` with `type` `hifz`. | 1. Sard-only student is absent.<br>2. Hifz student can appear if they meet min slots. | ALFQ-RANK-API-010 | `Alforqan/Alforqan.Core/Features/Ranking/Helpers/RankingSlotQueryHelper.cs` | Sard-only + hifz students |
| TC-13 | ALFQ-RANK-API-013 | Verify `sessionId` ranks that meeting including absences as 0% and min slots 1 | Main | One HalaqaSession with a scored eval and an absent student with no eval | 1. `POST /Ranking/Search` with that `sessionId`. | 1. Absent student is listed with 0%.<br>2. Students with one slot in that meeting qualify.<br>3. Sard slots are not included. | ALFQ-RANK-API-010 | `Alforqan/Alforqan.Core/Features/Ranking/Helpers/RankingSlotQueryHelper.cs` | One session, mixed attendance |
| TC-14 | ALFQ-RANK-API-014 | Verify hidden students still appear on Search with `showOnLeaderboard` false | Main | Staff JWT; a qualifying student with honor board off | 1. `POST /Ranking/Search` with `type` `all`. | 1. The hidden student is in the list.<br>2. The item has `showOnLeaderboard` false. | ALFQ-RANK-API-010 | `Alforqan/Alforqan.Core/Features/Ranking/Queries/Handlers/GetRankingSearchQueryHandler.cs` | Student with `ShowOnLeaderboard` false |

### Wave 3 — Honor board API

| ID | Key | Case | Type | Preconditions | Steps | Expected | Requires | Related code | Test data |
|----|-----|------|------|---------------|-------|----------|----------|--------------|-----------|
| TC-20 | ALFQ-RANK-API-020 | Verify parent JWT can call HonorBoard (200) and cannot call Search (403) | Main | Parent JWT; Reports module on | 1. `GET /API/V01/Ranking/HonorBoard?period=week`.<br>2. `POST /API/V01/Ranking/Search` with `type` `all`. | 1. HonorBoard returns HTTP 200 with `items` and `myEntries`.<br>2. Search returns HTTP 403. | ALFQ-RANK-API-010 | `Alforqan/Alforqan.Api/Controllers/Ranking/RankingController.cs` | Parent JWT |
| TC-21 | ALFQ-RANK-API-021 | Verify HonorBoard items omit `showOnLeaderboard` false; display ranks stay 1..n | Main | Three qualifying students; rank 1 has honor board off | 1. Build opted-in honor-board items with take 10. | 1. The hidden student is absent from Items.<br>2. First Item `displayRank` is 1 and true `rank` is 2.<br>3. Second Item `displayRank` is 2 and true `rank` is 3. | ALFQ-RANK-API-005 | `Alforqan/Alforqan.Core/Features/Ranking/Helpers/RankingHonorBoardHelper.cs`, `Alforqan/Alforqan.Core/Features/Ranking/Helpers/RankingScoreHelper.cs` | One hidden, two opted-in |
| TC-22 | ALFQ-RANK-API-022 | Verify `myEntries` includes a hidden child with true `rank` | Main | Parent with a hidden qualifying child who is not display-rank 1 | 1. Build myEntries for that child. | 1. myEntries includes the child.<br>2. `qualified` is true.<br>3. `rank` is the true rank among all qualifying students (not omitted). | ALFQ-RANK-API-021 | `Alforqan/Alforqan.Core/Features/Ranking/Helpers/RankingHonorBoardHelper.cs` | Hidden linked child |
| TC-23 | ALFQ-RANK-API-023 | Verify `period=week` and `period=allTime` use different date windows | Main | Server local date known | 1. Parse `week` and `allTime`.<br>2. Resolve ranges for today. | 1. Week start is today minus 6 days and end is today.<br>2. All time has no start or end.<br>3. Invalid period such as `month` is rejected. | - | `Alforqan/Alforqan.Core/Features/Ranking/Helpers/RankingHonorBoardHelper.cs` | Today 2026-08-31 |
| TC-24 | ALFQ-RANK-API-024 | Verify take is capped at 10 | Main | — | 1. Clamp take 0, 3, and 25. | 1. Take 0 becomes 10.<br>2. Take 3 stays 3.<br>3. Take 25 becomes 10. | - | `Alforqan/Alforqan.Core/Features/Ranking/Helpers/RankingHonorBoardHelper.cs` | - |
