# Authorization Services Guide

## How to Use Shared Authorization Services

### Step 1: Inject Services in Handler

```csharp
private readonly IAuthorizationRoleService roleService;
private readonly IAuthorizationUserService userService;

public MyHandler(
    IAuthorizationRoleService roleService,
    IAuthorizationUserService userService)
{
    this.roleService = roleService;
    this.userService = userService;
}
```

### Step 2: Call Service Methods

#### Role Operations

```csharp
// Create role
var roleManager = _hostRoleManagerFactory.CreateHostRoleManager(hostIdentityDbContext);
var response = await roleService.CreateRoleAsync(
    request.Name!,
    roleManager,
    cancellationToken);

// Update role
var response = await roleService.UpdateRoleAsync(
    request.Id!,
    request.Name!,
    roleManager,
    cancellationToken);

// Delete role
var response = await roleService.DeleteRoleAsync(
    request.id!,
    roleManager,
    cancellationToken);

// Get role by ID
var response = await roleService.GetRoleByIdAsync(
    request.Id,
    roleManager,
    cancellationToken);

// Get all roles
var response = await roleService.GetRolesListAsync(
    roleManager,
    cancellationToken);
```

#### User Roles Operations

```csharp
// Get user roles
var userManager = userManagerFactory.GetHostUserManager(hostIdentityDbContext);
var roleManager = hostRoleManagerFactory.CreateHostRoleManager(hostIdentityDbContext);
var response = await userService.GetUserRolesAsync(
    request.UserId,
    roleManager,
    userManager,
    cancellationToken);

// Modify user roles
var userRolesDto = request.UserRoles!.Select(r => new SharedDto.ModifyUserRolesCommandDto
{
    RoleName = r.RoleName,
    HasRole = r.HasRole
}).ToList();
var response = await userService.ModifyUserRolesAsync(
    request.UserId!,
    userRolesDto,
    roleManager,
    userManager,
    dbContext,
    cancellationToken);
```

#### User Claims Operations

```csharp
// Get user claims
var userManager = userManagerFactory.GetHostUserManager(hostIdentityDbContext);
var response = await userService.GetUserClaimsAsync(
    request.UserId,
    userManager,
    cancellationToken);

// Modify user claims
var userClaimsDto = request.UserClaims!.Select(c => new SharedDto.GetUserClaimsQueryResponseDto.UserClaimDto
{
    ClaimType = c.ClaimType,
    ClaimValue = c.ClaimValue
}).ToList();
var response = await userService.ModifyUserClaimsAsync(
    Guid.Parse(request.UserId!),
    userClaimsDto,
    userManager,
    dbContext,
    cancellationToken);
```

### Step 3: Add Host-Specific Logic (Optional)

```csharp
public async Task<GenericResponse<IdentityRole>> Handle(
    CreateHostRoleCommand request,
    CancellationToken cancellationToken)
{
    // Host-specific validation BEFORE service call
    if (!request.Name!.StartsWith("Host_", StringComparison.OrdinalIgnoreCase))
    {
        return BadRequest<IdentityRole>("Host roles must start with 'Host_'.");
    }

    var roleManager = hostRoleManagerFactory.CreateHostRoleManager(hostIdentityDbContext);
    
    // Shared core logic
    var result = await roleService.CreateRoleAsync(
        request.Name!,
        roleManager,
        cancellationToken);

    // Host-specific logic AFTER service call (optional)
    if (result.Succeeded)
    {
        // Log audit, send notification, etc.
    }

    return result;
}
```

### Step 4: Map DTOs Between Core and Shared

When using Shared DTOs in Core handlers:

```csharp
using CoreDto = Alforqan.Core.Features.Identity.Authorization.Queries.ResponseDtos;
using SharedDto = Alforqan.Shared.Dtos.Authorization;

// Map Core DTO to Shared DTO
var userRolesDto = request.UserRoles!.Select(r => new SharedDto.ModifyUserRolesCommandDto
{
    RoleName = r.RoleName,
    HasRole = r.HasRole
}).ToList();

// Call service with Shared DTO
var response = await userService.ModifyUserRolesAsync(...);

// Map Shared DTO back to Core DTO for response
if (response.Succeeded && response.Data != null)
{
    var coreDto = new CoreDto.GetUserRolesQueryResponseDto
    {
        UserId = response.Data.UserId,
        UserRoles = response.Data.UserRoles.Select(r => new CoreDto.UserRoleDto
        {
            RoleId = r.RoleId,
            RoleName = r.RoleName,
            HasRole = r.HasRole
        }).ToList()
    };
    return new GenericResponse<CoreDto.GetUserRolesQueryResponseDto>
    {
        Data = coreDto,
        StatusCode = response.StatusCode,
        Succeeded = response.Succeeded,
        Message = response.Message,
        Errors = response.Errors
    };
}
```

