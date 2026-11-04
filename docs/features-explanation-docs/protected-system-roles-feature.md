# Protected System Roles Feature - Detailed Explanation

## Overview

The Protected System Roles Feature in Alforqan prevents critical system roles from being modified or deleted through the API. This protection mechanism ensures that core roles (SystemAdmin, Admin, User) cannot be accidentally or maliciously altered, maintaining system integrity and preventing privilege escalation attacks. The protection applies uniformly to both tenant roles and host roles, ensuring consistent security across the multi-tenant architecture.

## Problem Statement

In a multi-tenant system with role-based access control, certain roles are fundamental to system operation:

1. **System Stability**: Core roles like "SystemAdmin", "Admin", and "User" are essential for system functionality. Deleting or renaming these roles could break authorization checks throughout the application.

2. **Security Risk**: Without protection, an attacker with admin privileges could:
   - Delete the "Admin" role, locking out legitimate administrators
   - Rename the "SystemAdmin" role to bypass authorization checks
   - Modify role names to exploit hardcoded role checks in the codebase

3. **Accidental Deletion**: Administrators might accidentally delete critical roles during role management operations, causing system-wide authorization failures.

4. **Code Dependencies**: The codebase references role names via constants (e.g., `ConstantsHelper.IdentityRoleConstants.AdminRole`). If roles are renamed or deleted, these references become invalid, causing runtime errors.

5. **Multi-Tenant Consistency**: Both tenant databases and the host database need consistent role protection. A role protected in one context but not another creates security inconsistencies.

## Architecture & Design

### Components

1. **ProtectedSystemRoles List** (`Alforqan.Shared/HelperClasses/ConstantsHelper.cs`)
   - Centralized list of protected role names
   - Defined as `IReadOnlyList<string>` for immutability
   - Contains: `SystemAdminRole`, `AdminRole`, `UserRole`
   - Single source of truth for all protection rules

2. **IsProtectedSystemRole() Method** (`Alforqan.Shared/HelperClasses/ConstantsHelper.cs`)
   - Static helper method to check if a role is protected
   - Returns `true` if role name exists in `ProtectedSystemRoles` list
   - Null-safe implementation
   - Used consistently across tenant and host contexts

3. **AuthorizationRoleService** (`Alforqan.Service/Implementation/Identity/Authorization/AuthorizationRoleService.cs`)
   - Service layer that enforces protection rules
   - Checks `IsProtectedSystemRole()` before allowing updates/deletes
   - Returns `UnprocessableEntity` response if protection check fails
   - Used by both tenant and host role management

4. **RoleSeeding** (`Alforqan.Infrastructure/Seeding/RoleSeeding.cs`)
   - Seeds protected roles into databases at startup
   - Validates that all protected roles exist in database
   - Throws exception if protected roles are missing
   - Separate methods for host roles and tenant roles

5. **Controllers** (Tenant: `AuthorizationController`, Host: `HostRolesController`)
   - Expose role management endpoints
   - Delegate to `AuthorizationRoleService` for business logic
   - Protection is enforced at service layer, not controller layer

## How It Works

### Flow Diagram - Role Update Attempt

```
1. User Attempts to Update Role
   ↓
2. Controller Receives Request (AuthorizationController or HostRolesController)
   ↓
3. Controller Sends Command/Query to MediatR Handler
   ↓
4. Handler Calls AuthorizationRoleService.UpdateRoleAsync()
   ↓
5. AuthorizationRoleService.UpdateRoleAsync()
   ├─ [1] Check if role exists by ID
   ├─ [2] Check if role is protected:
   │  └─ if (ConstantsHelper.IdentityRoleConstants.IsProtectedSystemRole(role.Name))
   │     └─ return UnprocessableEntity("Cannot modify system role")
   ├─ [3] Check for duplicate role names
   ├─ [4] Update role if not protected
   └─ [5] Return success response
   ↓
6. Response Returned to Client
   ├─ If protected: 422 Unprocessable Entity with error message
   └─ If not protected: 200 OK with updated role
```

### Flow Diagram - Role Delete Attempt

