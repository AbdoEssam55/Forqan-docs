# Shared Authorization Services Feature

## Problem Statement

The initial implementation of authorization features (role management, user roles/claims management) had significant code duplication between tenant and host handlers. Both tenant and host handlers contained nearly identical business logic for:

- Role CRUD operations (create, update, delete, get by ID, list)
- User roles management (get user roles, modify user roles)
- User claims management (get user claims, modify user claims)

This duplication violated the DRY (Don't Repeat Yourself) principle and made maintenance difficult. Any changes to business rules required updates in multiple places, increasing the risk of inconsistencies and bugs.

## Architecture and Design Decisions

### Why Services Accept Managers/Contexts as Parameters

The shared services (`IAuthorizationRoleService` and `IAuthorizationUserService`) accept `RoleManager<IdentityRole>`, `UserManager<AppUser>`, and `DbContext` as parameters rather than injecting them directly. This design decision was made for several reasons:

1. **Context Flexibility**: Handlers need to pass different managers/contexts based on whether they're operating in tenant or host context:
   - Tenant handlers pass `RoleManager<IdentityRole>` from tenant `AppDbContext`
   - Host handlers pass `RoleManager<IdentityRole>` from `HostIdentityDbContext` via `IHostRoleManagerFactory`
   - Tenant handlers pass `UserManager<AppUser>` from tenant context via `IUserManagerFactory.GetTenantUserManager(appDbContext)`
   - Host handlers pass `UserManager<AppUser>` from host context via `IUserManagerFactory.GetHostUserManager(hostIdentityDbContext)`

2. **Avoiding DI Conflicts**: If services injected managers directly, we'd face DI conflicts because the container can only provide one instance of `RoleManager<IdentityRole>` or `UserManager<AppUser>`. By passing them as parameters, handlers explicitly control which instance is used.

3. **Testability**: Services become pure business logic functions that are easier to unit test by passing mock managers/contexts.

### How Host-Specific Logic Flexibility Works

The architecture allows handlers to add host-specific logic in three ways:

#### 1. Before Service Calls (Pre-validation)

Handlers can add validation or business rules before calling the shared service:

```csharp
public async Task<GenericResponse<IdentityRole>> Handle(
    CreateHostRoleCommand request,
    CancellationToken cancellationToken)
{
    // Host-specific validation
    if (!request.Name!.StartsWith("Host_", StringComparison.OrdinalIgnoreCase))
    {
        return BadRequest<IdentityRole>("Host roles must start with 'Host_'.");
    }

    var roleManager = hostRoleManagerFactory.CreateHostRoleManager(hostIdentityDbContext);
    return await roleService.CreateRoleAsync(request.Name!, roleManager, cancellationToken);
}
```

#### 2. After Service Calls (Post-processing)

Handlers can add logic after successful service calls:

```csharp
var result = await roleService.CreateRoleAsync(...);
if (result.Succeeded)
{
    // Host-specific audit logging, notifications, etc.
    await _auditService.LogHostRoleCreation(...);
}
return result;
```

#### 3. Inside Services via Parameters (Shared Variations)

For truly shared logic with minor variations, services can accept optional parameters:

```csharp
// Example (not currently implemented, but possible)
public async Task<GenericResponse<IdentityRole>> CreateRoleAsync(
    string name,
    RoleManager<IdentityRole> roleManager,
    bool isHostRole = false,
    CancellationToken cancellationToken = default)
{
    // Shared logic...
    
    // Host-specific rule
    if (isHostRole && name.Equals("Admin", StringComparison.OrdinalIgnoreCase))
        return BadRequest<IdentityRole>("Host role 'Admin' is not allowed.");
    
    // Continue with shared logic...
}
```

**Note**: We prefer keeping host-specific logic in handlers rather than adding many boolean flags to services, as it keeps services cleaner and more maintainable.

### Transaction Management Approach

The `AuthorizationUserService` accepts `DbContext` as a parameter for transaction management:

- **Tenant handlers** pass `AppDbContext`
- **Host handlers** pass `HostIdentityDbContext`

The service manages transactions internally:

```csharp
var transaction = await dbContext.Database.BeginTransactionAsync(cancellationToken);
try
{
    // Operations...
    await transaction.CommitAsync(cancellationToken);
}
catch (Exception ex)
{
    await transaction.RollbackAsync(cancellationToken);
    return BadRequest<string>(localizer[SharedResourcesKeys.Fail]);
}
```

This ensures atomicity for operations that modify multiple tables (e.g., removing all user roles and adding new ones).

## How It Works

### Flow Diagram: Handler → Service → Manager

```
┌─────────────────────────────────────────────────────────────┐
│                    Controllers (API Layer)                   │
│  AuthorizationController  │  HostRolesController             │
└──────────────┬────────────┴──────────────┬──────────────────┘
               │                            │
               ▼                            ▼
┌──────────────────────────┐  ┌──────────────────────────────┐
│  Tenant Handlers         │  │  Host Handlers                │
│  (MediatR)              │  │  (MediatR)                    │
│  - AuthorizationCommand  │  │  - HostAuthorizationCommand  │
│  - AuthorizationQuery    │  │  - HostAuthorizationQuery    │
└──────────────┬───────────┘  └──────────────┬────────────────┘
               │                              │
               │  Can add host-specific      │
               │  logic before/after         │
               │                              │
               └──────────────┬──────────────┘
                              ▼
        ┌─────────────────────────────────────┐
        │   Shared Services (Business Logic) │
        │  - AuthorizationRoleService       │
        │  - AuthorizationUserService       │
        └─────────────────────────────────────┘
                              │
                              │ Accepts managers/contexts
                              │ as parameters
                              ▼
        ┌─────────────────────────────────────┐
        │   Factories & Managers               │
        │  - IHostRoleManagerFactory           │
        │  - IUserManagerFactory               │
        │  - RoleManager<IdentityRole>         │
        │  - UserManager<AppUser>              │
        └─────────────────────────────────────┘
```

### Example Flow: Creating a Host Role

1. **Request arrives**: `POST /API/V01/Host/Roles/CreateRole`
2. **Controller** receives request and sends `CreateHostRoleCommand` via MediatR
3. **HostAuthorizationCommandHandler** receives command:
   - Optionally adds host-specific validation
   - Gets `RoleManager` from `IHostRoleManagerFactory.CreateHostRoleManager(hostIdentityDbContext)`
   - Calls `roleService.CreateRoleAsync(name, roleManager, ct)`
4. **AuthorizationRoleService** executes shared business logic:
   - Checks for duplicate role names (case-insensitive)
   - Creates role via `roleManager.CreateAsync()`
   - Returns `GenericResponse<IdentityRole>`
5. **Handler** receives response:
   - Optionally adds post-processing (audit, notifications)
   - Returns response to controller
6. **Controller** returns response to client

## Integration with Existing Systems

### How Tenant Handlers Use Services

Tenant handlers inject the shared services and pass tenant-specific managers:

```csharp
public class AuthorizationCommandHandler : GenericResponseHandler
{
    private readonly IAuthorizationRoleService roleService;
    private readonly RoleManager<IdentityRole> roleManager; // Tenant role manager
    private readonly IUserManagerFactory userManagerFactory;

    public async Task<GenericResponse<IdentityRole>> Handle(
        CreateRoleCommand request,
        CancellationToken cancellationToken)
    {
        // Pass tenant role manager to shared service
        return await roleService.CreateRoleAsync(
            request.Name!,
            roleManager, // Tenant context
            cancellationToken);
    }
}
```

### How Host Handlers Use Services

Host handlers inject the shared services and pass host-specific managers:

```csharp
public class HostAuthorizationCommandHandler : GenericResponseHandler
{
    private readonly IAuthorizationRoleService roleService;
    private readonly IHostRoleManagerFactory hostRoleManagerFactory;
    private readonly IUserManagerFactory userManagerFactory;

    public async Task<GenericResponse<IdentityRole>> Handle(
        CreateHostRoleCommand request,
        CancellationToken cancellationToken)
    {
        // Get host role manager
        var roleManager = hostRoleManagerFactory.CreateHostRoleManager(hostIdentityDbContext);
        
        // Pass host role manager to shared service
        return await roleService.CreateRoleAsync(
            request.Name!,
            roleManager, // Host context
            cancellationToken);
    }
}
```

### How Factories Fit Into the Pattern

Factories (`IHostRoleManagerFactory`, `IUserManagerFactory`) resolve the correct manager instances:

- **IHostRoleManagerFactory**: Creates `RoleManager<IdentityRole>` using the provided `HostIdentityDbContext` parameter
- **IUserManagerFactory**: Provides `GetHostUserManager(HostIdentityDbContext)` and `GetTenantUserManager(AppDbContext)` methods that accept `DbContext` parameters

These factories are injected at the handler level, allowing handlers to request the appropriate manager for their context.

## Benefits and Trade-offs

### Benefits

1. **DRY Principle**: Business logic is centralized in services, eliminating duplication
2. **Maintainability**: Changes to business rules only need to be made in one place
3. **Consistency**: Tenant and host operations follow the same business rules
4. **Testability**: Services can be unit tested independently with mock managers
5. **Flexibility**: Handlers can add context-specific logic before/after service calls
6. **Separation of Concerns**: Business logic is separated from infrastructure concerns (managers, contexts)

### Trade-offs

1. **DTO Mapping Overhead**: Handlers need to map between Core DTOs and Shared DTOs (necessary due to circular dependency constraints)
2. **Parameter Passing**: Services accept more parameters than if they injected dependencies directly (but this is intentional for flexibility)
3. **Learning Curve**: Developers need to understand when to add logic in handlers vs services

### When to Add Logic in Handlers vs Services

**Add logic in handlers when:**
- Logic is specific to tenant or host context
- Logic is a one-off exception to shared rules
- Logic involves context-specific infrastructure (e.g., different audit systems)

**Add logic in services when:**
- Logic applies to both tenant and host contexts
- Logic is core business rule (e.g., duplicate name checking, protected role validation)
- Logic needs to be consistent across all contexts

## Related Files

### Service Layer
- `Alforqan.Service/Abstractions/Identity/Authorization/IAuthorizationRoleService.cs`
- `Alforqan.Service/Abstractions/Identity/Authorization/IAuthorizationUserService.cs`
- `Alforqan.Service/Implementation/Identity/Authorization/AuthorizationRoleService.cs`
- `Alforqan.Service/Implementation/Identity/Authorization/AuthorizationUserService.cs`

### Handlers
- `Alforqan.Core/Features/Identity/Authorization/Commands/Handlers/AuthorizationCommandHandler.cs`
- `Alforqan.Core/Features/Identity/Authorization/Queries/Handlers/AuthorizationQueryHandler.cs`
- `Alforqan.Core/Features/Identity/HostAuthorization/Commands/Handlers/HostAuthorizationCommandHandler.cs`
- `Alforqan.Core/Features/Identity/HostAuthorization/Queries/Handlers/HostAuthorizationQueryHandler.cs`

### Factories
- `Alforqan.Service/Abstractions/Identity/IHostRoleManagerFactory.cs`
- `Alforqan.Service/Implementation/Identity/HostRoleManagerFactory.cs`
- `Alforqan.Service/Abstractions/Identity/IUserManagerFactory.cs`
- `Alforqan.Service/Implementation/Identity/UserManagerFactory.cs`

### DTOs
- `Alforqan.Shared/Dtos/Authorization/GetUserRolesQueryResponseDto.cs`
- `Alforqan.Shared/Dtos/Authorization/GetUserClaimsQueryResponseDto.cs`
- `Alforqan.Shared/Dtos/Authorization/ModifyUserRolesCommandDto.cs`

### Documentation
- `docs/technical-docs/authorization-services-guide.md` - How to use the services
- `docs/features-explanation-docs/host-identity-system.md` - Host identity system overview

