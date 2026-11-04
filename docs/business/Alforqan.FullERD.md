#Host Database
erDiagram

    Tenants {
        uuid TenantId PK
        string Name
        string Code
        string PrimaryDomain
        string ContactName
        string ContactEmail
        string ContactPhone
        string PlanName
        tinyint Status
        datetime CreatedOn
        datetime UpdatedOn
    }

    TenantModules {
        uuid TenantModuleId PK
        uuid TenantId FK
        string ModuleKey
        boolean IsEnabled
        datetime EnabledOn
        datetime DisabledOn
    }

#Tenant Database

erDiagram
Users {
uuid UserId PK
string UserName
string PasswordHash
string Email
string PhoneNumber
string Role
boolean IsActive
datetime LastLoginOn
}

    StudyStages {
        int StageId PK
        string Name
        int SortOrder
    }

    Parents {
        uuid ParentId PK
        uuid UserId FK
        string FullName
        string PhoneNumber
        string SecondaryPhone
        string Address
        string Nationality
        boolean IsActive
    }

    Students {
        uuid StudentId PK
        uuid UserId FK
        string StudentCode
        string FullName
        int StageId FK
        tinyint Gender
        date DateOfBirth
        string Address
        string Nationality
        int ReachedSurahId FK
        tinyint HifzDirection
        boolean IsActive
        tinyint CurrentJuzNumber
    }

    StudentParents {
        uuid Id PK
        uuid StudentId FK
        uuid ParentId FK
        string RelationType
    }

    Teachers {
        uuid TeacherId PK
        uuid UserId FK
        string FullName
        string PhoneNumber
        decimal BaseSalary
        string Address
        string Nationality
        boolean IsActive
    }

    Halaqat {
        uuid HalaqaId PK
        string Name
        tinyint Type
        tinyint Level
        uuid TeacherId FK
        boolean IsActive
    }

    HalaqaStudents {
        uuid Id PK
        uuid HalaqaId FK
        uuid StudentId FK
        date JoinDate
        tinyint Status
        string StatusReason
        tinyint RequestedSeatType
        decimal RequestedDiscountAmount
        tinyint ApprovalStatus
        decimal ApprovedFeeAmount
        uuid ApprovedByUserId FK
        datetime ApprovedOn
        tinyint BillingPeriodType
        datetime BillingStartDate
        datetime BillingEndDate
    }

    HalaqaSchedules {
        uuid Id PK
        uuid HalaqaId FK
        tinyint DayOfWeek
        time StartTime
        time EndTime
        tinyint SessionType
    }

    HalaqaManagers {
        uuid Id PK
        uuid HalaqaId FK
        uuid UserId FK
        boolean IsPrimary
    }

    TeacherAssistants {
        uuid AssistantId PK
        uuid UserId FK
        string FullName
        string PhoneNumber
        boolean IsActive
    }

    HalaqaAssistants {
        uuid Id PK
        uuid HalaqaId FK
        uuid AssistantId FK
        tinyint Role
    }

    HalaqaAssistantPermissions {
        uuid Id PK
        uuid HalaqaAssistantId FK
        string PermissionKey
    }

    Surahs {
        int SurahId PK
        int Number
        string NameArabic
        string NameEnglish
    }

    Juzs {
        int JuzId PK
        int JuzNumber
        int FromSurahId FK
        int FromAyahNumber
        int ToSurahId FK
        int ToAyahNumber
    }

    QuranAyahs {
        int AyahId PK
        int SurahId FK
        int AyahNumber
        int JuzId FK
        int PageNumber
        int HizbNumber
        int RubNumber
        string TextUthmani
        string TextSimple
        int SajdahNumber
    }

    MemorizationReports {
        uuid ReportId PK
        uuid StudentId FK
        uuid TeacherId FK
        uuid HalaqaId FK
        date ReportDate
        tinyint ReportType
        int SurahId FK
        int FromAyah
        int ToAyah
        decimal TotalScore
        string Notes
        uuid CreatedByUserId FK
        uuid AssignmentId FK
    }

    MemorizationAyahDetails {
        uuid Id PK
        uuid ReportId FK
        int SurahId FK
        int AyahNumber
        int HifzMistakeCount
        int TashkeelMistakeCount
        int TajweedMistakeCount
        string Notes
    }

    NarrationReports {
        uuid NarrationId PK
        uuid StudentId FK
        uuid TeacherId FK
        uuid HalaqaId FK
        int JuzId FK
        date NarrationDate
        boolean CountsForTeacherReward
        string Notes
    }

    AttendanceSessions {
        uuid SessionId PK
        uuid HalaqaId FK
        uuid TeacherId FK
        date SessionDate
        datetime PlannedStartUtc
        datetime PlannedEndUtc
        string TimeZoneId
        string MeetingLink
        tinyint LinkVisibleMinutesBeforeStart
        boolean IsCancelled
    }

    AttendanceRecords {
        uuid Id PK
        uuid SessionId FK
        uuid StudentId FK
        tinyint Status
        string Notes
        datetime CheckInTime
        datetime CheckOutTime
    }

    TeacherAttendance {
        uuid Id PK
        uuid TeacherId FK
        date Date
        tinyint Status
        string Notes
        uuid SessionId FK
        datetime CheckInTime
        datetime CheckOutTime
    }

    StudentBehaviorsDaily {
        uuid BehaviorId PK
        uuid StudentId FK
        uuid HalaqaId FK
        uuid SessionId FK
        date BehaviorDate
        tinyint RespectScore
        tinyint CommitmentScore
        tinyint CooperationScore
        tinyint DisciplineScore
        tinyint CalmnessScore
        tinyint MoralityScore
        tinyint OverallLabel
        string Notes
        uuid RecordedByUserId FK
        datetime RecordedOn
    }

    PrayerTypes {
        int PrayerTypeId PK
        string Name
        string Code
        boolean IsOptional
    }

    PrayerAgeGroups {
        uuid AgeGroupId PK
        string Name
        tinyint MinAge
        tinyint MaxAge
        tinyint MinRequiredPrayersPerDay
        boolean IsSubmissionRequired
        int ExtraPrayerScoreDelta
        boolean IsActive
    }

    PrayerAgeGroupRules {
        uuid RuleId PK
        uuid AgeGroupId FK
        int PrayerTypeId FK
        boolean IsRequired
        int DoneScoreDelta
        int MissedScoreDelta
    }

    StudentPrayerSettings {
        uuid SettingId PK
        uuid StudentId FK
        boolean IsSubmissionRequired
        boolean OverrideGlobalRules
        tinyint MinRequiredPrayersPerDay
        int ExtraPrayerScoreDelta
        datetime EffectiveFrom
        datetime EffectiveTo
        uuid SetByUserId FK
        datetime SetOn
    }

    StudentPrayerSettingDetails {
        uuid Id PK
        uuid SettingId FK
        int PrayerTypeId FK
        boolean IsRequired
        int DoneScoreDelta
        int MissedScoreDelta
    }

    StudentDailyPrayers {
        uuid PrayerId PK
        uuid StudentId FK
        date PrayerDate
        tinyint FajrStatus
        tinyint DhuhrStatus
        tinyint AsrStatus
        tinyint MaghribStatus
        tinyint IshaStatus
        string Notes
        uuid RecordedByUserId FK
        datetime RecordedOn
        uuid PrayerSettingId FK
        int TotalRequiredPrayers
        int CompletedRequiredPrayers
        int ExtraPrayersCount
        int PrayerScoreDelta
    }

    Tests {
        uuid TestId PK
        uuid StudentId FK
        uuid TeacherId FK
        uuid HalaqaId FK
        date TestDate
        decimal TotalScore
        string Notes
    }

    TestQuestions {
        uuid QuestionId PK
        uuid TestId FK
        int SurahId FK
        int FromAyah
        int ToAyah
        decimal QuestionScore
        string Notes
    }

    TestQuestionAyahDetails {
        uuid Id PK
        uuid QuestionId FK
        int SurahId FK
        int AyahNumber
        int HifzMistakeCount
        int TashkeelMistakeCount
        int TajweedMistakeCount
        string Notes
    }

    StudentPointTransactions {
        uuid Id PK
        uuid StudentId FK
        datetime TransactionDate
        string SourceType
        uuid SourceId
        int PointsDelta
        string Notes
        uuid CreatedByUserId FK
    }

    TeacherPointTransactions {
        uuid Id PK
        uuid TeacherId FK
        datetime TransactionDate
        string Type
        uuid SourceId
        int PointsDelta
        string Notes
        uuid CreatedByUserId FK
    }

    RewardCategory {
        uuid CategoryId PK
        string Name
        string Description
        boolean IsActive
        int SortOrder
    }

    RewardItem {
        uuid RewardItemId PK
        uuid CategoryId FK
        string Name
        string Description
        int RequiredPoints
        boolean IsCashEquivalent
        boolean IsActive
    }

    Redemption {
        uuid RedemptionId PK
        uuid StudentId FK
        uuid RewardItemId FK
        datetime RequestDate
        tinyint Status
        datetime StatusChangedOn
        uuid StatusChangedByUserId FK
        string Notes
    }

    TeacherPayrolls {
        uuid PayrollId PK
        uuid TeacherId FK
        int Year
        int Month
        decimal BaseSalary
        decimal TotalBonus
        decimal TotalDeduction
        decimal NetSalary
        tinyint Status
        datetime GeneratedOn
        datetime ApprovedOn
        uuid ApprovedByUserId FK
    }

    PayrollBonusDetails {
        uuid Id PK
        uuid PayrollId FK
        string BonusType
        uuid ReferenceId
        decimal Amount
        string Notes
    }

    PayrollDeductionDetails {
        uuid Id PK
        uuid PayrollId FK
        string DeductionType
        uuid ReferenceId
        decimal Amount
        string Notes
    }

    ComplaintTypes {
        uuid ComplaintTypeId PK
        string Name
        string Description
        boolean IsActive
    }

    Complaint {
        uuid ComplaintId PK
        uuid ReporterUserId FK
        string ReporterName
        string ReporterEmail
        string ReporterPhone
        uuid ComplaintTypeId FK
        uuid TargetUserId FK
        uuid TargetStudentId FK
        uuid TargetParentId FK
        uuid TargetHalaqaId FK
        string Title
        string Message
        tinyint Status
        datetime CreatedOn
        uuid ReviewedByUserId FK
        datetime ReviewedOn
        string ReviewNotes
    }

    SystemIssueReports {
        uuid IssueId PK
        uuid ReporterUserId FK
        string ReporterName
        string ReporterEmail
        string IssueType
        tinyint Severity
        string Title
        string Description
        string UrlOrScreen
        tinyint Status
        datetime CreatedOn
        datetime ResolvedOn
        uuid AssignedToUserId FK
        string ResolutionNotes
    }

    LogEntry {
        uuid LogId PK
        uuid UserId FK
        string ActionType
        string Description
        string TargetTable
        string TargetRecordId
        datetime CreatedOn
        string IpAddress
        string UserAgent
    }

    LogTarget {
        uuid LogTargetId PK
        uuid LogId FK
        string TargetEntityName
        string TargetRecordId
    }

    StudentHalaqaGrades {
        uuid GradeId PK
        uuid StudentId FK
        uuid HalaqaId FK
        decimal MemorizationScore
        decimal RevisionScore
        decimal TotalScore
        datetime LastUpdatedOn
        uuid UpdatedByUserId FK
    }

    StudentAssignments {
        uuid AssignmentId PK
        uuid StudentId FK
        uuid HalaqaId FK
        tinyint AssignmentType
        int SurahId FK
        int FromAyah
        int ToAyah
        uuid SessionId FK
        datetime DueDate
        boolean StudentCanSelfSubmit
        uuid CreatedByUserId FK
        datetime CreatedOn
    }

    StudentAssignmentSubmissions {
        uuid SubmissionId PK
        uuid AssignmentId FK
        uuid SubmittedByUserId FK
        tinyint SubmittedByRole
        date RevisionDate
        datetime SubmittedOn
        boolean UsedQuranApp
        boolean IsApprovedByGuardian
        uuid ApprovedByParentId FK
        datetime ApprovedOn
    }

    StudentRevisionSettings {
        uuid SettingId PK
        uuid StudentId FK
        uuid HalaqaId FK
        int DefaultMaxDailySubmissions
        boolean AllowExtraSubmissions
        datetime EffectiveFrom
        datetime EffectiveTo
        uuid SetByUserId FK
        datetime SetOn
    }

    StudentDailyRevisionLimits {
        uuid Id PK
        uuid StudentId FK
        uuid HalaqaId FK
        date RevisionDate
        int AllowedSubmissionsCount
        int UsedSubmissionsCount
        uuid SetByUserId FK
        datetime SetOn
    }

    Notifications {
        uuid NotificationId PK
        string Title
        string Message
        tinyint Type
        uuid HalaqaId FK
        uuid SessionId FK
        datetime CreatedOn
        uuid CreatedByUserId FK
    }

    NotificationRecipients {
        uuid Id PK
        uuid NotificationId FK
        uuid UserId FK
        boolean IsRead
        datetime ReadOn
    }

    TeacherPointSettings {
        uuid Id PK
        uuid TeacherId FK
        boolean IsEnabled
        int PointsPerStudent
        int PointsPerSession
        datetime EffectiveFrom
        datetime EffectiveTo
    }

    Settings {
        uuid SettingId PK
        string Key
        string Value
        string Description
    }

    Roles {
        uuid RoleId PK
        string Name
        string Description
    }

    Permissions {
        uuid PermissionId PK
        string Key
        string Description
    }

    RolePermissions {
        uuid Id PK
        uuid RoleId FK
        uuid PermissionId FK
    }

    UserRoles {
        uuid Id PK
        uuid UserId FK
        uuid RoleId FK
    }

    UserPermissions {
        uuid Id PK
        uuid UserId FK
        uuid PermissionId FK
    }

    CertificateTemplates {
        uuid TemplateId PK
        string Name
        string ExternalDesignReference
        boolean IsDefault
    }

    Certificates {
        uuid CertificateId PK
        uuid StudentId FK
        uuid HalaqaId FK
        uuid SessionId FK
        uuid TemplateId FK
        tinyint CertificateType
        decimal MemorizationScore
        decimal RevisionScore
        string Description
        uuid GeneratedByUserId FK
        datetime GeneratedOn
    }

    OutboundMessages {
        uuid MessageId PK
        string Channel
        string Subject
        string Body
        string TemplateKey
        string RelatedEntityType
        uuid RelatedEntityId
        tinyint Status
        datetime CreatedOn
        uuid CreatedByUserId FK
        datetime SentOn
        string FailureReason
    }

    OutboundMessageRecipients {
        uuid Id PK
        uuid MessageId FK
        uuid UserId FK
        string RecipientType
        string Destination
    }

    StudentPayments {
        uuid PaymentId PK
        uuid PayerUserId FK
        tinyint PayerType
        decimal TotalAmount
        tinyint PaymentMethod
        datetime PaymentDate
        string EvidenceFileUrl
        string Notes
        tinyint Status
        uuid ReviewedByUserId FK
        datetime ReviewedOn
        string ReviewNotes
    }

    StudentPaymentAllocations {
        uuid AllocationId PK
        uuid PaymentId FK
        uuid StudentId FK
        tinyint PeriodType
        date PeriodStartDate
        date PeriodEndDate
        decimal AmountRequested
        decimal DiscountRequestedAmount
        string DiscountReason
        decimal AmountApproved
        decimal DiscountApprovedAmount
        tinyint AllocationStatus
        string AdminComment
    }


        Users ||--o{ Students                  : login_student
    Users ||--o{ Parents                   : login_parent
    Users ||--o{ Teachers                  : login_teacher
    Users ||--o{ TeacherAssistants         : assistant_login
    Users ||--o{ UserRoles                 : user_roles
    Users ||--o{ UserPermissions           : user_perms
    Users ||--o{ NotificationRecipients    : receives
    Users ||--o{ Certificates              : generates
    Users ||--o{ LogEntry                  : logs
    Users ||--o{ Complaint                 : reports
    Users ||--o{ SystemIssueReports        : reports_issue
    Users ||--o{ StudentBehaviorsDaily     : behavior_records
    Users ||--o{ StudentDailyPrayers       : prayer_records
    Users ||--o{ TeacherAttendance         : teacher_attendance
    Users ||--o{ StudentPointTransactions  : student_points_created
    Users ||--o{ TeacherPointTransactions  : teacher_points_created
    Users ||--o{ HalaqaManagers            : manages_halaqa
    Users ||--o{ StudentPrayerSettings     : set_prayer_settings
    Users ||--o{ StudentRevisionSettings   : set_revision_settings
    Users ||--o{ StudentDailyRevisionLimits: set_daily_revision_limits
    Users ||--o{ OutboundMessages          : created_messages
    Users ||--o{ OutboundMessageRecipients : recipient_user
    Users ||--o{ StudentPayments           : payer

    StudyStages ||--o{ Students             : stage

    Students ||--o{ StudentParents          : mapped
    Parents  ||--o{ StudentParents          : mapped

    Surahs   ||--o{ Students                : reached_surah

    Teachers ||--o{ Halaqat                 : leads
    Halaqat  ||--o{ HalaqaStudents          : has
    Students ||--o{ HalaqaStudents          : enrolls

    Halaqat  ||--o{ HalaqaSchedules         : schedule
    Halaqat  ||--o{ HalaqaManagers          : has_managers

    Surahs   ||--o{ QuranAyahs              : has_ayahs
    Juzs     ||--o{ QuranAyahs              : juz_ayahs

    Students ||--o{ MemorizationReports     : mem_reports
    Teachers ||--o{ MemorizationReports     : mem_writes
    Halaqat  ||--o{ MemorizationReports     : mem_context
    Surahs   ||--o{ MemorizationReports     : mem_surah

    StudentAssignments ||--o{ MemorizationReports    : evaluates
    MemorizationReports ||--o{ MemorizationAyahDetails : per_ayah
    Surahs ||--o{ MemorizationAyahDetails            : ayah_surah

    Students ||--o{ NarrationReports        : narration_reports
    Teachers ||--o{ NarrationReports        : narration_writes
    Halaqat  ||--o{ NarrationReports        : narration_context
    Juzs     ||--o{ NarrationReports        : narrated_juz

    Halaqat           ||--o{ AttendanceSessions : holds
    AttendanceSessions ||--o{ AttendanceRecords : session_records
    Students          ||--o{ AttendanceRecords  : attends

    Teachers ||--o{ TeacherAttendance        : attendance

    Students ||--o{ Tests                    : takes
    Teachers ||--o{ Tests                    : conducts
    Halaqat  ||--o{ Tests                    : hosted
    Tests    ||--o{ TestQuestions           : includes
    Surahs   ||--o{ TestQuestions           : q_surah
    TestQuestions ||--o{ TestQuestionAyahDetails : per_ayah
    Surahs       ||--o{ TestQuestionAyahDetails  : ayah_surah_q

    Students ||--o{ StudentPointTransactions : points
    Teachers ||--o{ TeacherPointTransactions : points

    RewardCategory ||--o{ RewardItem         : has
    RewardItem     ||--o{ Redemption         : redeemed
    Students       ||--o{ Redemption         : makes

    Teachers        ||--o{ TeacherPayrolls        : payroll
    TeacherPayrolls ||--o{ PayrollBonusDetails    : bonus
    TeacherPayrolls ||--o{ PayrollDeductionDetails: deduction

    ComplaintTypes ||--o{ Complaint          : type_used
    Students       ||--o{ Complaint          : about_student
    Parents        ||--o{ Complaint          : about_parent
    Halaqat        ||--o{ Complaint          : about_halaqa

    LogEntry ||--o{ LogTarget               : targets

    Students ||--o{ StudentHalaqaGrades     : has_grades
    Halaqat  ||--o{ StudentHalaqaGrades     : grade_context

    Students          ||--o{ StudentAssignments         : has_assignments
    Halaqat           ||--o{ StudentAssignments         : assignment_context
    AttendanceSessions||--o{ StudentAssignments         : next_session
    StudentAssignments||--o{ StudentAssignmentSubmissions : submissions
    Parents           ||--o{ StudentAssignmentSubmissions : approves

    Students ||--o{ StudentRevisionSettings    : revision_settings
    Halaqat  ||--o{ StudentRevisionSettings    : revision_halaqa_settings
    Students ||--o{ StudentDailyRevisionLimits : daily_revision_limits
    Halaqat  ||--o{ StudentDailyRevisionLimits : daily_revision_halaqa_limits

    Halaqat           ||--o{ Notifications          : halaqa_notifications
    AttendanceSessions||--o{ Notifications          : session_notifications
    Notifications     ||--o{ NotificationRecipients : delivered_to

    Roles       ||--o{ RolePermissions        : role_perm
    Permissions ||--o{ RolePermissions        : perm_role
    Roles       ||--o{ UserRoles              : role_users
    Permissions ||--o{ UserPermissions        : perm_users

    Teachers          ||--o{ TeacherAssistants         : assistant_profile
    TeacherAssistants ||--o{ HalaqaAssistants          : assist_in
    Halaqat           ||--o{ HalaqaAssistants          : has_assistants
    HalaqaAssistants  ||--o{ HalaqaAssistantPermissions: assistant_perm

    Students             ||--o{ Certificates        : certs
    Halaqat              ||--o{ Certificates        : halaqa_certs
    AttendanceSessions   ||--o{ Certificates        : session_certs
    CertificateTemplates ||--o{ Certificates        : uses_template

    Students           ||--o{ StudentBehaviorsDaily  : behaviors
    Halaqat            ||--o{ StudentBehaviorsDaily  : halaqa_behaviors
    AttendanceSessions ||--o{ StudentBehaviorsDaily  : session_behaviors

    Students ||--o{ StudentDailyPrayers       : prayers

    StudentPayments ||--o{ StudentPaymentAllocations : splits
    Students        ||--o{ StudentPaymentAllocations : payment_allocations


    Tenants ||--o{ TenantModules : has_modules

erDiagram

    Tenants {
        uuid TenantId PK
        string Name
        string Code
        string PrimaryDomain
        string ContactName
        string ContactEmail
        string ContactPhone
        string PlanName
        tinyint Status
        datetime CreatedOn
        datetime UpdatedOn
    }

    TenantModules {
        uuid TenantModuleId PK
        uuid TenantId FK
        string ModuleKey
        boolean IsEnabled
        datetime EnabledOn
        datetime DisabledOn
    }

    Users {
        uuid UserId PK
        string UserName
        string PasswordHash
        string Email
        string PhoneNumber
        string Role
        boolean IsActive
        datetime LastLoginOn
    }

    StudyStages {
        int StageId PK
        string Name
        int SortOrder
    }

    Parents {
        uuid ParentId PK
        uuid UserId FK
        string FullName
        string PhoneNumber
        string SecondaryPhone
        string Address
        string Nationality
        boolean IsActive
    }

    Students {
        uuid StudentId PK
        uuid UserId FK
        string StudentCode
        string FullName
        int StageId FK
        tinyint Gender
        date DateOfBirth
        string Address
        string Nationality
        int ReachedSurahId FK
        tinyint HifzDirection
        boolean IsActive
        tinyint CurrentJuzNumber
    }

    StudentParents {
        uuid Id PK
        uuid StudentId FK
        uuid ParentId FK
        string RelationType
    }

    Teachers {
        uuid TeacherId PK
        uuid UserId FK
        string FullName
        string PhoneNumber
        decimal BaseSalary
        string Address
        string Nationality
        boolean IsActive
    }

    Halaqat {
        uuid HalaqaId PK
        string Name
        tinyint Type
        tinyint Level
        uuid TeacherId FK
        boolean IsActive
    }

    HalaqaStudents {
        uuid Id PK
        uuid HalaqaId FK
        uuid StudentId FK
        date JoinDate
        tinyint Status
        string StatusReason
        tinyint RequestedSeatType
        decimal RequestedDiscountAmount
        tinyint ApprovalStatus
        decimal ApprovedFeeAmount
        uuid ApprovedByUserId FK
        datetime ApprovedOn
        tinyint BillingPeriodType
        datetime BillingStartDate
        datetime BillingEndDate
    }

    HalaqaSchedules {
        uuid Id PK
        uuid HalaqaId FK
        tinyint DayOfWeek
        time StartTime
        time EndTime
        tinyint SessionType
    }

    HalaqaManagers {
        uuid Id PK
        uuid HalaqaId FK
        uuid UserId FK
        boolean IsPrimary
    }

    TeacherAssistants {
        uuid AssistantId PK
        uuid UserId FK
        string FullName
        string PhoneNumber
        boolean IsActive
    }

    HalaqaAssistants {
        uuid Id PK
        uuid HalaqaId FK
        uuid AssistantId FK
        tinyint Role
    }

    HalaqaAssistantPermissions {
        uuid Id PK
        uuid HalaqaAssistantId FK
        string PermissionKey
    }

    Surahs {
        int SurahId PK
        int Number
        string NameArabic
        string NameEnglish
    }

    Juzs {
        int JuzId PK
        int JuzNumber
        int FromSurahId FK
        int FromAyahNumber
        int ToSurahId FK
        int ToAyahNumber
    }

    QuranAyahs {
        int AyahId PK
        int SurahId FK
        int AyahNumber
        int JuzId FK
        int PageNumber
        int HizbNumber
        int RubNumber
        string TextUthmani
        string TextSimple
        int SajdahNumber
    }

    MemorizationReports {
        uuid ReportId PK
        uuid StudentId FK
        uuid TeacherId FK
        uuid HalaqaId FK
        date ReportDate
        tinyint ReportType
        int SurahId FK
        int FromAyah
        int ToAyah
        decimal TotalScore
        string Notes
        uuid CreatedByUserId FK
        uuid AssignmentId FK
    }

    MemorizationAyahDetails {
        uuid Id PK
        uuid ReportId FK
        int SurahId FK
        int AyahNumber
        int HifzMistakeCount
        int TashkeelMistakeCount
        int TajweedMistakeCount
        string Notes
    }

    NarrationReports {
        uuid NarrationId PK
        uuid StudentId FK
        uuid TeacherId FK
        uuid HalaqaId FK
        int JuzId FK
        date NarrationDate
        boolean CountsForTeacherReward
        string Notes
    }

    AttendanceSessions {
        uuid SessionId PK
        uuid HalaqaId FK
        uuid TeacherId FK
        date SessionDate
        datetime PlannedStartUtc
        datetime PlannedEndUtc
        string TimeZoneId
        string MeetingLink
        tinyint LinkVisibleMinutesBeforeStart
        boolean IsCancelled
    }

    AttendanceRecords {
        uuid Id PK
        uuid SessionId FK
        uuid StudentId FK
        tinyint Status
        string Notes
        datetime CheckInTime
        datetime CheckOutTime
    }

    TeacherAttendance {
        uuid Id PK
        uuid TeacherId FK
        date Date
        tinyint Status
        string Notes
        uuid SessionId FK
        datetime CheckInTime
        datetime CheckOutTime
    }

    StudentBehaviorsDaily {
        uuid BehaviorId PK
        uuid StudentId FK
        uuid HalaqaId FK
        uuid SessionId FK
        date BehaviorDate
        tinyint RespectScore
        tinyint CommitmentScore
        tinyint CooperationScore
        tinyint DisciplineScore
        tinyint CalmnessScore
        tinyint MoralityScore
        tinyint OverallLabel
        string Notes
        uuid RecordedByUserId FK
        datetime RecordedOn
    }

    PrayerTypes {
        int PrayerTypeId PK
        string Name
        string Code
        boolean IsOptional
    }

    PrayerAgeGroups {
        uuid AgeGroupId PK
        string Name
        tinyint MinAge
        tinyint MaxAge
        tinyint MinRequiredPrayersPerDay
        boolean IsSubmissionRequired
        int ExtraPrayerScoreDelta
        boolean IsActive
    }

    PrayerAgeGroupRules {
        uuid RuleId PK
        uuid AgeGroupId FK
        int PrayerTypeId FK
        boolean IsRequired
        int DoneScoreDelta
        int MissedScoreDelta
    }

    StudentPrayerSettings {
        uuid SettingId PK
        uuid StudentId FK
        boolean IsSubmissionRequired
        boolean OverrideGlobalRules
        tinyint MinRequiredPrayersPerDay
        int ExtraPrayerScoreDelta
        datetime EffectiveFrom
        datetime EffectiveTo
        uuid SetByUserId FK
        datetime SetOn
    }

    StudentPrayerSettingDetails {
        uuid Id PK
        uuid SettingId FK
        int PrayerTypeId FK
        boolean IsRequired
        int DoneScoreDelta
        int MissedScoreDelta
    }

    StudentDailyPrayers {
        uuid PrayerId PK
        uuid StudentId FK
        date PrayerDate
        tinyint FajrStatus
        tinyint DhuhrStatus
        tinyint AsrStatus
        tinyint MaghribStatus
        tinyint IshaStatus
        string Notes
        uuid RecordedByUserId FK
        datetime RecordedOn
        uuid PrayerSettingId FK
        int TotalRequiredPrayers
        int CompletedRequiredPrayers
        int ExtraPrayersCount
        int PrayerScoreDelta
    }

    Tests {
        uuid TestId PK
        uuid StudentId FK
        uuid TeacherId FK
        uuid HalaqaId FK
        date TestDate
        decimal TotalScore
        string Notes
    }

    TestQuestions {
        uuid QuestionId PK
        uuid TestId FK
        int SurahId FK
        int FromAyah
        int ToAyah
        decimal QuestionScore
        string Notes
    }

    TestQuestionAyahDetails {
        uuid Id PK
        uuid QuestionId FK
        int SurahId FK
        int AyahNumber
        int HifzMistakeCount
        int TashkeelMistakeCount
        int TajweedMistakeCount
        string Notes
    }

    StudentPointTransactions {
        uuid Id PK
        uuid StudentId FK
        datetime TransactionDate
        string SourceType
        uuid SourceId
        int PointsDelta
        string Notes
        uuid CreatedByUserId FK
    }

    TeacherPointTransactions {
        uuid Id PK
        uuid TeacherId FK
        datetime TransactionDate
        string Type
        uuid SourceId
        int PointsDelta
        string Notes
        uuid CreatedByUserId FK
    }

    RewardCategory {
        uuid CategoryId PK
        string Name
        string Description
        boolean IsActive
        int SortOrder
    }

    RewardItem {
        uuid RewardItemId PK
        uuid CategoryId FK
        string Name
        string Description
        int RequiredPoints
        boolean IsCashEquivalent
        boolean IsActive
    }

    Redemption {
        uuid RedemptionId PK
        uuid StudentId FK
        uuid RewardItemId FK
        datetime RequestDate
        tinyint Status
        datetime StatusChangedOn
        uuid StatusChangedByUserId FK
        string Notes
    }

    TeacherPayrolls {
        uuid PayrollId PK
        uuid TeacherId FK
        int Year
        int Month
        decimal BaseSalary
        decimal TotalBonus
        decimal TotalDeduction
        decimal NetSalary
        tinyint Status
        datetime GeneratedOn
        datetime ApprovedOn
        uuid ApprovedByUserId FK
    }

    PayrollBonusDetails {
        uuid Id PK
        uuid PayrollId FK
        string BonusType
        uuid ReferenceId
        decimal Amount
        string Notes
    }

    PayrollDeductionDetails {
        uuid Id PK
        uuid PayrollId FK
        string DeductionType
        uuid ReferenceId
        decimal Amount
        string Notes
    }

    ComplaintTypes {
        uuid ComplaintTypeId PK
        string Name
        string Description
        boolean IsActive
    }

    Complaint {
        uuid ComplaintId PK
        uuid ReporterUserId FK
        string ReporterName
        string ReporterEmail
        string ReporterPhone
        uuid ComplaintTypeId FK
        uuid TargetUserId FK
        uuid TargetStudentId FK
        uuid TargetParentId FK
        uuid TargetHalaqaId FK
        string Title
        string Message
        tinyint Status
        datetime CreatedOn
        uuid ReviewedByUserId FK
        datetime ReviewedOn
        string ReviewNotes
    }

    SystemIssueReports {
        uuid IssueId PK
        uuid ReporterUserId FK
        string ReporterName
        string ReporterEmail
        string IssueType
        tinyint Severity
        string Title
        string Description
        string UrlOrScreen
        tinyint Status
        datetime CreatedOn
        datetime ResolvedOn
        uuid AssignedToUserId FK
        string ResolutionNotes
    }

    LogEntry {
        uuid LogId PK
        uuid UserId FK
        string ActionType
        string Description
        string TargetTable
        string TargetRecordId
        datetime CreatedOn
        string IpAddress
        string UserAgent
    }

    LogTarget {
        uuid LogTargetId PK
        uuid LogId FK
        string TargetEntityName
        string TargetRecordId
    }

    StudentHalaqaGrades {
        uuid GradeId PK
        uuid StudentId FK
        uuid HalaqaId FK
        decimal MemorizationScore
        decimal RevisionScore
        decimal TotalScore
        datetime LastUpdatedOn
        uuid UpdatedByUserId FK
    }

    StudentAssignments {
        uuid AssignmentId PK
        uuid StudentId FK
        uuid HalaqaId FK
        tinyint AssignmentType
        int SurahId FK
        int FromAyah
        int ToAyah
        uuid SessionId FK
        datetime DueDate
        boolean StudentCanSelfSubmit
        uuid CreatedByUserId FK
        datetime CreatedOn
    }

    StudentAssignmentSubmissions {
        uuid SubmissionId PK
        uuid AssignmentId FK
        uuid SubmittedByUserId FK
        tinyint SubmittedByRole
        date RevisionDate
        datetime SubmittedOn
        boolean UsedQuranApp
        boolean IsApprovedByGuardian
        uuid ApprovedByParentId FK
        datetime ApprovedOn
    }

    StudentRevisionSettings {
        uuid SettingId PK
        uuid StudentId FK
        uuid HalaqaId FK
        int DefaultMaxDailySubmissions
        boolean AllowExtraSubmissions
        datetime EffectiveFrom
        datetime EffectiveTo
        uuid SetByUserId FK
        datetime SetOn
    }

    StudentDailyRevisionLimits {
        uuid Id PK
        uuid StudentId FK
        uuid HalaqaId FK
        date RevisionDate
        int AllowedSubmissionsCount
        int UsedSubmissionsCount
        uuid SetByUserId FK
        datetime SetOn
    }

    Notifications {
        uuid NotificationId PK
        string Title
        string Message
        tinyint Type
        uuid HalaqaId FK
        uuid SessionId FK
        datetime CreatedOn
        uuid CreatedByUserId FK
    }

    NotificationRecipients {
        uuid Id PK
        uuid NotificationId FK
        uuid UserId FK
        boolean IsRead
        datetime ReadOn
    }

    TeacherPointSettings {
        uuid Id PK
        uuid TeacherId FK
        boolean IsEnabled
        int PointsPerStudent
        int PointsPerSession
        datetime EffectiveFrom
        datetime EffectiveTo
    }

    Settings {
        uuid SettingId PK
        string Key
        string Value
        string Description
    }

    Roles {
        uuid RoleId PK
        string Name
        string Description
    }

    Permissions {
        uuid PermissionId PK
        string Key
        string Description
    }

    RolePermissions {
        uuid Id PK
        uuid RoleId FK
        uuid PermissionId FK
    }

    UserRoles {
        uuid Id PK
        uuid UserId FK
        uuid RoleId FK
    }

    UserPermissions {
        uuid Id PK
        uuid UserId FK
        uuid PermissionId FK
    }

    CertificateTemplates {
        uuid TemplateId PK
        string Name
        string ExternalDesignReference
        boolean IsDefault
    }

    Certificates {
        uuid CertificateId PK
        uuid StudentId FK
        uuid HalaqaId FK
        uuid SessionId FK
        uuid TemplateId FK
        tinyint CertificateType
        decimal MemorizationScore
        decimal RevisionScore
        string Description
        uuid GeneratedByUserId FK
        datetime GeneratedOn
    }

    OutboundMessages {
        uuid MessageId PK
        string Channel
        string Subject
        string Body
        string TemplateKey
        string RelatedEntityType
        uuid RelatedEntityId
        tinyint Status
        datetime CreatedOn
        uuid CreatedByUserId FK
        datetime SentOn
        string FailureReason
    }

    OutboundMessageRecipients {
        uuid Id PK
        uuid MessageId FK
        uuid UserId FK
        string RecipientType
        string Destination
    }

    StudentPayments {
        uuid PaymentId PK
        uuid PayerUserId FK
        tinyint PayerType
        decimal TotalAmount
        tinyint PaymentMethod
        datetime PaymentDate
        string EvidenceFileUrl
        string Notes
        tinyint Status
        uuid ReviewedByUserId FK
        datetime ReviewedOn
        string ReviewNotes
    }

    StudentPaymentAllocations {
        uuid AllocationId PK
        uuid PaymentId FK
        uuid StudentId FK
        tinyint PeriodType
        date PeriodStartDate
        date PeriodEndDate
        decimal AmountRequested
        decimal DiscountRequestedAmount
        string DiscountReason
        decimal AmountApproved
        decimal DiscountApprovedAmount
        tinyint AllocationStatus
        string AdminComment
    }


        Users ||--o{ Students                  : login_student
    Users ||--o{ Parents                   : login_parent
    Users ||--o{ Teachers                  : login_teacher
    Users ||--o{ TeacherAssistants         : assistant_login
    Users ||--o{ UserRoles                 : user_roles
    Users ||--o{ UserPermissions           : user_perms
    Users ||--o{ NotificationRecipients    : receives
    Users ||--o{ Certificates              : generates
    Users ||--o{ LogEntry                  : logs
    Users ||--o{ Complaint                 : reports
    Users ||--o{ SystemIssueReports        : reports_issue
    Users ||--o{ StudentBehaviorsDaily     : behavior_records
    Users ||--o{ StudentDailyPrayers       : prayer_records
    Users ||--o{ TeacherAttendance         : teacher_attendance
    Users ||--o{ StudentPointTransactions  : student_points_created
    Users ||--o{ TeacherPointTransactions  : teacher_points_created
    Users ||--o{ HalaqaManagers            : manages_halaqa
    Users ||--o{ StudentPrayerSettings     : set_prayer_settings
    Users ||--o{ StudentRevisionSettings   : set_revision_settings
    Users ||--o{ StudentDailyRevisionLimits: set_daily_revision_limits
    Users ||--o{ OutboundMessages          : created_messages
    Users ||--o{ OutboundMessageRecipients : recipient_user
    Users ||--o{ StudentPayments           : payer

    StudyStages ||--o{ Students             : stage

    Students ||--o{ StudentParents          : mapped
    Parents  ||--o{ StudentParents          : mapped

    Surahs   ||--o{ Students                : reached_surah

    Teachers ||--o{ Halaqat                 : leads
    Halaqat  ||--o{ HalaqaStudents          : has
    Students ||--o{ HalaqaStudents          : enrolls

    Halaqat  ||--o{ HalaqaSchedules         : schedule
    Halaqat  ||--o{ HalaqaManagers          : has_managers

    Surahs   ||--o{ QuranAyahs              : has_ayahs
    Juzs     ||--o{ QuranAyahs              : juz_ayahs

    Students ||--o{ MemorizationReports     : mem_reports
    Teachers ||--o{ MemorizationReports     : mem_writes
    Halaqat  ||--o{ MemorizationReports     : mem_context
    Surahs   ||--o{ MemorizationReports     : mem_surah

    StudentAssignments ||--o{ MemorizationReports    : evaluates
    MemorizationReports ||--o{ MemorizationAyahDetails : per_ayah
    Surahs ||--o{ MemorizationAyahDetails            : ayah_surah

    Students ||--o{ NarrationReports        : narration_reports
    Teachers ||--o{ NarrationReports        : narration_writes
    Halaqat  ||--o{ NarrationReports        : narration_context
    Juzs     ||--o{ NarrationReports        : narrated_juz

    Halaqat           ||--o{ AttendanceSessions : holds
    AttendanceSessions ||--o{ AttendanceRecords : session_records
    Students          ||--o{ AttendanceRecords  : attends

    Teachers ||--o{ TeacherAttendance        : attendance

    Students ||--o{ Tests                    : takes
    Teachers ||--o{ Tests                    : conducts
    Halaqat  ||--o{ Tests                    : hosted
    Tests    ||--o{ TestQuestions           : includes
    Surahs   ||--o{ TestQuestions           : q_surah
    TestQuestions ||--o{ TestQuestionAyahDetails : per_ayah
    Surahs       ||--o{ TestQuestionAyahDetails  : ayah_surah_q

    Students ||--o{ StudentPointTransactions : points
    Teachers ||--o{ TeacherPointTransactions : points

    RewardCategory ||--o{ RewardItem         : has
    RewardItem     ||--o{ Redemption         : redeemed
    Students       ||--o{ Redemption         : makes

    Teachers        ||--o{ TeacherPayrolls        : payroll
    TeacherPayrolls ||--o{ PayrollBonusDetails    : bonus
    TeacherPayrolls ||--o{ PayrollDeductionDetails: deduction

    ComplaintTypes ||--o{ Complaint          : type_used
    Students       ||--o{ Complaint          : about_student
    Parents        ||--o{ Complaint          : about_parent
    Halaqat        ||--o{ Complaint          : about_halaqa

    LogEntry ||--o{ LogTarget               : targets

    Students ||--o{ StudentHalaqaGrades     : has_grades
    Halaqat  ||--o{ StudentHalaqaGrades     : grade_context

    Students          ||--o{ StudentAssignments         : has_assignments
    Halaqat           ||--o{ StudentAssignments         : assignment_context
    AttendanceSessions||--o{ StudentAssignments         : next_session
    StudentAssignments||--o{ StudentAssignmentSubmissions : submissions
    Parents           ||--o{ StudentAssignmentSubmissions : approves

    Students ||--o{ StudentRevisionSettings    : revision_settings
    Halaqat  ||--o{ StudentRevisionSettings    : revision_halaqa_settings
    Students ||--o{ StudentDailyRevisionLimits : daily_revision_limits
    Halaqat  ||--o{ StudentDailyRevisionLimits : daily_revision_halaqa_limits

    Halaqat           ||--o{ Notifications          : halaqa_notifications
    AttendanceSessions||--o{ Notifications          : session_notifications
    Notifications     ||--o{ NotificationRecipients : delivered_to

    Roles       ||--o{ RolePermissions        : role_perm
    Permissions ||--o{ RolePermissions        : perm_role
    Roles       ||--o{ UserRoles              : role_users
    Permissions ||--o{ UserPermissions        : perm_users

    Teachers          ||--o{ TeacherAssistants         : assistant_profile
    TeacherAssistants ||--o{ HalaqaAssistants          : assist_in
    Halaqat           ||--o{ HalaqaAssistants          : has_assistants
    HalaqaAssistants  ||--o{ HalaqaAssistantPermissions: assistant_perm

    Students             ||--o{ Certificates        : certs
    Halaqat              ||--o{ Certificates        : halaqa_certs
    AttendanceSessions   ||--o{ Certificates        : session_certs
    CertificateTemplates ||--o{ Certificates        : uses_template

    Students           ||--o{ StudentBehaviorsDaily  : behaviors
    Halaqat            ||--o{ StudentBehaviorsDaily  : halaqa_behaviors
    AttendanceSessions ||--o{ StudentBehaviorsDaily  : session_behaviors

    Students ||--o{ StudentDailyPrayers       : prayers

    StudentPayments ||--o{ StudentPaymentAllocations : splits
    Students        ||--o{ StudentPaymentAllocations : payment_allocations


    Tenants ||--o{ TenantModules : has_modules
