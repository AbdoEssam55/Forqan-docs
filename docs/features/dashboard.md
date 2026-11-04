# Dashboard

## Overview

The tenant dashboard provides aggregate statistics for the logged-in tenant (e.g. counts of students, teachers, halaqat, or recent activity). All endpoints require tenant authentication.

## Base route

`/API/V01/Dashboard`

## Main operations

- **Stats**: `GET Stats` – returns dashboard statistics (e.g. GetDashboardStatsQuery result). Used by the tenant app dashboard page to show summary cards or charts.

Honor boards (this week + all time) will call `GET /API/V01/Ranking/HonorBoard` (see [ranking.md](ranking.md)). Any authenticated tenant user with the Reports module. Not on the public marketing landing.

## Controller

`Alforqan.Api/Controllers/Dashboard/DashboardController.cs`
