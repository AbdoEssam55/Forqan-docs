# Authentication Base Builder Pattern Guide

## How to Create a Base Builder for Authentication Operations

### 1. Create Input Interface (if needed)

Create marker interface in `Alforqan.Core/Features/Identity/Tenant/Authentication/Shared/InputInterfaces/`:

```csharp
public interface IMyOperationInput
{
    string? Property1 { get; set; }
    string? Property2 { get; set; }
}
```

Update command/query models to implement the interface.

### 2. Create Base Interface

Create in `Alforqan.Core/Builders/Abstractions/Identity/Base/Authentication/Commands/`:

```csharp
public interface IMyOperationCommandBuilderBase<TInput, TResponse>
    : IBuilder<TInput, TResponse>
{
    Task<GenericResponse<TResponse>> Step1();
    Task<GenericResponse<TResponse>> Step2();
    // ... other step methods
}
```

### 3. Create Base Builder Implementation

Create in `Alforqan.Core/Builders/Implementations/Identity/Base/Authentication/Commands/`:

```csharp
public abstract class MyOperationCommandBuilderBase<TInput, TResponse, TDbContext>
    : Builder<TInput, TResponse, TDbContext>
    where TInput : IMyOperationInput
    where TDbContext : DbContext
{
    #region abstract properties
    protected abstract UserManager<AppUser> UserManager { get; }
    protected abstract ICustomAuthenticationService AuthService { get; }
    protected abstract IRefreshTokenRepo RefreshTokenRepo { get; }
    // ... other abstract dependencies
    #endregion

    #region protected properties
    protected AppUser? _user { get; set; }
    // ... other shared state
    #endregion

    #region constructors
    protected MyOperationCommandBuilderBase(IServiceProvider serviceProvider, TDbContext dbContext)
        : base(serviceProvider, dbContext)
    {
    }
    #endregion

    #region methods
    // Shared methods implemented here
    // Abstract methods for tenant/host-specific logic
    #endregion
}
```

### 4. Identify Shared vs Different Logic

**Shared methods** (implement in base):
- Identical logic between tenant and host
- Use abstract properties for dependencies

**Abstract methods** (override in concrete builders):
- Different implementations (e.g., SignInManager vs UserManager)
- Different return types or service calls

### 5. Create Tenant Builder

In `Alforqan.Core/Builders/Implementations/Identity/Tenant/Authentication/Commands/`:

```csharp
public class MyOperationCommandBuilder
    : MyOperationCommandBuilderBase<MyOperationCommand, MyResponseDto, AppDbContext>,
      IMyOperationCommandBuilder
{
    #region private fields
    private readonly IUserManagerFactory userManagerFactory;
    private readonly ICustomAuthenticationService customAuthenticationService;
    private readonly IRefreshTokenRepo refreshTokenRepo;
    #endregion

    #region private properties
    private UserManager<AppUser>? _tenantUserManager;
    private UserManager<AppUser> TenantUserManager =>
        _tenantUserManager ??= userManagerFactory.GetTenantUserManager(_dbContext);
    #endregion

    #region abstract property implementations
    protected override UserManager<AppUser> UserManager => TenantUserManager;
    protected override ICustomAuthenticationService AuthService => customAuthenticationService;
    protected override IRefreshTokenRepo RefreshTokenRepo => refreshTokenRepo;
    #endregion

    #region constructors
    public MyOperationCommandBuilder(
        IUserManagerFactory userManagerFactory,
        ICustomAuthenticationService customAuthenticationService,
        IRefreshTokenRepo refreshTokenRepo,
        IServiceProvider serviceProvider,
        AppDbContext dbContext) : base(serviceProvider, dbContext)
    {
        this.userManagerFactory = userManagerFactory;
        this.customAuthenticationService = customAuthenticationService;
        this.refreshTokenRepo = refreshTokenRepo;
    }
    #endregion

    #region methods
    // Override only abstract methods
    public override async Task<GenericResponse<MyResponseDto>> DifferentMethod()
    {
        // Tenant-specific implementation
        return null;
    }
    #endregion
}
```

