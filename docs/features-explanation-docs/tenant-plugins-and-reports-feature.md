# Tenant Plugins & Student Reports Feature

## Problem Statement

In a multi-tenant system like Alforqan, different tenants (centers/masjids) often require:

1.  **Unique Feature Sets**: Some need advanced reporting, while others only need basic lists.
2.  **Varied Behaviors**: Even for the same feature (e.g., "Student Report"), the _implementation_ needs to differ (e.g., "Simple Text Report" vs "Ministry Compliant Official Transcript").

Hardcoding `if (tenant == "A")` checks throughout the codebase leads to unmaintainable spaghetti code.

## Solution: Modules & Plugins Strategy

We implemented a two-layer customization system:

1.  **Modules (Feature Flags)**: Boolean flags (Enabled/Disabled) that gate access to entire features (API endpoints).
2.  **Plugins (Strategy Pattern)**: A mechanism to swap out service implementations dynamically based on the tenant's configuration.

### Architecture Overview

#### 1. Module Gating

- **Component**: `ModuleAuthorizationFilter` & `[RequiresModule(ModuleKey)]`.
- **Behavior**: Before an action executes, the filter checks if the current tenant has the required module enabled in the Host DB. If not, it returns `403 Forbidden`.
- **Use Case**: Preventing a "Basic Plan" tenant from accessing "Advanced Reporting" endpoints.

#### 2. Plugin Resolution (The "Plugin Factory")

- **Component**: `IPluginFactory`.
- **Mechanism**:
  - Services are registered as **Keyed Services** in DI (e.g., `Key="Default"`, `Key="Formal"`).
  - The `TenantContext` contains a `PluginName` property (e.g., "Formal").
  - `IPluginFactory.GetService<T>()` resolves the implementation that matches the tenant's `PluginName`.
- **Benefit**: The business logic (Handlers/Builders) doesn't need to know _which_ implementation it's using. It just asks for the service.

#### 3. Builder Integration (`PluginBuilder`)

- **Component**: `PluginBuilder<TInput, TResponse, TDbContext>`.
- **Role**: A base class for Builders that need plugin resolution.
- **Method**: `A_ResolvePluginService(out TService service)` encapsulates the try/catch logic of fetching the service from the factory.

---

## Feature: Student Progress Reports

This is the first real business feature using this system.

### Business Need

- **Standard Center**: Needs a simple text summary of memorization.
- **Official Institute**: Needs a formal transcript with specific formatting to meet ministry standards.

### Implementation Details

1.  **Abstraction**: `IReportGeneratorService` defines the contract (`GenerateProgressReportAsync`).
2.  **Implementations**:
    - `DefaultReportGeneratorService`: Returns a simple text report.
    - `FormalReportGeneratorService`: Returns a "Ministry of Education" style transcript.
3.  **Endpoint**: `GET /api/v1/Students/{id}/Report`.
    - Protected by: `[RequiresModule(TenantModuleKey.Reports)]`.
    - Authorized: `[Authorize]`.
4.  **Flow**:
    ```mermaid
    graph TD
        A[Client Request] -->|GET /Report| B(ModuleAuthorizationFilter)
        B -- Module Disabled --> C[403 Forbidden]
        B -- Module Enabled --> D[GetStudentReportQueryHandler]
        D -->|Use Builder| E[GetStudentReportQueryBuilder]
        E -->|Call| F[PluginBuilder.A_ResolvePluginService]
        F -->|Request| G[IPluginFactory]
        G -->|Read TenantContext.PluginName| H{PluginName?}
        H -- "Default" --> I[DefaultReportGeneratorService]
        H -- "Formal" --> J[FormalReportGeneratorService]
        I & J -->|Generate| K[Report String]
        K -->|Return| L[Response]
    ```

### Integration Steps

To add a new report type:

1.  Create `MyNewReportGeneratorService : IReportGeneratorService`.
2.  Register it with a new key (e.g., "Special").
3.  Assign "Special" as the `PluginName` for the tenant in the Host DB.
4.  No changes needed in Controller, Handler, or Builder!

## Security & Isolation

- **Module Check**: Ensures tenants can't access features they haven't paid for/enabled.
- **Tenant Isolation**: The `IPluginFactory` relies on `ITenantContext`, which is securely resolved per request. It never leaks logic between tenants.
