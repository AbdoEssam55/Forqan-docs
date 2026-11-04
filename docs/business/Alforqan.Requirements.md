# 📄 Document A – Phase 1 Requirements

## 1. Objective of Phase 1

Build the **first working version** of the Qur’an Halaqa System that focuses on:

- Recording and tracking **memorization reports**
- Recording and tracking **revision (muraaja’ah) reports**
- Recording and tracking **narration (khatmah / juz) reports**
- Supporting **basic student–guardian–teacher structure and halaqat**
- Providing **simple rankings of top students**
- Implementing a **basic cash payment & discount request flow**

All of this must be implemented **according to the final ERD** we built, so that the system can be expanded later without redesigning the database.

---

## 2. Actors & Roles in Phase 1

Phase 1 will work with these roles:

1. **Admin**

   - Manages tenants (if multi-institute).
   - Manages users (students, guardians, teachers).
   - Manages halaqat and assignments.
   - Reviews and approves **payments** and discounts.
   - Can see global reports/rankings.

2. **Teacher**

   - Sees only the halaqat assigned to him/her.
   - Views students in each halaqa.
   - Creates **memorization/revision assignments**.
   - Records **memorization reports**.
   - Records **narration reports**.
   - Can see basic performance of students in his/her halaqat.

3. **Student**

   - Has a login (username/password).
   - Is assigned to one or more halaqat.
   - Can view own memorization/revision/narration history.
   - Can submit **revision assignments** (where allowed).
   - Can submit **cash payment request** for own fees.

4. **Guardian (Parent)**

   - Has a login linked to phone number.
   - Linked to one or multiple students via **StudentCode**.
   - Can view children’s reports and payments.
   - Can submit **revision completion** on behalf of a child.
   - Can submit **cash payment request** for one or more children.

_(More advanced roles like manager, assistant, etc. are future-phase in terms of functionality, even if they exist in the ERD.)_

---

## 3. Core Domain Concepts in Phase 1

### 3.1 Tenants & Users

- **Tenants**: Allow the system to host multiple institutes in the future.  
   For Phase 1, you can operate with one tenant, but the structure is in place (`Tenants` table).
- **Users**: Generic accounts with:
  - `UserName`, `PasswordHash`, `Email`, `PhoneNumber`, `Role` (Student / Parent / Teacher / Admin).
  - Linked to specific profiles:
    - `Students`
    - `Parents`
    - `Teachers`

The **linking** is:

- `Users → Students`
- `Users → Parents`
- `Users → Teachers`

Each profile carries personal data (name, phone, address, nationality, etc.).

---

### 3.2 Students, Parents and Linking

- **Students**
  - Keep personal info:
    - Name, gender, date of birth, address, nationality.
    - Study stage (`StudyStages`).
    - Qur’an progress: `ReachedSurahId`, `CurrentJuzNumber`, `HifzDirection` (from left/right).
  - Each student has a unique `StudentCode` to link guardians.
- **Parents (Guardians)**
  - Each parent has:
    - A `User` account.
    - Phone number, secondary phone, address, nationality.
- **Student–Parent Relationship**
  - Many-to-many via `StudentParents`:
    - One student can have **0, 1 or many** parents.
    - One parent can have **multiple** children.
  - When a parent registers:
    - Enters each child’s `StudentCode`.
    - System links them in `StudentParents`.

---

### 3.3 Teachers & Halaqat

- **Teachers**
  - Personal info + base salary (used later in payroll system).
- **Halaqat**
  - Each halaqa has:
    - Name
    - Type (memorization / narration / mixed)
    - Level
    - Main teacher (`TeacherId`)
    - Active flag
- **Halaqa Students**
  - `HalaqaStudents` links students to halaqat.
  - Contains status (active, finished, etc.).
  - Already prepared for seat type and billing info, but in phase 1 you’ll only:
    - Use basic join (student is in halaqa)
    - Optionally set initial billing period information if needed.

_(More advanced manager/assistant relations to Halaqa exist in ERD but can be ignored functionally in phase 1.)_

---

### 3.4 Qur’an Structure

To support reports and tests:

