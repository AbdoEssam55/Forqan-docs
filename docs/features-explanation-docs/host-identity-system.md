# Host Identity System

## Problem Statement

In a multi-tenant system, platform administrators (SystemAdmin, DesignAdmin, etc.) need to manage tenants and system-wide configurations. These users should:

- Exist independently of any tenant database
- Be able to log in even if tenant databases are unavailable
- Have clear separation from tenant-level users (Admin, Teacher, Student, Parent)
- Use the same authentication mechanisms but with distinct authorization scopes

The original implementation stored SystemAdmin in a tenant database, which created operational risks: if that tenant database was down, platform administrators couldn't log in to manage the system.

## Architecture Overview

The Host Identity System implements a **dual Identity architecture**:

- **Host Identity** (Host DB): Platform administrators (SystemAdmin, DesignAdmin, etc.)
  - Stored in `HostIdentityDbContext` using the Host database connection
  - Uses `HostScheme` JWT authentication
  - Endpoints under `/API/V01/Host/*` prefix

- **Tenant Identity** (Tenant DBs): Tenant users (Admin, Teacher, Student, Parent)
  - Stored in `AppDbContext` using tenant-specific database connections
  - Uses `TenantScheme` JWT authentication (default)
  - Endpoints under standard `/API/V01/*` routes

### Database Separation

```
┌─────────────────────────────────────┐
│         Host Database                │
├─────────────────────────────────────┤
│  • Tenants (metadata)                │
│  • TenantModules                     │
│  • AspNetUsers (host users)          │
│  • AspNetRoles (host roles)          │
│  • RefreshTokens (host tokens)       │
└─────────────────────────────────────┘

┌─────────────────────────────────────┐
│      Tenant Database (per tenant)    │
├─────────────────────────────────────┤
│  • Students, Teachers, Parents       │
│  • Halaqat, Reports                  │
│  • AspNetUsers (tenant users)       │
│  • AspNetRoles (tenant roles)        │
│  • RefreshTokens (tenant tokens)     │
└─────────────────────────────────────┘
```

## Design Decisions

### Why AddIdentityCore Instead of AddIdentity

**Problem**: Calling `AddIdentity<AppUser, IdentityRole>()` twice (once for tenant, once for host) causes Dependency Injection conflicts. The last registration wins, making it impossible to resolve the correct `UserManager<AppUser>` for each context.

**Solution**: Use `AddIdentityCore<AppUser>()` for Host Identity. This registers:
- `UserManager<AppUser>` (scoped)
- `SignInManager<AppUser>` (scoped)
- Token providers

But does NOT register `RoleManager<IdentityRole>` by default, avoiding conflicts with the tenant `RoleManager`.

**Trade-off**: `RoleManager` must be created manually when needed (e.g., during seeding), but this provides explicit control over which `RoleManager` instance is used.

### Why Unified `/API/V01/Host/*` Prefix

**Problem**: Host endpoints need to bypass tenant resolution middleware. Without a unified prefix, each host endpoint would need individual checks.

**Solution**: All host operations use the unified prefix `/API/V01/Host/*`:
- `/API/V01/Host/Auth/Login` - Host authentication
- `/API/V01/Host/Tenants/*` - Tenant management
- Future host endpoints follow the same pattern

**Benefits**:
- Single check in `TenantResolutionMiddleware` to skip tenant resolution
- Clear API organization
- Easy to identify host vs tenant endpoints

### Why Dual JWT Schemes (HostScheme vs TenantScheme)

**Problem**: Tokens from host login and tenant login need different validation contexts and authorization scopes.

**Solution**: Two separate JWT authentication schemes:
- `"HostScheme"`: Validates tokens for host users, used by host endpoints
- `"TenantScheme"`: Validates tokens for tenant users, default scheme for tenant endpoints

**Implementation**:
```csharp
[Authorize(AuthenticationSchemes = "HostScheme", Roles = "SystemAdmin")]
public class TenantsController : ControllerBase { }
```

**Benefits**:
- Clear separation of authentication contexts
- Can apply different token validation rules per scheme if needed
- Explicit authorization requirements

### Why UserManagerFactory Pattern

**Problem**: Services need to resolve the correct `UserManager<AppUser>` (host vs tenant) but DI container can only provide one instance when using the same type.

**Solution**: `IUserManagerFactory` pattern that constructs `UserManager` instances on-demand using the appropriate `DbContext`:

```csharp
public interface IUserManagerFactory
{
    UserManager<AppUser> GetHostUserManager(HostIdentityDbContext dbContext);
    UserManager<AppUser> GetTenantUserManager(AppDbContext dbContext);
}
```

**Implementation**: Factory accepts `DbContext` as parameter and constructs `UserManager` instances using:
- `HostIdentityDbContext` for host operations
- `AppDbContext` for tenant operations

**Key Design Decision**: Factories accept `DbContext` as parameter (not resolve from DI) to ensure transaction consistency. This guarantees that `UserManager` always uses the same `DbContext` instance as the builder's transaction, preventing silent transaction breaks when multiple `DbContext` instances exist in the same scope.

**Benefits**:
- Explicit control over which `UserManager` is used
- No DI conflicts
- Services can request the correct instance based on context
- **Transaction safety**: UserManager always uses the same DbContext as the caller's transaction

## How It Works

### Request Flow for Host Endpoints

```
1. Request arrives: POST /API/V01/Host/Auth/Login
   ↓
2. TenantResolutionMiddleware checks path
   - Path starts with "/API/V01/Host" → Skip tenant resolution
   - CurrentTenant remains null
   ↓
3. HostAuthController receives request
   - [AllowAnonymous] allows unauthenticated access
   ↓
4. HostLoginCommandBuilder processes login
   - Uses IUserManagerFactory.GetHostUserManager(hostIdentityDbContext)
   - Queries HostIdentityDbContext for user
   - Validates password against Host DB
   ↓
5. HostCustomAuthenticationService generates token
   - Creates JWT with HostScheme claims
   - Stores refresh token in HostIdentityDbContext
   ↓
6. Response returns token to client
```

### Authentication Flow

```
Host User Login:
┌─────────────┐
│   Client    │
└──────┬──────┘
       │ POST /API/V01/Host/Auth/Login
       ↓
┌──────────────────────────┐
│ HostAuthController       │
│ (AllowAnonymous)         │
└──────┬───────────────────┘
       │
       ↓
┌──────────────────────────┐
│ HostLoginCommandBuilder  │
│ - GetHostUserManager(ctx)│
│ - Query HostIdentityDB   │
│ - Validate password      │
└──────┬───────────────────┘
       │
       ↓
┌──────────────────────────┐
│ HostCustomAuthService    │
│ - Generate JWT           │
│ - Create RefreshToken    │
└──────┬───────────────────┘
       │
       ↓
┌──────────────────────────┐
│ Token Response            │
│ - accessToken (HostScheme)│
│ - refreshToken            │
└──────────────────────────┘

Tenant User Login:
┌─────────────┐
│   Client    │
└──────┬──────┘
       │ POST /API/V01/Auth/Login
       ↓
┌──────────────────────────┐
│ TenantResolution         │
│ - Resolve tenant         │
│ - Set CurrentTenant      │
└──────┬───────────────────┘
       │
       ↓
┌──────────────────────────┐
│ AuthenticationController  │
│ (AllowAnonymous)         │
└──────┬───────────────────┘
       │
       ↓
┌──────────────────────────┐
│ LoginCommandBuilder      │
│ - GetTenantUserManager(ctx)│
│ - Query AppDbContext     │
│ - Validate password      │
└──────┬───────────────────┘
       │
       ↓
┌──────────────────────────┐
│ CustomAuthService        │
│ - Generate JWT           │
│ - Create RefreshToken    │
└──────┬───────────────────┘
       │
       ↓
┌──────────────────────────┐
│ Token Response            │
│ - accessToken (TenantScheme)│
│ - refreshToken            │
└──────────────────────────┘
```

### Tenant Resolution Middleware Behavior

```csharp
public async Task InvokeAsync(HttpContext context, ITenantRepo tenantRepo)
{
    var path = context.Request.Path.Value ?? string.Empty;
    
    // Check if host endpoint
    var isHostEndpoint = path.StartsWith(
        CustomRouter.HostRouting.HostEndpointPrefix, 
        StringComparison.OrdinalIgnoreCase
    );

    if (isHostEndpoint)
    {
        // Skip tenant resolution - host endpoints don't need tenant context
        await _next(context);
        return;
    }

    // For tenant endpoints, resolve tenant and set CurrentTenant
    // ... tenant resolution logic ...
}
```

### Token Generation and Validation

