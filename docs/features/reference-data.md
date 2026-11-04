# Reference Data

## Overview

Reference data endpoints expose read-only Qur'an and study metadata: **Surahs**, **Juzs**, **QuranAyahs**, and **StudyStages**. Used by the tenant app for dropdowns, verse selection, and study-stage assignment. All endpoints are under tenant context and require tenant authentication unless explicitly allowed anonymous.

## Base routes

- `/API/V01/ReferenceData/Surahs`
- `/API/V01/ReferenceData/Juzs`
- `/API/V01/ReferenceData/QuranAyahs`
- `/API/V01/ReferenceData/StudyStages`

## Main operations

Each controller typically exposes:

- **List**: `GET List` or equivalent – all surahs, juzs, ayahs (possibly paged or by surah/juz), or study stages. Response shape matches DTOs used by the frontend (e.g. for select options or verse picker).

Data is usually seeded or loaded from static/store; not tenant-specific content.

## Controllers

`Alforqan.Api/Controllers/ReferenceData/SurahsController.cs`  
`Alforqan.Api/Controllers/ReferenceData/JuzsController.cs`  
`Alforqan.Api/Controllers/ReferenceData/QuranAyahsController.cs`  
`Alforqan.Api/Controllers/ReferenceData/StudyStagesController.cs`
