# Ranking

## Overview

Ranking endpoints return ordered lists of top students (e.g. by memorization progress or report count) for a specific halaqa or globally across the tenant. The API is gated by the Reports module (`[RequiresModule(TenantModuleKey.Reports)]`). All endpoints require tenant authentication.

Students have `ShowOnLeaderboard` (default **true**). Ranking and honor-board surfaces will respect this flag. Staff **Search** still lists hidden students (`showOnLeaderboard: false`). HonorBoard **Items** omit hidden students. Private **myEntries** still include them.

## Scoring (live slots — not `StudentHalaqaGrades`)

- Each completed hifz eval, revision eval, and decided sard result is one **slot**. An absence with no eval for that session+student is a 0% slot. Excused attendance is skipped.
- `rankScore` = equal average of all slot percents (absences pull the overall average down; they are **not** in hifz/revision/sard type averages).
- Qualify when `slotCount >= 3`. When a single `sessionId` is set, min slots is **1**.
- Sort: average desc, then slot count desc, then last occurred desc.
- Honor-board **display** ranks 1–10 among opted-in students; `Rank` is the true rank among all qualifying students.

## Base route

`/API/V01/Ranking`

## Main operations

- **Search** (staff): `POST Search` – body: `type` (`all` | `hifz` | `revision` | `sard`), `periodStart`, `periodEnd` (inclusive start, exclusive next day), `halaqaId`, `sessionId` (HalaqaSession; ignored for sard; min slots 1), `pageNumber`, `pageSize` (cap 100). Invalid `type` → 400. **No teacher scope** — teachers see the whole tenant. Page after aggregate + rank. Hidden students are included.
- **Halaqa ranking**: `POST Halaqa/{halaqaId}` – thin wrapper over Search (`type=all`, that halaqaId).
- **Global ranking**: `POST Global` – thin wrapper over Search (`type=all`, no halaqa).

Staff Search / wrappers require `ViewMemorizationReportsClaim` **or** `ViewNarrationReportsClaim` (parents/students → 403). Reports module still required.

- **HonorBoard**: `GET HonorBoard?period=week|allTime&take=10` – any authenticated tenant user (no staff reports claim). Invalid `period` → 400. `take` default 10, max 10. Type `all`, no halaqa/session. Min slots **3**. Week = today-6 through today (server local date). All time = no date filter. **Items**: opted-in only, `displayRank` 1..n, `rank` is true rank among all qualifying students. **myEntries**: acting user’s student row (`Students.UserId`) or linked children (`StudentParents` via `Parents.UserId`); empty for staff. Includes hidden / outside top 10. If `slotCount < 3`: `qualified` false and `rank` null.

The tenant dashboard will call HonorBoard for this-week and all-time charts.

## Related documentation

- [Module gating](../features-explanation-docs/module-gating-on-tenant-endpoints-explained.md) – Reports module must be enabled

## Controller

`Alforqan.Api/Controllers/Ranking/RankingController.cs`