- **Surahs**
  - List of surahs (id, number, Arabic name, English name).
- **Juzs**
  - Each juz with:
    - Start surah + ayah
    - End surah + ayah.
- **QuranAyahs**
  - Each ayah:
    - Surah, ayah number, juz id, page, hizb, rub, text, etc.
  - Used for:
    - Per-ayah mistake tracking.
    - Future detailed features.

In Phase 1, you will at least use:

- `Surahs`
- `Juzs`
- Possibly `QuranAyahs` if you implement per-ayah mistake details (recommended).

---

## 4. Phase 1 – Functional Modules

### 4.1 Memorization Assignments & Reports

#### 4.1.1 Assignments

- **StudentAssignments**
  - The teacher defines **next-session homework** for a student in a halaqa:
    - `AssignmentType`: Memorization (one range) or Revision (one or more non-overlapping master ranges)
    - From surah/ayah → to surah/ayah (stored in `StudentAssignmentRanges`; envelope on parent row)
    - `Status`: Open (at most one per student+halaqa+type), Done, Cancelled
    - Target session (`SessionId`) and optional `AssignedFromSessionId` (session where teacher set homework after evaluation)
    - Due date (defaults to next scheduled session of the halaqa)
  - After evaluation outcome, teacher sets homework for the **next** lesson; that range locks the next session evaluation unless the teacher confirms a rewrite.

This allows the teacher to plan what the student should memorize or revise for the **next lesson**.

#### 4.1.2 Memorization Reports

- **MemorizationReports**
  - Created by the teacher after hearing the student.
  - Contains:
    - Student, teacher, halaqa
    - Date
    - Report type (memorization / revision if used)
    - Surah, from ayah, to ayah
    - Total score
    - Notes
    - `AssignmentId` (link to the assignment that the report evaluates)
- **MemorizationAyahDetails**
  - Optional but recommended from Phase 1:
    - Per-ayah mistake counts:
      - Hifz mistakes
      - Tashkeel mistakes
      - Tajweed mistakes
    - Notes per ayah.
  - Phase 1 logic:
    - Teacher inputs mistakes per ayah.
    - System aggregates to compute **TotalScore** for the report.

#### 4.1.3 Reporting & Ranking

- For each student:
  - Show history of memorization reports (date, surah, range, score).
- For each halaqa:
  - Show students and their average/total memorization scores for:
    - Today
    - This week
    - This month
    - Custom date range
- Global:
  - Top student in each halaqa for a period.
  - Top student across all halaqat for a period.

---

### 4.2 Revision Assignments & Submissions

#### 4.2.1 Teacher-side: create revision assignments

- Using the same `StudentAssignments` table (`AssignmentType = Revision`), the teacher:
  - Specifies surah(s) and ayah range that must be revised.
  - Defines due date.
  - Sets whether student is allowed to submit by himself or only through guardian.

#### 4.2.2 Student/Guardian-side: submit revision

- **StudentAssignmentSubmissions**
  - Student or guardian submits:
    - Which assignment
    - Date of revision
    - Whether a Qur’an app was used (`UsedQuranApp`)
    - Optional notes in the future, if needed.
  - Guardian can approve (`IsApprovedByGuardian`, `ApprovedByParentId`).

#### 4.2.3 Daily limitations

- The ERD already supports:
  - `StudentRevisionSettings`
  - `StudentDailyRevisionLimits`

For Phase 1, you can implement a **simple rule**:

- One revision submission per day per student per halaqa.
- Optional: Teacher can increase allowed submissions using the `StudentDailyRevisionLimits` or `StudentRevisionSettings` if you want to implement this now.

#### 4.2.4 Impact on student score

- Revision status affects the student’s **daily/overall score**.
- Exact scoring rules can be simple in Phase 1 (e.g., full points if done, zero if not).

---

### 4.3 Narration (Khatmah / Juz) Reports

- **NarrationReports**
  - For when a student completes a **Juz**.
  - Teacher records:
    - Student, teacher, halaqa
    - Juz (via `JuzId`)
    - Date
    - Whether it should count for teacher reward (`CountsForTeacherReward`)
    - Notes
  - **Does not** affect student academic grade directly (by design).
  - Used later in payroll / rewards for teachers.

