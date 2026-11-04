# Plan Limits – Technical Guide

This guide explains how to add plan-based limits, call the limits service in a handler, and introduce new limit keys.

## Data Model

- **PlanLimit** (Host DB): `PlanId`, `LimitKey` (string, e.g. `"Students"`, `"Users"`), `MaxValue` (int). Unique on `(PlanId, LimitKey)`.
- **Unlimited**: No row for that plan/key, or `MaxValue < 0` (e.g. `-1`).

## Adding a PlanLimit Row

1. Insert into `PlanLimits` (Host DB) for the desired `PlanId` and `LimitKey`, with the desired `MaxValue`.
2. You can do this via SQL, a seed script, or an admin UI that manages plans and limits.
3. Example: Plan "Starter" (PlanId = …) has limit key `"Students"` with `MaxValue = 100`.

## Using the Service in a Handler

1. **Inject** `IPlanLimitsService` and `ITenantContext` in the service/handler that creates the entity.
2. **Get tenant**: Use `ITenantContext.CurrentTenant` (or ensure tenant is resolved for the request).
3. **Get max**:  
   `var maxResponse = await _planLimitsService.GetLimitMaxAsync(tenantId, "Students", cancellationToken);`  
   If `!maxResponse.Succeeded`, return the error. Otherwise `max = maxResponse.Data`.
4. **Unlimited**: If `max == null` or `max < 0`, skip the count check.
5. **Current count**: Get the current count of the resource (e.g. students) from the tenant DB (e.g. `baseRepo.GetTableNoTracking().CountAsync()`).
6. **Enforce**: If `currentCount + 1 > max.Value`, return `UnprocessableEntity<T>(localizer[SharedResourcesKeys.Plan.LimitExceeded])` (or 403 if you prefer).
7. **Proceed**: Otherwise call the existing create logic (e.g. `base.AddAsync`).

## Example: Students

- **Limit key**: `"Students"` (constant e.g. in `PlanLimitKeys.Students`).
- **Where**: `StudentService.AddAsync` overrides `BaseService.AddAsync`, gets max via `IPlanLimitsService.GetLimitMaxAsync(tenantId, PlanLimitKeys.Students)`, gets current student count, enforces, then calls `base.AddAsync`.

## Adding a New Limit Key for Your Product

1. **Choose a key**: e.g. `"Users"`, `"Organizations"`.
2. **Add PlanLimit rows**: For each plan that should have a limit, insert a row with that `LimitKey` and the desired `MaxValue`.
3. **Enforce in code**: In the service/handler that creates that entity, call `IPlanLimitsService.GetLimitMaxAsync(tenantId, "YourKey")`, get current count, and return `Plan_LimitExceeded` when over limit.
4. **Localization**: `SharedResourcesKeys.Plan.LimitExceeded` is already localized (en/ar); the message is generic (“Your plan limit has been reached…”).

## Interface Summary

- **IPlanLimitsService.GetLimitMaxAsync(tenantId, limitKey, cancellationToken)**  
  Returns `GenericResponse<int?>`: `Data` is the max value, or `null` for unlimited. `Succeeded == false` on error (e.g. tenant not found).

## References

- **Explanation**: [plan-based-limits-explained.md](../features-explanation-docs/plan-based-limits-explained.md)
- **Entity**: `Alforqan.Data/Entities/Tenants/PlanLimit.cs`
- **Repo**: `ITenantRepo.GetLimitMaxForTenantAsync`