**Host Token Generation**:
- Uses `IHostCustomAuthenticationService`
- Retrieves roles/claims from `HostIdentityDbContext`
- JWT includes `HostScheme` identifier
- Refresh token stored in `HostIdentityDbContext.RefreshTokens`

**Tenant Token Generation**:
- Uses `ICustomAuthenticationService`
- Retrieves roles/claims from `AppDbContext` (current tenant)
- JWT uses default `TenantScheme`
- Refresh token stored in `AppDbContext.RefreshTokens`

**Token Validation**:
- `HostScheme` tokens validated against Host DB user store
- `TenantScheme` tokens validated against Tenant DB user store
- Authorization attributes specify which scheme to use:
  ```csharp
  [Authorize(AuthenticationSchemes = "HostScheme", Roles = "SystemAdmin")]
  ```

## Key Components

### HostIdentityDbContext

**Location**: `Alforqan.Infrastructure/Data/HostIdentityDbContext.cs`

**Purpose**: EF Core DbContext for host-level Identity operations.

**Inheritance**: `IdentityDbContext<AppUser>`

**Key Features**:
- Uses Host DB connection string
- Contains `DbSet<RefreshToken>` for host refresh tokens
- Configures RefreshToken relationship with AppUser

**Usage**:
```csharp
public class HostIdentityDbContext : IdentityDbContext<AppUser>
{
    public DbSet<RefreshToken> RefreshTokens { get; set; }
    
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);
        // Configure RefreshToken relationships
    }
}
```

### HostAuthController

**Location**: `Alforqan.Api/Controllers/Host/Auth/HostAuthController.cs`

**Purpose**: Authentication endpoints for platform administrators.

**Route**: `/API/V01/Host/Auth`

**Endpoints**:
- `POST /Login` - Host admin login
- `POST /RefreshToken` - Refresh host token (TODO)
- `PATCH /Logout` - Host admin logout (TODO)

**Authorization**: Login endpoint uses `[AllowAnonymous]`, others use `[Authorize(AuthenticationSchemes = "HostScheme")]`

### HostRolesController

**Location**: `Alforqan.Api/Controllers/Host/Roles/HostRolesController.cs`

**Purpose**: Role and user management endpoints for platform administrators.

**Route**: `/API/V01/Host/Roles`

**Endpoints**:

**Role Management**:
- `GET /List` - Get all host roles
- `GET /{id}` - Get host role by ID
- `POST /CreateRole` - Create a new host role
- `PATCH /UpdateRole` - Update host role name
- `DELETE /{id}` - Delete host role

**User Roles Management**:
- `GET /Users/{id}/Roles` - Get host user roles with assignment status
- `PUT /Users/Roles` - Modify host user roles (remove all, add selected)

**User Claims Management**:
- `GET /Users/{id}/Claims` - Get host user claims with assignment status
- `PUT /Users/Claims` - Modify host user claims (remove all, add selected)

**Authorization**: All endpoints require `[Authorize(AuthenticationSchemes = "HostScheme", Roles = ConstantsHelper.IdentityRoleConstants.SystemAdminRole)]`

**Architecture**: Uses shared authorization services (`IAuthorizationRoleService`, `IAuthorizationUserService`) to eliminate code duplication with tenant authorization handlers. Handlers can add host-specific logic before/after service calls if needed.

**Related Documentation**: See `docs/features-explanation-docs/shared-authorization-services-feature.md` for details on the shared services architecture.

### IUserManagerFactory

**Location**: `Alforqan.Service/Abstractions/Identity/IUserManagerFactory.cs`

**Purpose**: Factory to resolve correct `UserManager<AppUser>` instance.

**Methods**:
- `GetHostUserManager(HostIdentityDbContext dbContext)` - Returns UserManager for the provided HostIdentityDbContext
- `GetTenantUserManager(AppDbContext dbContext)` - Returns UserManager for the provided AppDbContext

**Implementation**: Accepts `DbContext` as parameter and uses `ActivatorUtilities.CreateInstance` to construct `UserManager` instances. This ensures:
- The `UserManager` uses the exact `DbContext` instance passed by the caller
- Transaction consistency: `UserManager` operations participate in the same transaction as the caller
- No factory-level caching that could break transaction boundaries