In Phase 1:

- Just store the narration information.
- Optional simple view:
  - History of narrated parts for a student.
  - Count of juz completed by each student.

---

### 4.4 Basic Ranking & Analytics

Using:

- `MemorizationReports`
- `StudentAssignments` + `StudentAssignmentSubmissions`
- Optionally `StudentHalaqaGrades` if you want to materialize scores

Phase 1 must allow:

1. **Per Halaqa:**

   - List students with:
     - Total memorization score for a period
     - Total revision score for a period (if tracked separately)
   - Identify **top students** and **weak students** (e.g., below a threshold).

2. **Global:**

   - Top students across all halaqat (based on memorization+revision score) for a period.

The formulas can be relatively simple in phase 1 (you can refine them in future phases).

---

### 4.5 Basic Cash Payment & Discount Requests

This is **only** for manual / cash payments in Phase 1.

#### 4.5.1 Payment capture

- **StudentPayments**
  - A payment record:
    - `PayerUserId` (student or guardian)
    - `PayerType` (Student / Guardian)
    - `TotalAmount`
    - `PaymentMethod` = Cash (in this phase)
    - `PaymentDate`
    - `EvidenceFileUrl` (uploaded receipt photo / PDF)
    - `Notes`
    - `Status` (Pending, Approved, Rejected, etc.)
    - `ReviewedByUserId`, `ReviewedOn`, `ReviewNotes`
- **StudentPaymentAllocations**
  - Splits one payment across:
    - One or more students
    - Period type (month, term, custom from–to date)
    - For each allocation:
      - `AmountRequested`
      - `DiscountRequestedAmount` + `DiscountReason`
      - `AmountApproved`
      - `DiscountApprovedAmount`
      - `AllocationStatus` (Pending / Approved / Rejected)
      - `AdminComment`

#### 4.5.2 Admin payment review screen

For each payment (and each allocation):

- Show:
  - How much this student **should have paid until now** (simplify rule in phase 1).
  - How much was actually paid before.
  - How much is being paid now.
  - Requested discount details.
- Actions:
  - Approve / Reject allocation.
  - Approve payment (if all allocations are resolved).
  - Optionally partially approve (some allocations approved, others rejected).

No online gateway integration in Phase 1; only manual/cash with uploaded proof.

---

### 4.6 Out-of-scope in Phase 1 (but present in ERD)

The following are **not implemented functionally** in Phase 1, although their tables may already exist:

- Attendance system (students & teachers).
- Behaviour evaluation.
- Daily prayer tracking & prayer settings.
- Points system and reward store.
- Payroll calculations and detailed HR management.
- Complaints and system issue reporting.
- Notifications, WhatsApp/email sending (OutboundMessages).
- Teacher assistants and per-halaqa assistant permissions.
- Advanced roles & permissions UI.
- Certificates generation.
- Global reports for all halaqat, weak students reports, etc. (only simple ranking in Phase 1).

These will be part of **future phases**.

---

# 📄 Document B – Future Phases Requirements (Vision)

This document collects **all other requirements** that are not committed in Phase 1, but which we designed for in the ERD and discussions.

You can later slice these into Phase 2, Phase 3, etc.

---

## 1. Accounts & Roles (Advanced)

- Full role & permission system:
  - `Roles`, `Permissions`, `RolePermissions`, `UserRoles`, `UserPermissions`.
  - Example:
    - Two teachers can have different permissions (e.g., one can see payroll, the other cannot).
    - Assistants have custom permissions per halaqa.
- Manager vs Admin distinction:
  - Admin: global control over tenant (institute).
  - Manager: responsible for specific halaqat, can:
    - Approve halaqa creation.
    - Approve/decline student registration in halaqa.
    - Review complaints for their halaqat.
    - Adjust revision/prayer settings for students in their halaqat.

---

## 2. Halaqa Management & Scheduling (Full)

Beyond the basics in Phase 1:

- **Advanced Halaqa Management**
  - Waiting lists.
  - Auto-accept rules.
  - Maximum capacity, age limits, stage/level constraints.
