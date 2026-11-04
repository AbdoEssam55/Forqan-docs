# DbContext Consistency and Transaction Safety Guide

## How to Use UserManagerFactory in Builders

```csharp
public class LoginCommandBuilder : AppBuilder<LoginCommand, TokenResponseDto>
{
    private readonly IUserManagerFactory userManagerFactory;
    private UserManager<AppUser>? _tenantUserManager;
    
    private UserManager<AppUser> TenantUserManager => 
        _tenantUserManager ??= userManagerFactory.GetTenantUserManager((AppDbContext)_dbContext);

    public async Task<GenericResponse<TokenResponseDto>> GetUserByUsernameOrEmail()
    {
        _user = await TenantUserManager.Users
            .Where(user => user.Email == _input.Email)
            .FirstOrDefaultAsync(_cancellationToken);
        return null;
    }
}
```

## How to Use UserManagerFactory in Host Builders

```csharp
public class HostLoginCommandBuilder : HostBuilder<HostLoginCommand, TokenResponseDto>
{
    private readonly IUserManagerFactory userManagerFactory;
    private UserManager<AppUser>? _hostUserManager;
    
    private UserManager<AppUser> HostUserManager => 
        _hostUserManager ??= userManagerFactory.GetHostUserManager((HostIdentityDbContext)_dbContext);

    public async Task<GenericResponse<TokenResponseDto>> GetUserByUsernameOrEmail()
    {
        _user = await HostUserManager.Users
            .Where(user => user.Email == _input.Email)
            .FirstOrDefaultAsync(_cancellationToken);
        return null;
    }
}
```

## How to Use UserManagerFactory in Handlers

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
        var tenantUserManager = userManagerFactory.GetTenantUserManager(dbContext);
        // Use tenantUserManager...
    }
}
```

## How to Use Services Called from Builders

**⚠️ CRITICAL**: When services are called from builders, they must accept `UserManager` as parameter to maintain transaction consistency.

### Service Interface

```csharp
public interface ICustomAuthenticationService
{
    Task<(string token, string jwtId)> GenerateUserTokenAsync(
        AppUser user, 
        UserManager<AppUser> userManager, 
        CancellationToken cancellationToken = default);
    
    RefreshToken GenerateRefreshToken(string userId, string jwtId);
}
```

### Service Implementation

```csharp
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
        
        // Use provided UserManager - NO .Result!
        var userRoles = await userManager.GetRolesAsync(user);
        var userAuthClaims = await userManager.GetClaimsAsync(user);
        // ... build token ...
        return (tokenString, jwtId);
    }
}
```

### Builder Usage

```csharp
public class LoginCommandBuilder : AppBuilder<LoginCommand, TokenResponseDto>
{
    private readonly ICustomAuthenticationService customAuthenticationService;
    private UserManager<AppUser>? _tenantUserManager;
    
    private UserManager<AppUser> TenantUserManager => 
        _tenantUserManager ??= userManagerFactory.GetTenantUserManager((AppDbContext)_dbContext);

    public async Task<GenericResponse<TokenResponseDto>> GenerateAccessTokenAndRefreshToken()
    {
        // Pass cached UserManager to service
        var (token, jwtId) = await customAuthenticationService
            .GenerateUserTokenAsync(_user!, TenantUserManager, _cancellationToken);
        
        var refreshToken = customAuthenticationService.GenerateRefreshToken(_user!.Id, jwtId);
        return null;
    }
}
```

## How to Use HostRoleManagerFactory in Handlers

```csharp
public class HostAuthorizationCommandHandler : GenericResponseHandler
{
    private readonly IHostRoleManagerFactory hostRoleManagerFactory;
    private readonly HostIdentityDbContext hostIdentityDbContext;

    public HostAuthorizationCommandHandler(
        IHostRoleManagerFactory hostRoleManagerFactory,
        HostIdentityDbContext hostIdentityDbContext)
    {
        this.hostRoleManagerFactory = hostRoleManagerFactory;
        this.hostIdentityDbContext = hostIdentityDbContext;
    }

    public async Task<GenericResponse<IdentityRole>> Handle(
        CreateHostRoleCommand request,
        CancellationToken cancellationToken)
    {
        var roleManager = hostRoleManagerFactory.CreateHostRoleManager(hostIdentityDbContext);
        return await roleService.CreateRoleAsync(request.Name!, roleManager, cancellationToken);
    }
}
```

## How to Use Factories in Program.cs Seeding

```csharp
using (var scope = app.Services.CreateScope())
{
    var userManagerFactory = scope.ServiceProvider.GetRequiredService<IUserManagerFactory>();
    var appDbContext = scope.ServiceProvider.GetRequiredService<AppDbContext>();
    var hostIdentityDbContext = scope.ServiceProvider.GetRequiredService<HostIdentityDbContext>();
    
    var tenantUserManager = userManagerFactory.GetTenantUserManager(appDbContext);
    var hostUserManager = userManagerFactory.GetHostUserManager(hostIdentityDbContext);
    
    // Use managers for seeding...
}
```

## Migration: Updating Existing Code

### Factory Calls

**Before**:
```csharp
var userManager = userManagerFactory.GetTenantUserManager();
```

**After (Builders)**:
```csharp
var userManager = userManagerFactory.GetTenantUserManager((AppDbContext)_dbContext);
```

**After (Handlers/Services)**:
```csharp
var userManager = userManagerFactory.GetTenantUserManager(dbContext);
```

### Service Calls from Builders

**Before (❌ Wrong - breaks transaction)**:
```csharp
// Service uses its own DbContext from DI
var token = customAuthenticationService.GenerateUserToken(_user!, out string jwtId);
```

**After (✅ Correct - maintains transaction)**:
```csharp
// Service uses builder's UserManager (same DbContext)
var (token, jwtId) = await customAuthenticationService
    .GenerateUserTokenAsync(_user!, TenantUserManager, _cancellationToken);
```

**Key Changes**:
1. Service method is now `async` and returns `Task<(string token, string jwtId)>`
2. Service accepts `UserManager` as parameter
3. Builder passes cached `UserManager` to service
4. Use `await` instead of `.Result` (prevents deadlocks)