**Related Documentation**: 
- **Feature Explanation**: `docs/features-explanation-docs/dbcontext-consistency-transaction-safety-feature.md` - Detailed explanation of the transaction safety pattern
- **Technical Guide**: `docs/technical-docs/dbcontext-consistency-transaction-safety-guide.md` - Step-by-step usage guide

### HostCustomAuthenticationService

**Location**: `Alforqan.Service/Implementation/Identity/Authentication/HostCustomAuthenticationService.cs`

**Purpose**: Generates JWT tokens and refresh tokens for host users.

**Key Methods**:
- `GenerateUserToken(AppUser user, out string jwtId)` - Creates JWT with host user claims
- `GenerateRefreshToken(string userId, string jwtId)` - Creates refresh token for Host DB

**Difference from Tenant**: Uses `IUserManagerFactory.GetHostUserManager(hostIdentityDbContext)` to retrieve roles/claims from Host DB.

### HostRefreshTokenRepo

**Location**: `Alforqan.Infrastructure/Implementation/Identity/RefreshTokens/HostRefreshTokenRepo.cs`

**Purpose**: Repository for managing refresh tokens in Host DB.

**Base Class**: `HostBaseRepo<RefreshToken, Guid>`

**Operations**: Uses `HostIdentityDbContext` for all refresh token operations.

### HostBuilder and HostDbContextBuilder

**Location**: 
- `Alforqan.Core/Implementations/Base/HostBuilder.cs` (for HostIdentityDbContext)
- `Alforqan.Core/Implementations/Base/HostDbContextBuilder.cs` (for HostDbContext)

**Purpose**: Base builder classes for host-level operations.

**Usage**:
- `HostBuilder<,>` - For operations using `HostIdentityDbContext` (authentication, user management)
- `HostDbContextBuilder<,>` - For operations using `HostDbContext` (tenant management, tenant modules)

## Integration Points

### With Existing Tenant Identity

**Separation**:
- Host Identity uses `AddIdentityCore` to avoid DI conflicts
- Tenant Identity uses `AddIdentity` (full registration)
- Both use same `AppUser` entity type but different DbContexts

**Shared Components**:
- Same `AppUser` entity class
- Same `IdentityRole` class
- Same JWT settings (issuer, audience, secret key)
- Same refresh token entity structure

**Distinct Components**:
- Separate DbContexts (HostIdentityDbContext vs AppDbContext)
- Separate authentication services (HostCustomAuthenticationService vs CustomAuthenticationService)
- Separate refresh token repositories (HostRefreshTokenRepo vs RefreshTokenRepo)
- Separate login endpoints (HostAuthController vs AuthenticationController)

### Middleware Integration

**TenantResolutionMiddleware**:
- Checks request path against `CustomRouter.HostRouting.HostEndpointPrefix`
- Skips tenant resolution for host endpoints
- Allows host operations to proceed without tenant context

**ErrorHandlerMiddleware**:
- Handles exceptions from both host and tenant operations
- Localizes error messages appropriately

### Seeding Integration

**Program.cs Seeding Flow**:
1. Seed Host DB (tenants, modules) using `HostDbContext`
2. Seed tenant roles using tenant `RoleManager`
3. Seed host roles using host `RoleManager` (if tables exist)
4. Seed host users (SystemAdmin) using `IUserManagerFactory.GetHostUserManager(hostIdentityDbContext)`
5. Seed tenant users (Admin) using `IUserManagerFactory.GetTenantUserManager(appDbContext)`

**Error Handling**:
- Host Identity seeding wrapped in try-catch
- If Host Identity tables don't exist, logs warning and skips seeding
- Application continues to start successfully

## Benefits and Trade-offs

### Benefits

1. **Operational Independence**: Platform admins can log in even if tenant databases are down
2. **Clear Separation**: Host and tenant users are clearly separated at the database level
3. **Scalability**: Host operations don't depend on tenant database availability
4. **Security**: Host users have distinct authentication context and authorization scopes
5. **Maintainability**: Unified prefix makes it easy to identify and manage host endpoints

### Trade-offs

1. **Complexity**: Dual Identity systems require careful management of UserManager instances
2. **Code Duplication**: Some authentication logic is duplicated between host and tenant (e.g., token generation)
3. **Migration Management**: Separate migrations for HostIdentityDbContext must be applied independently
4. **Seeding Complexity**: Seeding logic must handle both systems and check for table existence

### Future Considerations