```
1. User Attempts to Delete Role
   ↓
2. Controller Receives Request
   ↓
3. Handler Calls AuthorizationRoleService.DeleteRoleAsync()
   ↓
4. AuthorizationRoleService.DeleteRoleAsync()
   ├─ [1] Check if role exists by ID
   ├─ [2] Check if role is protected:
   │  └─ if (ConstantsHelper.IdentityRoleConstants.IsProtectedSystemRole(role.Name))
   │     └─ return UnprocessableEntity("Cannot delete system role")
   ├─ [3] Delete role if not protected
   └─ [4] Return success response
   ↓
5. Response Returned to Client
   ├─ If protected: 422 Unprocessable Entity with error message
   └─ If not protected: 200 OK with success message
```

### Protection Check Implementation

```csharp
// In ConstantsHelper.IdentityRoleConstants
public static bool IsProtectedSystemRole(string? roleName)
{
    return !string.IsNullOrEmpty(roleName) &&
           ProtectedSystemRoles.Contains(roleName);
}

// In AuthorizationRoleService.UpdateRoleAsync()
if (ConstantsHelper.IdentityRoleConstants.IsProtectedSystemRole(roleWithId.Name))
    return UnprocessableEntity<IdentityRole>
        (localizer[SharedResourcesKeys.Authorization.CannotModifySystemRole]);

// In AuthorizationRoleService.DeleteRoleAsync()
if (ConstantsHelper.IdentityRoleConstants.IsProtectedSystemRole(roleWithId.Name))
    return UnprocessableEntity<string>
        (localizer[SharedResourcesKeys.Authorization.CannotDeleteSystemRole]);
```

### Role Seeding and Validation

**Seeding Process:**
```csharp
// At application startup
await RoleSeeding.SeedHostRolesAsync(hostRoleManager);    // Seeds SystemAdmin
await RoleSeeding.SeedTenantRolesAsync(tenantRoleManager); // Seeds Admin, User

// Each method:
foreach (var roleName in HostRoles) // or TenantRoles
{
    var roleExists = await roleManager.RoleExistsAsync(roleName);
    if (!roleExists)
    {
        await roleManager.CreateAsync(new IdentityRole { Name = roleName });
    }
}
```

**Validation Process:**
```csharp
// At application startup
await RoleSeeding.ValidateHostRolesAsync(hostRoleManager, logger);
await RoleSeeding.ValidateTenantRolesAsync(tenantRoleManager, logger);

// Each method:
var dbRoles = await roleManager.Roles
    .Where(r => HostRoles.Contains(r.Name!))
    .Select(r => r.Name!)
    .ToListAsync();

var missingRoles = HostRoles.Except(dbRoles).ToList();

if (missingRoles.Any())
{
    throw new InvalidOperationException(
        $"Missing roles in database: {string.Join(", ", missingRoles)}");
}
```

## Design Decisions

### Why Centralized List?

- **Single Source of Truth**: All protection rules defined in one place (`ConstantsHelper`)
- **Consistency**: Same protection rules apply everywhere (tenant and host)
- **Maintainability**: Adding/removing protected roles requires only one code change
- **Type Safety**: Constants prevent typos and enable compile-time checking

### Why Service Layer Enforcement?

- **Separation of Concerns**: Controllers remain thin, business logic in services
- **Reusability**: Same service used by multiple controllers (tenant and host)
- **Testability**: Service logic can be unit tested independently
- **Consistency**: All role operations go through the same protection checks

### Why Both Tenant and Host Protection?

- **Multi-Tenant Architecture**: System has both tenant databases and host database
- **Consistent Security**: Same protection rules should apply regardless of context
- **Prevent Confusion**: Administrators shouldn't be able to delete protected roles in one context but not another
- **Code Reuse**: Same `IsProtectedSystemRole()` method works for both contexts

### Why Seeding Validation?

- **Startup Safety**: Application fails fast if protected roles are missing
- **Database Integrity**: Ensures database state matches code expectations
- **Early Detection**: Catches issues before they cause runtime errors
- **Documentation**: Validation code documents which roles are required

