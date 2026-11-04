# Plan-Based Limits Feature

## Overview

Plan-based limits enforce configurable maximum counts (or values) per tenant plan when creating (or updating) tenant-scoped entities. For example, a "Starter" plan might allow at most 100 students, while "Enterprise" is unlimited. Limits are **generic**: the platform does not hard-code entity types; each product defines its own **limit keys** (e.g. `"Students"`, `"Users"`, `"Orders"`) and wires checks before create.

## Why Plan Limits?

- **Monetization**: Plans can be differentiated by how much of a resource (e.g. students, users) a tenant can have.
- **Fair use**: Prevents a single tenant from over-consuming shared infrastructure.
- **Clarity**: Tenants know their plan boundaries; exceeding a limit returns a clear, localized error (e.g. "Your plan limit has been reached").

## Generic Design

- **Limit key**: A string (e.g. `"Students"`, `"Users"`). Any product can define new keys and add corresponding rows in the `PlanLimits` table.
- **Max value**: Stored per plan and key in the Host DB (`PlanLimit` entity: `PlanId`, `LimitKey`, `MaxValue`). No limit row or `MaxValue < 0` means **unlimited**.
- **Where to check**: In the command handler or service that creates the entity (e.g. `StudentService.AddAsync`). The handler gets the max from `IPlanLimitsService.GetLimitMaxAsync(tenantId, limitKey)`, gets the current count (e.g. from the tenant DB), and if `currentCount + 1 > max`, returns 403 or a validation-style error (e.g. `UnprocessableEntity` with a localized message).

## Flow

1. **Create entity (e.g. Student)**  
   API receives a create request in a tenant context.

2. **Resolve tenant and plan**  
   The service uses `ITenantContext` to get the current tenant and `IPlanLimitsService.GetLimitMaxAsync(tenantId, "Students")` to get the plan’s max for that key.

3. **Get current count**  
   The handler/service gets the current count of the resource (e.g. students) from the tenant DB.

4. **Enforce**  
   If a limit is configured (`max != null` and `max >= 0`) and `currentCount + 1 > max`, return an error (e.g. `UnprocessableEntity` with `Plan_LimitExceeded`). Otherwise proceed with create.

5. **Persist**  
   If within limit, create the entity as usual.

## Where to Check

- **Explicit check in handler/service**: Preferred. Each create path that is subject to a limit explicitly calls `IPlanLimitsService` and compares current count to max. This keeps the feature generic (no product-specific metadata in core) and easy to follow.
- **Pipeline behavior**: Alternatively, a MediatR behavior could read a limit key from command metadata and enforce generically; the plan favours the explicit check for clarity.

## References

- **Technical guide**: [plan-limits-guide.md](../technical-docs/plan-limits-guide.md) – how to add PlanLimit rows, call the service in a handler, and add new limit keys.
- **Roadmap**: [multi-tenant-platform-scope-and-roadmap.md](../multi-tenant-platform-scope-and-roadmap.md) section 8.1.
