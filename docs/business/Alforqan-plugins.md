🧩 Alforqan Plugin & Customization Guidelines

(For Developers & Client Customizations)

Goal:
Allow specific clients/tenants to customize certain behaviors (payments, scoring, notifications, etc.) without modifying Core code, and without breaking updates for all tenants.

Alforqan supports a plugin-ready architecture, but not runtime plugin loading.
This means:

We have extension points (interfaces).

We allow per-tenant overrides through DI & configuration.

We keep Core clean, stable, and updateable.

Custom logic lives in isolated plugin projects like:
Alforqan.Plugins.ClientX.*

1. Principles
✔ 1. One Codebase, Many Behaviors

All tenants share the same backend version.
Differences come from:

Plugin implementations

Per-tenant settings

Strategy pattern

DI selection rules

✔ 2. Never Hard-Code Tenant Checks

❌ BAD

if (tenantCode == "DAR_ALFURQAN")
{
    // special discount logic...
}


✔ GOOD

var rules = _paymentRuleEngine;
await rules.ValidateAsync(payment);

✔ 3. Always use Interfaces for Variable Logic

Put interfaces in neutral layers:

Alforqan.Core

Alforqan.Application

Never inside Infrastructure or WebApi.

Examples of extension-point interfaces:

public interface IPaymentRuleEngine
{
    Task<PaymentValidationResult> ValidateAsync(StudentPayment payment, CancellationToken ct);
}

public interface IStudentPointsStrategy
{
    Task<int> CalculateDeltaAsync(StudentPointContext context, CancellationToken ct);
}

public interface INotificationChannelProvider
{
    Task SendAsync(NotificationMessage msg, CancellationToken ct);
}

✔ 4. Default Implementations Live in Core

Example:

Alforqan.Application
 └─ Payments
     └─ DefaultPaymentRuleEngine.cs

✔ 5. Plugin Implementations Live in Alforqan.Plugins.*

Each client gets a separate plugin module:

src/
  Alforqan.Plugins.ClientA
  Alforqan.Plugins.ClientB


A client may only modify its own plugin folder.

✔ 6. Core must never reference plugins

Plugins may depend on Core.
Core must never depend on plugins.

2. Architecture Overview
+------------------------+
|      Web API           |
|  - Loads tenant        |
|  - Resolves DI         |
+-----------+------------+
            |
            v
+------------------------+
|  Application Layer     |
|  - Uses interfaces     |
|  - Business logic      |
|  - Policies/Rules      |
+-----------+------------+
            |
            v
+------------------------+
|  Core (Domain)         |
|  - Entities            |
|  - Contracts (interfaces) *
|  - Events              |
+-----------+------------+
            |
            v
+------------------------+
|  Infrastructure        |
|  - Default providers   |
|  - EF Core / Email     |
+-----------+------------+
            |
            v
+------------------------+
|  Plugins (Optional)    |
|  - Per-tenant logic *  |
|  - Overrides           |
+------------------------+


Interfaces live in Core/Application.
Default services live in Infrastructure.
Custom implementations live in Plugins.

3. How Plugins Override Behavior

Plugins override behavior using DI.

3.1 Example: Payment Rules

Core Interface (stable):

public interface IPaymentRuleEngine
{
    Task<PaymentValidationResult> ValidateAsync(StudentPayment payment, CancellationToken ct);
}


Default Implementation (used by all tenants):

public class DefaultPaymentRuleEngine : IPaymentRuleEngine
{ ... }


Client Plugin Implementation:

// in Alforqan.Plugins.ClientX
public class ClientXPaymentRuleEngine : IPaymentRuleEngine
{
    public Task<PaymentValidationResult> ValidateAsync(StudentPayment payment, CancellationToken ct)
    {
        // Custom rules only for Client X
    }
}


Plugin DI Registration:

services.AddScoped<IPaymentRuleEngine, ClientXPaymentRuleEngine>();


Tenant-based Resolver (optional):
If needed:

public class TenantBasedPaymentRuleResolver : IPaymentRuleEngine
{
    private readonly ITenantContext _tenant;
    private readonly DefaultPaymentRuleEngine _default;
    private readonly ClientXPaymentRuleEngine _clientX;

    public Task<PaymentValidationResult> ValidateAsync(StudentPayment p, CancellationToken ct)
    {
        return _tenant.Code switch
        {
            "CLIENT_X" => _clientX.ValidateAsync(p, ct),
            _ => _default.ValidateAsync(p, ct),
        };
    }
}

4. Allowed Customization Areas

Clients may override behaviors only via plugins for:

Allowed:

Payment rules

Points calculation

Ranking formulas

Notification channels

Export logic (PDF/Excel)

Specialized validation rules

Custom student scoring policies

Workflow rules (e.g., auto-approve)

Not Allowed:

Editing domain entities

Editing migrations

Changing HTTP controllers

Changing Core or Application layers

Changing multi-tenant architecture

Changing authentication logic

Direct DB schema changes

5. Folder Structure

Recommended repo layout:

/src
   /Alforqan.Core
   /Alforqan.Application
   /Alforqan.Infrastructure
   /Alforqan.WebApi
   /Alforqan.Plugins
        /ClientA
        /ClientB
        /SamplePlugin


Each plugin is a normal .NET project referencing Core and Application.

6. Adding a New Plugin

Create a new folder:

/src/Alforqan.Plugins.ClientX


Add a .csproj referencing:

Alforqan.Core

Alforqan.Application

Alforqan.Infrastructure (optional)

Implement the needed interfaces:

IPaymentRuleEngine
IStudentPointsStrategy
INotificationChannelProvider


Register plugin services in:

ClientXPluginInstaller.cs


Enable plugin via configuration:

"TenantCustomization": "ClientX"

7. Plugin Installer Pattern

In each plugin:

public static class ClientXPluginInstaller
{
    public static IServiceCollection AddClientXPlugins(this IServiceCollection services)
    {
        services.AddScoped<IPaymentRuleEngine, ClientXPaymentRuleEngine>();
        services.AddScoped<IStudentPointsStrategy, ClientXPointsStrategy>();

        return services;
    }
}


Then in WebApi:

if (tenantCustomization == "ClientX")
{
    services.AddClientXPlugins();
}

8. Rules for Codex / Cursor

Codex MUST follow these rules when generating backend code:

✔ Always depend on interfaces, never concrete classes.
✔ Never hard-code tenant checks inside business services.
✔ When implementing logic likely to vary per client, create an extension interface.
✔ Put interfaces in Core/Application.
✔ Put default implementations in Infrastructure.
✔ Put custom implementations in Plugins.
✔ Avoid modifying existing Core/Application code when adding a new rule.
✔ Use DI & resolvers to choose the correct implementation per tenant.
9. Example: Student Points Strategy

Core:

public interface IStudentPointsStrategy
{
    Task<int> CalculateDeltaAsync(StudentPointContext context, CancellationToken ct);
}


Default:

public class DefaultPointsStrategy : IStudentPointsStrategy
{
    public Task<int> CalculateDeltaAsync(StudentPointContext ctx, CancellationToken ct)
        => Task.FromResult(ctx.BaseDelta);
}


Plugin:

public class ClientXPointsStrategy : IStudentPointsStrategy
{
    public Task<int> CalculateDeltaAsync(StudentPointContext ctx, CancellationToken ct)
    {
        // Custom calc
    }
}

10. Updating Plugins When Core Changes

When the core updates:

Interfaces may get new optional parameters (avoid breaking changes).

Plugin implementers update their implementation only.

Core remains untouched.

Other tenants continue using default logic.

11. Summary

The system is plugin-ready.

Plugins = isolated override logic.

Core stays clean and updateable.

Tenants get custom behavior without forking the codebase.

Codex has clear rules to follow for clean development.