### Why UnprocessableEntity (422) Instead of Forbidden (403)?

- **Semantic Correctness**: 422 indicates the request is well-formed but semantically invalid
- **User Experience**: Clear error message explains why the operation failed
- **Distinction**: Different from authorization failures (403) - user is authorized but operation is invalid

## Security Considerations

### Preventing Privilege Escalation

Protected roles prevent attackers from:
- **Deleting Admin Role**: Locking out legitimate administrators
- **Renaming SystemAdmin**: Bypassing authorization checks that reference role names
- **Creating Confusion**: Using similar role names to exploit hardcoded checks

### Defense in Depth

Protection is enforced at multiple levels:
1. **Service Layer**: `AuthorizationRoleService` checks before operations
2. **Database Constraints**: (Optional) Could add database-level constraints
3. **Code Constants**: Role names referenced via constants, not strings
4. **Seeding Validation**: Startup validation ensures roles exist

### Audit Trail

All role operations should be logged:
- Who attempted to modify/delete a protected role
- When the attempt occurred
- What role was targeted
- Whether the operation succeeded or was blocked

## Integration with Existing Systems

### Tenant Role Management

**AuthorizationController** manages tenant roles:
```csharp
[Authorize(Roles = ConstantsHelper.IdentityRoleConstants.AdminRole)]
[Authorize(Policy = PoliciesHelper.CreateRoleClaim)]
[HttpPost(CustomRouter.AuthRouting.CreateRole)]
public async Task<IActionResult> CreateRole([FromBody] CreateRoleCommand command)
{
    var response = await Mediator.Send(command);
    return ReturnControllerResponse(response);
}
```

**Handler** uses `AuthorizationRoleService`:
```csharp
public class UpdateRoleCommandHandler : IRequestHandler<UpdateRoleCommand, GenericResponse<IdentityRole>>
{
    private readonly IAuthorizationRoleService roleService;
    
    public async Task<GenericResponse<IdentityRole>> Handle(...)
    {
        return await roleService.UpdateRoleAsync(
            request.Id, 
            request.Name, 
            roleManager);
    }
}
```

### Host Role Management

**HostRolesController** manages host roles:
```csharp
[Authorize(Roles = ConstantsHelper.IdentityRoleConstants.SystemAdminRole)]
[HttpPut(CustomRouter.HostRolesRouting.UpdateRole)]
public async Task<IActionResult> UpdateRole([FromBody] UpdateHostRoleCommand command)
{
    var response = await Mediator.Send(command);
    return ReturnControllerResponse(response);
}
```

Uses the same `AuthorizationRoleService`, ensuring consistent protection.

### Role Seeding Integration

**Application Startup** (`Program.cs` or `Startup.cs`):
```csharp
// Seed host roles
using (var scope = app.Services.CreateScope())
{
    var hostRoleManager = scope.ServiceProvider.GetRequiredService<RoleManager<IdentityRole>>();
    await RoleSeeding.SeedHostRolesAsync(hostRoleManager);
    await RoleSeeding.ValidateHostRolesAsync(hostRoleManager, logger);
}

// Seed tenant roles (per tenant database)
// Called when tenant database is initialized
await RoleSeeding.SeedTenantRolesAsync(tenantRoleManager);
await RoleSeeding.ValidateTenantRolesAsync(tenantRoleManager, logger);
```

## Benefits

1. **System Stability**: Prevents accidental deletion of critical roles
2. **Security**: Prevents privilege escalation and role manipulation attacks
3. **Consistency**: Same protection rules across tenant and host contexts
4. **Maintainability**: Centralized list makes it easy to add/remove protected roles
5. **Type Safety**: Constants prevent typos and enable refactoring
6. **Early Detection**: Startup validation catches missing roles before runtime errors
7. **Clear Error Messages**: Users understand why operations fail
8. **Code Safety**: Hardcoded role references remain valid

## Limitations & Considerations

1. **Code Changes Required**: Adding/removing protected roles requires code changes and application restart. No runtime configuration.

2. **Database-Level Protection**: Protection is enforced at application layer only. Direct database access could bypass protection (though this is a general security concern, not specific to this feature).

