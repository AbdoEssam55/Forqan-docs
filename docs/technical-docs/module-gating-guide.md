# Module Gating – How to Add [RequiresModule] to Tenant Endpoints

Short steps to add module gating to a tenant controller or action, and a reference table of endpoint → module.

---

## 1. Add [RequiresModule] to an action

When adding a new tenant API that belongs to a module (Reports, PaymentsBasic), add the attribute on the action (or on the controller if all actions require the same module).

**On a single action:**

```csharp
[HttpGet("{id}/Report")]
[Authorize]
[RequiresModule(TenantModuleKey.Reports)]
public async Task<IActionResult> GetReport(int id)
{
    // ...
}
```

**On a controller (all actions require the same module):**

```csharp
[Route($"{CustomRouter._rule}/[controller]")]
[RequiresModule(TenantModuleKey.PaymentsBasic)]
public class PaymentsController : ControllerBase
{
    // All actions require PaymentsBasic
}
```

- Use `Alforqan.Api.Attributes.RequiresModuleAttribute` and `Alforqan.Data.Enums.Tenants.TenantModuleKey`.
- Core-scoped endpoints (e.g. students CRUD, tenant config, auth) do **not** use the attribute.

---

## 2. Endpoint → module reference table

| Controller / Action | Module | Notes |
|---------------------|--------|--------|
| StudentsController – GetList, GetById, Create, Update, Delete | Core | No attribute |
| StudentsController – GetReport | Reports | `[RequiresModule(TenantModuleKey.Reports)]` |
| TenantConfigController – GetConfig, GetGreeting | None | No module |
| AuthenticationController – all actions | None | No module |
| (Future) ReportsController – any | Reports | Add `[RequiresModule(TenantModuleKey.Reports)]` on controller or each action |
| (Future) PaymentsController – any | PaymentsBasic | Add `[RequiresModule(TenantModuleKey.PaymentsBasic)]` on controller or each action |

When adding new tenant APIs (e.g. reports list, payment create), apply the attribute so tenants without that module receive 403 with a clear message.

---

## 3. Convention

**When adding a new tenant API that belongs to a module (Reports, PaymentsBasic), add `[RequiresModule(TenantModuleKey.X)]` on the action or controller.** No code change is required in `ModuleAuthorizationFilter`; it reads the attribute from endpoint metadata automatically.
