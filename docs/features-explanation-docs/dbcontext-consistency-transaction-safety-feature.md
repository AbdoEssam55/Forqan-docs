# DbContext Consistency and Transaction Safety Feature

## Problem Statement

When using `UserManager` and `RoleManager` in builders with transactions, there was a critical issue: factory-level caching could cause `UserManager`/`RoleManager` to use a different `DbContext` instance than the builder's transaction, leading to silent transaction breaks and data inconsistency.

### The Problem Scenario

1. Builder receives `DbContext A` from DI (or creates it via `IDbContextFactory`)
2. Builder starts a transaction on `DbContext A`
3. Builder calls `userManagerFactory.GetTenantUserManager()` (no parameter)
4. Factory resolves `DbContext B` from DI (different instance)
5. Factory caches `UserManager` with `DbContext B`
6. Builder's transaction uses `DbContext A`, but `UserManager` uses `DbContext B`
7. **Result**: Operations are in different transactions → data inconsistency

### Why This Happens

Multiple `DbContext` instances can exist in the same scope when:
- Using `IDbContextFactory<T>.CreateDbContext()` to create new contexts
- Creating new `DbContext` instances manually (`new AppDbContext()`)
- Using `IServiceProvider.CreateScope()` to create nested scopes

Factory-level caching (`_hostUserManager`, `_tenantUserManager`) captures the first `DbContext` instance resolved from DI, which may not match the builder's `DbContext`.

## Architecture and Design Decisions

### Solution Approach

**Approach**: Factories accept `DbContext` as parameter and remove caching. Caching happens in builders (per builder instance).

### Updated Factory Interface

```csharp
public interface IUserManagerFactory
{
    UserManager<AppUser> GetHostUserManager(HostIdentityDbContext dbContext);
    UserManager<AppUser> GetTenantUserManager(AppDbContext dbContext);
}

public interface IHostRoleManagerFactory
{
    RoleManager<IdentityRole> CreateHostRoleManager(HostIdentityDbContext dbContext);
}
```

### Factory Implementation

```csharp
public class UserManagerFactory : IUserManagerFactory
{
    private readonly IServiceProvider _serviceProvider;

    public UserManagerFactory(IServiceProvider serviceProvider)
    {
        _serviceProvider = serviceProvider;
    }

    public UserManager<AppUser> GetTenantUserManager(AppDbContext dbContext)
    {
        var describer = _serviceProvider.GetRequiredService<IdentityErrorDescriber>();
        var userStore = new UserStore<AppUser, IdentityRole, AppDbContext, string>(dbContext, describer);

        // DI fills all the other dependencies of UserManager
        return ActivatorUtilities.CreateInstance<UserManager<AppUser>>(_serviceProvider, userStore);
    }

    public UserManager<AppUser> GetHostUserManager(HostIdentityDbContext dbContext)
    {
        var describer = _serviceProvider.GetRequiredService<IdentityErrorDescriber>();
        var userStore = new UserStore<AppUser, IdentityRole, HostIdentityDbContext, string>(dbContext, describer);

        // DI fills all the other dependencies of UserManager
        return ActivatorUtilities.CreateInstance<UserManager<AppUser>>(_serviceProvider, userStore);
    }
}
```

**Key Design Decisions**:
- **No caching in factory**: Each call creates a new `UserManager` instance with the provided `DbContext`
- **Accept DbContext as parameter**: Caller explicitly controls which `DbContext` is used
- **Use ActivatorUtilities.CreateInstance**: Clean construction with DI handling all dependencies
- **Caching in builders**: Builders cache `UserManager`/`RoleManager` per builder instance to avoid repeated construction

## How It Works

### Transaction Flow

```
1. Builder receives DbContext A from base class (_dbContext)
   ↓
2. Builder starts transaction on DbContext A
   ↓
3. Builder calls factory.GetTenantUserManager((AppDbContext)_dbContext)
   ↓
4. Factory creates UserManager with DbContext A (same instance)
   ↓
5. Builder caches UserManager (per builder instance)
   ↓
6. All operations use DbContext A → Same transaction ✅
```

### Builder Pattern

Builders cache `UserManager`/`RoleManager` per builder instance:

