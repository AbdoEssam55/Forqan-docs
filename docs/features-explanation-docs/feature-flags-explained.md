# Feature Flags Feature

## Overview

Per-tenant feature flags allow turning features on or off (and optionally rolling out by percentage) **in addition to** modules. Modules remain the primary "capability" switch (e.g. Reports, Payments); feature flags are for gradual rollouts, A/B testing, or product-specific toggles. The design is **generic**: the platform does not hard-code flag names; each product defines its own **flag keys** (e.g. `"NewCheckout"`, `"BetaDashboard"`, `"AdvancedReports"`).

## Why Feature Flags?

- **Gradual rollout**: Enable a feature for a percentage of tenants (e.g. 50%) using deterministic hash so the same tenant always gets the same result.
- **A/B or experiments**: Toggle behaviour per tenant without changing modules.
- **Product-specific toggles**: Any product (Alforqan, e-commerce, clinical) can define its own keys and gate UI or API by flag.

## Generic Design

- **Flag key**: A string (e.g. `"NewCheckout"`, `"BetaDashboard"`). Any product defines new keys; no platform enum.
- **Storage**: Host DB (`TenantFeatureFlag` entity: `TenantId`, `FlagKey`, `IsEnabled`, `RolloutPercentage`). One row per tenant per flag. Unique on `(TenantId, FlagKey)`.
- **Rollout %**: Optional 0–100. When set, "enabled" is resolved by a deterministic hash of `(tenantId + flagKey)` so the same tenant consistently gets the same result. If `RolloutPercentage` is null, use only `IsEnabled`.
- **Where to read**: Tenant app reads `GET /API/V01/TenantConfig`; response includes `FeatureFlags` (dictionary of key → enabled). Backend handlers can use `IFeatureFlagService.IsEnabledAsync(tenantId, flagKey)` when only a single flag is needed.

## Flow

1. **Host admin** sets a flag for a tenant (e.g. `PUT Host/Tenants/{tenantId}/feature-flags/NewCheckout` with `{ "isEnabled": true, "rolloutPercentage": 50 }`).
2. **Tenant config** is loaded with `FeatureFlags` included; each flag’s resolved value is computed (if rollout % is set, use hash bucket &lt; percentage; else use `IsEnabled`).
3. **Tenant app** reads `featureFlags` from config and gates UI (e.g. show "New checkout" only when `featureFlags["NewCheckout"] === true`).
4. **Backend** (optional): A handler can call `IFeatureFlagService.IsEnabledAsync(tenantId, "NewCheckout")` to gate API behaviour.

## Where to Check

- **Tenant config**: Preferred for frontend. Call `GET /API/V01/TenantConfig` once per session; use `response.featureFlags[key]` to show/hide features.
- **IFeatureFlagService**: Use in backend when you need to check a single flag (e.g. in a MediatR handler or service) without loading full config.
- **Optional**: A `[RequiresFeatureFlag("Key")]` attribute and filter could be added later (see technical guide) for endpoint-level gating.

## References

- **Technical guide**: [feature-flags-guide.md](../technical-docs/feature-flags-guide.md) – entity, API, how to add a new flag key.
- **DB entity**: [tenant-feature-flag-entity.md](../DB%20entities/tenant-feature-flag-entity.md).
- **Roadmap**: [multi-tenant-platform-scope-and-roadmap.md](../../../docs/multi-tenant-platform-scope-and-roadmap.md) section 8.6.
