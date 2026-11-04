# Student Settings and Grades

## Overview

Three controllers manage student-related settings and grades: **StudentRevisionSettings** (revision/muraaja'ah settings per student or global), **StudentDailyRevisionLimits** (daily revision limits), and **StudentHalaqaGrades** (grades or progress in a halaqa). All require tenant authentication and appropriate tenant claims.

## Base routes

- `/API/V01/StudentRevisionSettings`
- `/API/V01/StudentDailyRevisionLimits`
- `/API/V01/StudentHalaqaGrades`

## Main operations (each controller)

Standard CRUD pattern: `GET List`, `POST List`, `GET {id}`, `POST create`, `PUT {id}`, `DELETE {id}` (exact routes follow `CustomRouter.BaseRouting`). Use cases:

- **StudentRevisionSettings**: Configure how revision is tracked or required for a student.
- **StudentDailyRevisionLimits**: Set or read daily revision limits (e.g. max pages per day).
- **StudentHalaqaGrades**: Record or read grade/progress for a student in a specific halaqa.

See each controller for policy and route constants.

## Controllers

`Alforqan.Api/Controllers/StudentRevisionSettings/StudentRevisionSettingsController.cs`  
`Alforqan.Api/Controllers/StudentDailyRevisionLimits/StudentDailyRevisionLimitsController.cs`  
`Alforqan.Api/Controllers/StudentHalaqaGrades/StudentHalaqaGradesController.cs`