```csharp
public class LoginCommandBuilder : AppBuilder<LoginCommand, TokenResponseDto>
{
    private readonly IUserManagerFactory userManagerFactory;
    private UserManager<AppUser>? _tenantUserManager;
    
    // Cache per builder instance
    private UserManager<AppUser> TenantUserManager => 
        _tenantUserManager ??= userManagerFactory.GetTenantUserManager((AppDbContext)_dbContext);

    public async Task<GenericResponse<TokenResponseDto>> GetUserByUsernameOrEmail()
    {
        // Uses the same DbContext as builder's transaction
        _user = await TenantUserManager.Users
            .Where(user => user.Email == _input.Email)
            .FirstOrDefaultAsync(_cancellationToken);
        return null;
    }
}
```

### Handler Pattern

Handlers inject `DbContext` and pass it to factories:

```csharp
public class AuthorizationCommandHandler : GenericResponseHandler
{
    private readonly IUserManagerFactory userManagerFactory;
    private readonly AppDbContext dbContext;

    public AuthorizationCommandHandler(
        IUserManagerFactory userManagerFactory,
        AppDbContext dbContext)
    {
        this.userManagerFactory = userManagerFactory;
        this.dbContext = dbContext;
    }

    public async Task<GenericResponse<string>> Handle(
        ModifyUserRolesCommand request,
        CancellationToken cancellationToken)
    {
        // Pass the injected DbContext
        var tenantUserManager = userManagerFactory.GetTenantUserManager(dbContext);
        
        // Use tenantUserManager...
        return await userService.ModifyUserRolesAsync(
            request.UserId!,
            userRolesDto,
            roleManager,
            tenantUserManager,
            dbContext,
            cancellationToken);
    }
}
```

### Service Pattern

**⚠️ CRITICAL: Services Called from Builders**

When services are called from builders (e.g., `CustomAuthenticationService.GenerateUserTokenAsync()`), the service must use the builder's `UserManager` to maintain transaction consistency. Services should accept `UserManager` as a parameter when called from builders.

**Problem**: If a service uses its own injected `DbContext` from DI, it may use a different `DbContext` instance than the builder's transaction, breaking transaction consistency.

**Solution**: Services accept `UserManager` as parameter when called from builders:

```csharp
public interface ICustomAuthenticationService
{
    /// <summary>
    /// Generates JWT token for user (async, accepts UserManager for transaction safety)
    /// Returns tuple: (token, jwtId)
    /// </summary>
    Task<(string token, string jwtId)> GenerateUserTokenAsync(
        AppUser user, 
        UserManager<AppUser> userManager, 
        CancellationToken cancellationToken = default);
    
    RefreshToken GenerateRefreshToken(string userId, string jwtId);
}

public class CustomAuthenticationService : ICustomAuthenticationService
{
    private readonly JWTSettings jwtSettings;
    private readonly IHttpContextAccessor httpContextAccessor;

    public CustomAuthenticationService(
        JWTSettings jwtSettings,
        IHttpContextAccessor httpContextAccessor)
    {
        this.jwtSettings = jwtSettings;
        this.httpContextAccessor = httpContextAccessor;
    }

    public async Task<(string token, string jwtId)> GenerateUserTokenAsync(
        AppUser user, 
        UserManager<AppUser> userManager, 
        CancellationToken cancellationToken = default)
    {
        var jwtId = Guid.NewGuid().ToString();
        // ... build claims ...
        
        // Use provided UserManager (from builder's transaction) - NO .Result!
        var userRoles = await userManager.GetRolesAsync(user);
        var userAuthClaims = await userManager.GetClaimsAsync(user);
        // ... build token ...
        return (tokenString, jwtId);
    }
}
```

**Builder Usage**:

```csharp
public class LoginCommandBuilder : AppBuilder<LoginCommand, TokenResponseDto>
{
    private readonly ICustomAuthenticationService customAuthenticationService;
    private UserManager<AppUser>? _tenantUserManager;
    
    private UserManager<AppUser> TenantUserManager => 
        _tenantUserManager ??= userManagerFactory.GetTenantUserManager((AppDbContext)_dbContext);

    public async Task<GenericResponse<TokenResponseDto>> GenerateAccessTokenAndRefreshToken()
    {
        // Pass cached UserManager to service - ensures same DbContext
        var (token, jwtId) = await customAuthenticationService
            .GenerateUserTokenAsync(_user!, TenantUserManager, _cancellationToken);
        
        var refreshToken = customAuthenticationService.GenerateRefreshToken(_user!.Id, jwtId);
        // ...
    }
}
```