- **Schedules**
  - `HalaqaSchedules` with:
    - Days of week.
    - Start and end times.
    - Session type (memorization / revision / recitation).
  - Main dashboard showing:
    - Halaqa status: in progress, about to start, finished.
    - Next upcoming sessions.
- **Halaqa Managers & Assistants**
  - `HalaqaManagers`: multiple managers per halaqa, with primary manager flag.
  - `TeacherAssistants`, `HalaqaAssistants`, `HalaqaAssistantPermissions`:
    - Per-halaqa assistant roles (assistant / co-teacher / supervisor).
    - Fine-grained permissions: can take attendance, can add reports, can edit assignments, etc.

---

## 3. Attendance System

### 3.1 Student Attendance

- **AttendanceSessions**
  - Each actual lesson instance:
    - Halaqa, teacher, date, planned start/end, timezone, meeting link, visibility window.
  - Could be used for online / physical sessions.
- **AttendanceRecords**
  - For each student and session:
    - Status: Present, Absent, Late, Excused.
    - Check-in and check-out timestamps.
    - Notes.
- **Reports**
  - Per halaqa:
    - Count of present/absent/late/excused.
    - Attendance percentage per student.
  - Global:
    - Compare halaqat in attendance.
    - Trends over time.

### 3.2 Teacher Attendance

- **TeacherAttendance**
  - Tracks daily and per-session teacher attendance.
  - Used later in payroll calculations:
    - Deductions for absence or lateness.
    - Bonuses for consistency.

---

## 4. Behaviour Evaluation

- **StudentBehaviorsDaily**
  - For each lesson/session:
    - Scores for:
      - Respect
      - Commitment
      - Cooperation
      - Discipline
      - Calmness
      - Morality
    - Overall label (Excellent, Very Good, Good, Needs Improvement).
    - Notes and who recorded them.
- **Reports**
  - Track the behavior trajectory of a student over time.
  - Identify students needing extra support.

---

## 5. Daily Prayer Tracking

### 5.1 Prayer Types & Age Rules

- **PrayerTypes**
  - Fajr, Dhuhr, Asr, Maghrib, Isha, Sunnah etc.
- **PrayerAgeGroups & Rules**
  - For each age-range (e.g., 5–10, 10–25, >25):
    - `PrayerAgeGroups`: define min/max age and how many required prayers per day.
    - `PrayerAgeGroupRules`: define for each prayer:
      - Required or not.
      - Score increase if done.
      - Score change if missed.
    - Extra prayer (e.g., Sunnah) reward.
- Concept:
  - Admin defines **global default rules** by age.

### 5.2 Per-student Prayer Settings

- **StudentPrayerSettings** and **StudentPrayerSettingDetails**
  - Teacher or manager can override global settings for specific student:
    - For example: “This student only needs to submit Asr and Maghrib.”
    - Customize done/missed score deltas.
    - Allow no prayer submission required.

### 5.3 Daily Submissions

- **StudentDailyPrayers**
  - Student submits once per day:
    - Status for each prayer (Done / Missed / Other statuses).
    - Linked to which setting was applied.
    - Total required prayers, completed required prayers, extra prayers, and prayer score delta.
    - Notes and who recorded them (student or guardian).
- **Reports**
  - Show commitment to prayer per student, per halaqa, global.
  - “Top students in prayer adherence”.

---

## 6. Points System & Reward Store

### 6.1 Student Points

- **StudentPointTransactions**
  - Every event that changes student points:
    - SourceType (e.g., “FinishJuz”, “MonthlyAchievement”, “ExtraGoodBehaviour”, etc.).
    - SourceId (link to report/test/etc.).
    - PointsDelta.
    - Notes and who created it.
- Automatic point rules (later configurable):
  - Finishing a Juz.
  - Finishing a revision plan.
  - Full attendance in a period.
  - Winning competitions.

### 6.2 Teacher Points

- **TeacherPointTransactions**
  - For rewards/penalties:
    - narration completions.
    - competition outcomes.
    - other achievements.