3. **Role Name Changes**: Changing a protected role name requires:
   - Adding new role to protected list
   - Migrating users from old role to new role
   - Updating all code references
   - Removing old role from protected list
   - Deleting old role from database

4. **No Granular Permissions**: All protected roles are equally protected. Cannot protect some operations (update) but allow others (delete) for the same role.

5. **No Role Hierarchy**: System doesn't support role hierarchies (e.g., "Admin" inherits from "User"). All roles are independent.

6. **Startup Dependency**: Application startup depends on protected roles existing. If roles are manually deleted from database, application will fail to start (by design).

7. **Migration Complexity**: Changing protected role names requires careful migration to avoid breaking existing user assignments.

## Future Enhancements

Potential improvements:

1. **Runtime Configuration**: Allow protected roles to be configured via database or configuration file
2. **Granular Protection**: Protect specific operations (update vs delete) per role
3. **Role Hierarchies**: Support role inheritance and hierarchy-based protection
4. **Audit Logging**: Enhanced logging for protected role operation attempts
5. **Protection Levels**: Different protection levels (e.g., "Cannot delete" vs "Cannot modify")
6. **Temporary Protection Override**: Allow temporary override for maintenance (with proper authorization)
7. **Role Templates**: Predefined role templates that include protection rules
8. **Protection Metadata**: Store protection rules in database for easier management

## Code Examples

### Checking if Role is Protected

```csharp
// In service or handler
if (ConstantsHelper.IdentityRoleConstants.IsProtectedSystemRole(roleName))
{
    return UnprocessableEntity<string>(
        localizer[SharedResourcesKeys.Authorization.CannotDeleteSystemRole]);
}
```

### Adding a New Protected Role

```csharp
// 1. Add constant
public static class IdentityRoleConstants
{
    public const string TeacherRole = "Teacher";
}

// 2. Add to protected list
public static readonly IReadOnlyList<string> ProtectedSystemRoles = new[]
{
    SystemAdminRole,
    AdminRole,
    UserRole,
    TeacherRole // New protected role
};

// 3. Update RoleSeeding (if needed)
private static readonly IReadOnlyList<string> TenantRoles = new[]
{
    AdminRole,
    UserRole,
    TeacherRole // Add to appropriate list (HostRoles or TenantRoles)
};
```

### Service Implementation

```csharp
public async Task<GenericResponse<IdentityRole>> UpdateRoleAsync(
    string id,
    string name,
    RoleManager<IdentityRole> roleManager,
    CancellationToken cancellationToken = default)
{
    // Check if role exists
    var roleWithId = await roleManager.FindByIdAsync(id);
    if (roleWithId == null)
        return NotFound<IdentityRole>(localizer[SharedResourcesKeys.NotFound]);

    // PROTECTION CHECK
    if (ConstantsHelper.IdentityRoleConstants.IsProtectedSystemRole(roleWithId.Name))
        return UnprocessableEntity<IdentityRole>
            (localizer[SharedResourcesKeys.Authorization.CannotModifySystemRole]);

    // Continue with update...
    roleWithId.Name = name;
    var result = await roleManager.UpdateAsync(roleWithId);
    
    return Success(roleWithId);
}
```

## Related Files

- **Core Implementation**: 
  - `Alforqan.Shared/HelperClasses/ConstantsHelper.cs` - Protected roles list and `IsProtectedSystemRole()` method
- **Service Layer**: 
  - `Alforqan.Service/Implementation/Identity/Authorization/AuthorizationRoleService.cs` - Protection enforcement
- **Seeding**: 
  - `Alforqan.Infrastructure/Seeding/RoleSeeding.cs` - Role seeding and validation
- **Controllers**: 
  - `Alforqan.Api/Controllers/Identity/Authorization/AuthorizationController.cs` - Tenant role management
  - `Alforqan.Api/Controllers/Host/Roles/HostRolesController.cs` - Host role management
- **Technical Guide**: 
  - `docs/technical-docs/protected-system-roles-guide.md` - Step-by-step usage guide

