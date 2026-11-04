# Host Support Plans Feature

## Problem statement

The platform needs to offer support tiers (e.g. Basic, Premium, Enterprise Support) alongside subscription plans. Support plans can have a price, billing period, and a list of rules (e.g. "24/7 support", "Dedicated account manager") that describe what is included. Host admins must be able to create, update, and deactivate these plans so they can be assigned to tenants (e.g. when provisioning or in tenant settings).

## Architecture and design

- **Entity**: `SupportPlan` (Platform.Data) – NameAr/NameEn, Price, CurrencyCode, PeriodType (e.g. Monthly), period labels, IsPopular, SortOrder, IsActive. Stored in Host DB.
- **Rules**: `SupportPlanRule` – SupportPlanId, LabelAr/LabelEn, IsIncluded, SortOrder. One-to-many with SupportPlan.
- **API**: Host Support Plans controller under `/API/V01/Host/SupportPlans` – List, GetById, Create, Update, DeleteById, UnActivate. All require host auth and appropriate claims.
- **Relationship to tenants**: Support plans are host-level catalog items. How they are linked to tenants (e.g. Tenant.SupportPlanId or a separate assignment table) is product-specific; the feature provides the CRUD surface for the catalog.

## How it works

1. Host admin calls Host Support Plans API to list, create, update, or deactivate support plans and their rules (via DTOs that include rules).
2. Services use Host DB (e.g. HostDbContext or host-scoped repos) to persist SupportPlan and SupportPlanRule.
3. Frontend host app (e.g. Support Plans list and form pages) uses host client to call these endpoints. Tenant creation/editing flows can load support plans for dropdown/selection if the product links tenants to support plans.

## Benefits and trade-offs

- **Single catalog**: All support tiers defined in one place; consistent naming and rules across tenants.
- **Localization**: NameAr/NameEn and rule labels allow Arabic/English display.
- **Soft lifecycle**: UnActivate allows hiding a plan without deleting history.

## Related files

- `Platform.Data/Entities/Tenants/SupportPlan.cs`, `SupportPlanRule.cs`
- `Alforqan.Api/Controllers/Host/SupportPlans/HostSupportPlansController.cs`
- `Alforqan.Core/Features/SupportPlans/` (Queries/Commands/Handlers)
- Feature doc: [host-support-plans.md](../features/host-support-plans.md)
