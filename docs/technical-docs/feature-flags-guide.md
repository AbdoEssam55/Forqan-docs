# Feature Flags – Technical Guide

This guide explains the data model, APIs, and how to add or use feature flags.

## Data Model

- **TenantFeatureFlag** (Host DB): `TenantId`, `FlagKey` (string, max 100), `IsEnabled` (bool), `RolloutPercentage` (int?, 0–100). Unique on `(TenantId, FlagKey)`.
- **Relationship**: Many-to-one with Tenant. Tenant has collection `FeatureFlags`.

## Reading Flags (Tenant App)

1. Call **GET /API/V01/TenantConfig** (tenant-scoped).
2. Response includes **FeatureFlags**: `Dictionary<string, bool>` (key → resolved enabled).
3. Resolved value: if `IsEnabled` is false, always false; if `RolloutPercentage` is null, true; otherwise `GetStableBucket(tenantId, flagKey) < RolloutPercentage`.
4. Use `featureFlags["YourKey"]` to gate UI or routes.

## Host Admin API

### Get tenant feature flags

- **GET** **/API/V01/Host/Tenants/{tenantId}/feature-flags**
- Returns list of `{ flagKey, isEnabled, rolloutPercentage }`.
- Requires host auth and **ViewTenantsClaim**.

### Upsert a flag

- **PUT** **/API/V01/Host/Tenants/{tenantId}/feature-flags/{flagKey}**
- Body: `{ "isEnabled": true, "rolloutPercentage": 50 }` (rolloutPercentage optional).
- Creates or updates the flag for that tenant. Requires **UpdateTenantClaim**.

### Remove a flag

- **DELETE** **/API/V01/Host/Tenants/{tenantId}/feature-flags/{flagKey}**
- Removes the flag for that tenant. Requires **UpdateTenantClaim**.

## Using IFeatureFlagService in Backend

1. **Inject** `IFeatureFlagService` in your handler or service.
2. **Check**:  
   `var response = await _featureFlagService.IsEnabledAsync(tenantId, "YourKey", cancellationToken);`  
   If `response.Succeeded`, `response.Data` is the resolved enabled (true/false). If the flag is not configured, `Data` is false.

## Adding a New Flag Key

1. **Choose a key**: e.g. `"NewCheckout"`, `"BetaReports"`. No platform registration required.
2. **Host admin**: Use PUT to set the flag for tenants (optionally with rollout %).
3. **Tenant app**: Read `featureFlags["YourKey"]` from TenantConfig and gate UI.
4. **Backend** (optional): Call `IFeatureFlagService.IsEnabledAsync(tenantId, "YourKey")` where you need to gate behaviour.

## Optional: [RequiresFeatureFlag] Pattern

To gate entire endpoints by flag (similar to `[RequiresModule]`):

1. Define an attribute `[RequiresFeatureFlag("Key")]` and an authorization filter that reads `ITenantContext` and `IFeatureFlagService.IsEnabledAsync(tenantId, key)`; if false, return 403.
2. Apply the attribute to controller actions that require the flag.
3. Document which endpoints require which flags.

## Rollout Helper

- **FeatureFlagRolloutHelper.GetStableBucket(tenantId, flagKey)** (Alforqan.Shared): Returns 0–99 deterministically for the same tenant and key. Used so rollout % is consistent per tenant.

## References

- **Explanation**: [feature-flags-explained.md](../features-explanation-docs/feature-flags-explained.md)
- **Entity**: `Alforqan.Data/Entities/Tenants/TenantFeatureFlag.cs`
- **Repo**: `ITenantRepo.GetFeatureFlagAsync`, `GetFeatureFlagsByTenantIdAsync`, `UpsertFeatureFlagAsync`, `RemoveFeatureFlagAsync`