- **TeacherPointSettings**
  - Enable/disable points per teacher.
  - Points per student/session.
  - Effective date ranges.

### 6.3 Reward Store

- **RewardCategory** and **RewardItem**
  - Categories of rewards (e.g., Books, Gifts, Cash Equivalent).
  - Items with required points and active flag.
- **Redemption**
  - Student requests to redeem an item.
  - Status (Pending, Approved, Rejected).
  - Admin reviews and approves, logs who changed status and when.

---

## 7. Payroll & HR

- **TeacherPayrolls**
  - Monthly payroll record per teacher:
    - Base salary
    - Total bonuses
    - Total deductions
    - Net salary
    - Status (Draft, Approved)
    - Generated/approved dates & who approved.
- **PayrollBonusDetails**, **PayrollDeductionDetails**
  - Explain exactly why the amount was added/deducted:
    - BonusType/DeductionType.
    - ReferenceId (link to request/report).
    - Amount and notes.
- Integration with:
  - Teacher attendance.
  - Narration reports (`CountsForTeacherReward`).
  - Behaviour and activity metrics.
  - Complaints (negative impact).

---

## 8. Complaints & System Issues

### 8.1 Complaints

- **ComplaintTypes** and **Complaint**
  - Any role (student, parent, teacher) can submit complaint:
    - Against a teacher, parent, student, halaqa, or the system.
  - Complaints have:
    - Type, target, status (Pending, Accepted, Rejected, UnderReview).
    - Review notes and reviewer.
- Integration:
  - Negative complaint decisions can:
    - Affect teacher points (`TeacherPointTransactions`).
    - Affect payroll deductions (`PayrollDeductionDetails`).

### 8.2 System Issue Reports

- **SystemIssueReports**
  - Used to report bugs, design problems, technical issues.
  - Can be anonymous or linked to user.
  - Can be assigned to admins/developers, track status, resolution, and notes.

---

## 9. Notifications & Messaging

### 9.1 System Notifications

- **Notifications** and **NotificationRecipients**
  - For system events like:
    - Halaqa created / approved.
    - Halaqa about to start / finished.
    - New assignment posted.
    - Payment status changed.
  - Per-recipient read/unread status.

### 9.2 Outbound Email / WhatsApp / SMS

- **OutboundMessages** and **OutboundMessageRecipients**
  - Support sending:
    - Memorization reports for a specific period to selected students or parents.
    - Payment reminders.
    - Announcements.
- Channels:
  - Email, WhatsApp, possibly SMS (depending on integration).
- Messages can be tied to entities:
  - e.g., `RelatedEntityType = 'MemorizationReport'`, `RelatedEntityId = ReportId`.

---

## 10. Certificates

- **CertificateTemplates**
  - Templates designed e.g. in Canva.
  - External reference ID for template.
- **Certificates**
  - Generated for:
    - Narration completions.
    - Course completion.
    - Excellence awards.
  - Linked to:
    - Student, halaqa, session, template.
    - Scores and description.
  - Printable / downloadable PDFs.

---

## 11. Advanced Analytics & Dashboards

Using all the collected data, future phases can include:

- Per-halaqa performance dashboard:
  - Average grades.
  - Distribution of strong/weak students.
- Global halaqa comparison dashboard:
  - Rank halaqat by performance and attendance.
- Weak student reports:
  - Students below threshold with full context (grades, mistakes, notes).
- Absence dashboards:
  - Attendance stats per halaqa and global.
- Activity and logging dashboards:
  - From `LogEntry` and `LogTarget`:
    - Who did what, when, and on which record.

---

## 12. Future Payment Extensions

- Add payment methods:
  - Bank transfer, online gateway (Stripe, PayTabs, HyperPay, etc.).
- Automatic reconciliation with payment providers.
- Auto-blocking or warning for unpaid periods.

---

## 13. Future E-Learning & Streaming

- Extend `AttendanceSessions`:
  - Strong integration with Zoom/Google Meet/other platforms.
  - Join links, automatic attendance tracking.
- Video/audio streaming:
  - For live halaqat.
  - Possibly recorded sessions.
- Student content library:
  - Recorded lessons, attached materials.
