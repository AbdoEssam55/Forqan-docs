# Backend Documentation Index

This folder contains documentation for the Alforqan backend: **platform (host)** APIs, **tenant (Alforqan)** APIs, cross-cutting technical guides, security, architecture, and business/ERD references.

---

## Host system (platform admin)

Platform administrators use the Host API to manage tenants, host users, roles, plans, support plans, feature flag definitions, host branding, and tenant users.

| Doc | Description |
|-----|-------------|
| [features/host-tenants-api.md](features/host-tenants-api.md) | Host Tenants API – tenant CRUD, modules, branding, feature flags, metrics, export |
| [features/host-users.md](features/host-users.md) | Host users CRUD and password management |
| [features/host-roles-and-claims.md](features/host-roles-and-claims.md) | Host roles and claims CRUD and assignment |
| [features/host-plans.md](features/host-plans.md) | Host plans CRUD |
| [features/host-support-plans.md](features/host-support-plans.md) | Host support plans CRUD |
| [features/host-feature-flags-definitions.md](features/host-feature-flags-definitions.md) | Host feature flag definitions and tenant assignment |
| [features/host-branding.md](features/host-branding.md) | Platform (host) branding API |
| [features/host-tenant-users.md](features/host-tenant-users.md) | Manage a tenant's users from host |
| [technical-docs/host-api-overview-guide.md](technical-docs/host-api-overview-guide.md) | Host API route prefix and controller overview |
| [features-explanation-docs/host-identity-system.md](features-explanation-docs/host-identity-system.md) | Host identity, auth, dual Identity architecture |
| [features-explanation-docs/host-support-plans-feature.md](features-explanation-docs/host-support-plans-feature.md) | Support plans purpose and design |
| [features-explanation-docs/host-branding-feature.md](features-explanation-docs/host-branding-feature.md) | Platform vs tenant branding |
| [features-explanation-docs/host-tenant-users-feature.md](features-explanation-docs/host-tenant-users-feature.md) | Why and how host manages tenant users |

---

## Tenant (Alforqan) system

Tenant-scoped APIs for the Qur'an Halaqa product: students, parents, teachers, halaqat, reports, payments, assignments, dashboard, ranking, reference data, tenant config, and tenant auth.

| Doc | Description |
|-----|-------------|
| [features/students.md](features/students.md) | Students CRUD and list |
| [features/parents.md](features/parents.md) | Parents CRUD and list |
| [features/teachers.md](features/teachers.md) | Teachers CRUD and list |
| [features/halaqat.md](features/halaqat.md) | Halaqat CRUD and list |
| [features/memorization-reports.md](features/memorization-reports.md) | Memorization reports CRUD |
| [features/narration-reports.md](features/narration-reports.md) | Narration reports CRUD |
| [features/student-payments.md](features/student-payments.md) | Student payments and allocations |
| [features/student-assignments.md](features/student-assignments.md) | Assignments and submissions |
| [features/student-settings-and-grades.md](features/student-settings-and-grades.md) | Revision settings, daily limits, halaqa grades |
| [features/student-parents.md](features/student-parents.md) | Student–parent linking |
| [features/dashboard.md](features/dashboard.md) | Tenant dashboard |
| [features/ranking.md](features/ranking.md) | Ranking (e.g. top students) |
| [features/reference-data.md](features/reference-data.md) | Surahs, Juzs, Ayahs, StudyStages |
| [features/tenant-config.md](features/tenant-config.md) | TenantConfig for tenant app |
| [features/tenant-auth-and-authorization.md](features/tenant-auth-and-authorization.md) | Tenant login, roles, claims |

Tenant-related platform docs (resolution, branding, module gating, etc.): see [features-explanation-docs/](features-explanation-docs/) and [technical-docs/](technical-docs/) for tenant-resolution, tenant-branding, module-gating, feature-flags, tenant-analytics, tenant-provisioning, tenant-data-export.

---

## Cross-cutting

- **Technical guides**: [technical-docs/](technical-docs/) – tenant resolution, branding, provisioning, module gating, plan limits, feature flags, auth builder pattern, authorization, DB context consistency, localization, validation, controller API, base repository, [hifz-direction-progress.md](technical-docs/hifz-direction-progress.md), etc.
- **Feature explanations**: [features-explanation-docs/](features-explanation-docs/) – architecture and "how it works" for shared and platform features.
- **API guidelines**: [api-guidelines/](api-guidelines/) – e.g. mutation endpoints.
- **Security**: [security/](security/) – testing, checklist, OWASP, race conditions, idempotency, concurrency, payment API guidelines.
- **Architecture**: [architecture/](architecture/) – deployment architecture.
- **DB entities**: [DB entities/](DB%20entities/) – tenant, tenant-module, tenant-metric, tenant-feature-flag.
- **Business**: [business/](business/) – requirements, ERD, plugins.