**Key Points**:
- ✅ Services accept `UserManager` as parameter when called from builders
- ✅ Builders pass their cached `UserManager` to services
- ✅ Services use `await` (no `.Result` - prevents deadlocks)
- ✅ All operations use the same `DbContext` → transaction safety

## Integration with Existing Systems

### Builder Integration

- Builders inherit from `AppBuilder<,>` or `HostBuilder<,>` which provide `_dbContext`
- Builders cast `_dbContext` to the appropriate type and pass to factories
- Builders cache `UserManager`/`RoleManager` per builder instance

### Handler Integration

- Handlers inject `DbContext` via constructor
- Handlers pass `DbContext` to factories when needed
- Handlers can use `UserManager`/`RoleManager` for the duration of the request

### Service Integration

**When Services Are Called from Builders**:
- Services accept `UserManager`/`RoleManager` as parameters (not `DbContext`)
- Builders pass their cached `UserManager`/`RoleManager` to services
- Services use async methods (no `.Result` calls)
- All operations use the same `DbContext` as the builder's transaction

**When Services Are Called from Handlers**:
- Services can still accept `UserManager`/`RoleManager` as parameters
- Or handlers can create `UserManager`/`RoleManager` and pass to services
- Services use async methods (no `.Result` calls)

## Benefits and Trade-offs

### Benefits

1. **Transaction Safety**: `UserManager`/`RoleManager` always use the same `DbContext` as the caller's transaction
2. **Correctness**: No silent transaction breaks when multiple `DbContext` instances exist in scope
3. **Explicit Control**: Caller explicitly controls which `DbContext` is used
4. **Maintainability**: Clear pattern - factories construct, builders/handlers cache
5. **Clean Code**: Uses DI properly via `ActivatorUtilities.CreateInstance`

### Trade-offs

1. **Parameter Passing**: All factory calls must include `DbContext` parameter (more verbose but explicit)
2. **No Factory Caching**: Each factory call creates a new instance (but builders cache, so minimal overhead)
3. **Migration Required**: Existing code must be updated to pass `DbContext` parameters
4. **Service Method Signatures**: Services called from builders must accept `UserManager` as parameter (breaking change, but necessary for transaction safety)

## Important Considerations

### Type Parameters

Ensure `UserStore`/`RoleStore` generic parameters match your actual types:

```csharp
// Correct for AppUser : IdentityUser (string key)
var userStore = new UserStore<AppUser, IdentityRole, AppDbContext, string>(dbContext, describer);

// Correct for IdentityRole (string key)
var roleStore = new RoleStore<IdentityRole, HostIdentityDbContext, string>(dbContext, describer);
```

### Identity Services Registration

`ActivatorUtilities.CreateInstance` requires all `UserManager` dependencies to be registered in DI:

- `IOptions<IdentityOptions>` - Registered by `AddIdentity`/`AddIdentityCore`
- `IPasswordHasher<AppUser>` - Registered by `AddIdentity`/`AddIdentityCore`
- `IUserValidator<AppUser>` - Registered by `AddIdentity`/`AddIdentityCore`
- `IPasswordValidator<AppUser>` - Registered by `AddIdentity`/`AddIdentityCore`
- `ILookupNormalizer` - Registered by `AddIdentity`/`AddIdentityCore`
- `IdentityErrorDescriber` - Registered by `AddIdentity`/`AddIdentityCore`
- `ILogger<UserManager<AppUser>>` - Registered by default logging

## Related Files

- `Alforqan.Service/Abstractions/Identity/IUserManagerFactory.cs` - Factory interface
- `Alforqan.Service/Implementation/Identity/UserManagerFactory.cs` - Factory implementation
- `Alforqan.Service/Abstractions/Identity/IHostRoleManagerFactory.cs` - Role manager factory interface
- `Alforqan.Service/Implementation/Identity/HostRoleManagerFactory.cs` - Role manager factory implementation
- `Alforqan.Core/Builders/Implementations/Base/Builder.cs` - Base builder with transaction support
- `Alforqan.Core/Builders/Implementations/Base/AppBuilder.cs` - Tenant builder base class
- `Alforqan.Core/Builders/Implementations/Base/HostBuilder.cs` - Host builder base class

## Related Documentation

- **Technical Guide**: `docs/technical-docs/dbcontext-consistency-transaction-safety-guide.md` - Step-by-step usage guide