### 6. Create Host Builder

In `Alforqan.Core/Builders/Implementations/Identity/Host/HostAuth/Commands/`:

```csharp
public class HostMyOperationCommandBuilder
    : MyOperationCommandBuilderBase<HostMyOperationCommand, MyResponseDto, HostIdentityDbContext>,
      IHostMyOperationCommandBuilder
{
    #region private fields
    private readonly IUserManagerFactory userManagerFactory;
    private readonly IHostCustomAuthenticationService hostCustomAuthenticationService;
    private readonly IHostRefreshTokenRepo hostRefreshTokenRepo;
    #endregion

    #region private properties
    private UserManager<AppUser>? _hostUserManager;
    private UserManager<AppUser> HostUserManager =>
        _hostUserManager ??= userManagerFactory.GetHostUserManager(_dbContext);
    #endregion

    #region abstract property implementations
    protected override UserManager<AppUser> UserManager => HostUserManager;
    protected override ICustomAuthenticationService AuthService => hostCustomAuthenticationService; // Cast to base
    protected override IRefreshTokenRepo RefreshTokenRepo => hostRefreshTokenRepo; // Cast to base
    #endregion

    #region constructors
    public HostMyOperationCommandBuilder(
        IUserManagerFactory userManagerFactory,
        IHostCustomAuthenticationService hostCustomAuthenticationService,
        IHostRefreshTokenRepo hostRefreshTokenRepo,
        IServiceProvider serviceProvider,
        HostIdentityDbContext hostDbContext) : base(serviceProvider, hostDbContext)
    {
        this.userManagerFactory = userManagerFactory;
        this.hostCustomAuthenticationService = hostCustomAuthenticationService;
        this.hostRefreshTokenRepo = hostRefreshTokenRepo;
    }
    #endregion

    #region methods
    // Override only abstract methods
    public override async Task<GenericResponse<MyResponseDto>> DifferentMethod()
    {
        // Host-specific implementation
        return null;
    }
    #endregion
}
```

### 7. Update Builder Interfaces

**Tenant interface** (`Alforqan.Core/Builders/Abstractions/Identity/Tenant/Authentication/Commands/`):

```csharp
public interface IMyOperationCommandBuilder
    : IMyOperationCommandBuilderBase<MyOperationCommand, MyResponseDto>
{
}
```

**Host interface** (`Alforqan.Core/Builders/Abstractions/Identity/Host/HostAuth/Commands/`):

```csharp
public interface IHostMyOperationCommandBuilder
    : IMyOperationCommandBuilderBase<HostMyOperationCommand, MyResponseDto>
{
}
```

## Key Patterns

### Abstract Properties for Dependencies

Use abstract properties for services that differ between tenant/host:

```csharp
protected abstract UserManager<AppUser> UserManager { get; }
protected abstract ICustomAuthenticationService AuthService { get; }
protected abstract IRefreshTokenRepo RefreshTokenRepo { get; }
```

### Protected Properties for Shared State

Store intermediate results used across steps:

```csharp
protected AppUser? _user { get; set; }
protected string? _token { get; set; }
protected bool _isValid { get; set; }
```

### Abstract Methods for Different Implementations

When logic differs significantly:

```csharp
public abstract Task<GenericResponse<TResponse>> SignInTheUser();
```

### Interface Inheritance for DRY

Base interfaces eliminate method signature duplication:

```csharp
public interface IMyOperationCommandBuilderBase<TInput, TResponse>
    : IBuilder<TInput, TResponse>
{
    // Common method signatures
}
```

## Files to Reference

- Base Builders: `Alforqan.Core/Builders/Implementations/Identity/Base/Authentication/Commands/`
- Base Interfaces: `Alforqan.Core/Builders/Abstractions/Identity/Base/Authentication/Commands/`
- Examples: `LoginCommandBuilderBase`, `RefreshTokenCommandBuilderBase`, `LogoutCommandBuilderBase`