- Consider extracting shared authentication logic into base classes
- May need to add more host-specific roles (e.g., DesignAdmin, SupportAdmin)
- Refresh token and logout endpoints for host users still need implementation
- Consider rate limiting specifically for host authentication endpoints

## Related Files

### Core Implementation
- `Alforqan.Infrastructure/Data/HostIdentityDbContext.cs` - Host Identity DbContext
- `Alforqan.Infrastructure/Data/HostIdentityDbContextFactory.cs` - Design-time factory
- `Alforqan.Infrastructure/ModuleInfrastructureDependencies.cs` - Host Identity registration
- `Alforqan.Shared/ModuleSharedDependencies.cs` - Dual JWT scheme configuration

### Services and Repositories
- `Alforqan.Service/Abstractions/Identity/IUserManagerFactory.cs` - Factory interface
- `Alforqan.Service/Implementation/Identity/UserManagerFactory.cs` - Factory implementation
- `Alforqan.Service/Abstractions/Identity/IHostRoleManagerFactory.cs` - Host role manager factory interface
- `Alforqan.Service/Implementation/Identity/HostRoleManagerFactory.cs` - Host role manager factory implementation
- `Alforqan.Service/Abstractions/Identity/Authorization/IAuthorizationRoleService.cs` - Shared role management service interface
- `Alforqan.Service/Abstractions/Identity/Authorization/IAuthorizationUserService.cs` - Shared user roles/claims service interface
- `Alforqan.Service/Implementation/Identity/Authorization/AuthorizationRoleService.cs` - Shared role management service
- `Alforqan.Service/Implementation/Identity/Authorization/AuthorizationUserService.cs` - Shared user roles/claims service
- `Alforqan.Service/Abstractions/Identity/Authentication/IServices/IHostCustomAuthenticationService.cs` - Host auth service interface
- `Alforqan.Service/Implementation/Identity/Authentication/HostCustomAuthenticationService.cs` - Host auth service
- `Alforqan.Infrastructure/Abstractions/Identity/RefreshTokens/IHostRefreshTokenRepo.cs` - Host refresh token repo interface
- `Alforqan.Infrastructure/Implementation/Identity/RefreshTokens/HostRefreshTokenRepo.cs` - Host refresh token repo

### Controllers and Endpoints
- `Alforqan.Api/Controllers/Host/Auth/HostAuthController.cs` - Host authentication controller
- `Alforqan.Api/Controllers/Host/Roles/HostRolesController.cs` - Host role and user management controller
- `Alforqan.Api/Controllers/Tenants/TenantsController.cs` - Uses HostScheme authorization

### Middleware
- `Alforqan.Api/Middlewares/TenantResolutionMiddleware.cs` - Skips tenant resolution for host routes

### Routing
- `Alforqan.Data/AppMetaData/CustomRouter.cs` - Defines `HostRouting` class with unified prefix

### Builders
- `Alforqan.Core/Builders/Implementations/Base/HostBuilder.cs` - Base builder for HostIdentityDbContext
- `Alforqan.Core/Builders/Implementations/Base/HostDbContextBuilder.cs` - Base builder for HostDbContext
- `Alforqan.Core/Builders/Implementations/Identity/HostAuth/Commands/HostLoginCommandBuilder.cs` - Host login builder

### Commands and Handlers
- `Alforqan.Core/Features/Identity/HostAuth/Commands/Models/HostLoginCommand.cs` - Host login command
- `Alforqan.Core/Features/Identity/HostAuth/Commands/Handlers/HostAuthenticationCommandHandler.cs` - Host auth handler
- `Alforqan.Core/Features/Identity/HostAuthorization/Commands/Models/` - Host role and user management commands
- `Alforqan.Core/Features/Identity/HostAuthorization/Commands/Handlers/HostAuthorizationCommandHandler.cs` - Host authorization command handler
- `Alforqan.Core/Features/Identity/HostAuthorization/Queries/Models/` - Host role and user management queries
- `Alforqan.Core/Features/Identity/HostAuthorization/Queries/Handlers/HostAuthorizationQueryHandler.cs` - Host authorization query handler

### Seeding
- `Alforqan.Infrastructure/Seeding/AppUserSeeding.cs` - Contains `SeedHostUsersAsync` and `SeedTenantUsersAsync`
- `Alforqan.Api/Program.cs` - Orchestrates seeding for both systems

### Migrations
- `Alforqan.Infrastructure/Migrations/HostIdentityDb/` - Host Identity migrations directory





