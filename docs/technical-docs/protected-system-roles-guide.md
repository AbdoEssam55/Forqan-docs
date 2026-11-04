# Protected System Roles Guide

## Overview

Protected system roles are roles that cannot be modified or deleted through the API. This protection applies to **both tenant roles and host roles**. The `IsProtectedSystemRole()` method works identically for both contexts, ensuring consistent protection across the system.

## How to Add a New Protected System Role

1. Open `Alforqan.Shared/HelperClasses/ConstantsHelper.cs`

2. Add role constant in `IdentityRoleConstants`:
   ```csharp
   public const string TeacherRole = "Teacher";
   ```

3. Add role to `ProtectedSystemRoles` list:
   ```csharp
   public static readonly IReadOnlyList<string> ProtectedSystemRoles = new[]
   {
       SystemAdminRole,
       AdminRole,
       UserRole,
       TeacherRole // Add here
   };
   ```

4. Restart application - role will be automatically seeded and validated

## How to Remove a Protected System Role

1. Open `Alforqan.Shared/HelperClasses/ConstantsHelper.cs`

2. Remove role from `ProtectedSystemRoles` list:
   ```csharp
   public static readonly IReadOnlyList<string> ProtectedSystemRoles = new[]
   {
       SystemAdminRole,
       AdminRole,
       UserRole
       // TeacherRole removed
   };
   ```

3. (Optional) Keep constant definition if used elsewhere in codebase

4. Restart application - role is no longer protected

## How to Change a Protected Role Name

1. Add new role following "How to Add a New Protected System Role"

2. Migrate users from old role to new role:
   ```sql
   UPDATE AspNetUserRoles 
   SET RoleId = (SELECT Id FROM AspNetRoles WHERE Name = 'NewRoleName')
   WHERE RoleId = (SELECT Id FROM AspNetRoles WHERE Name = 'OldRoleName')
   ```

3. Update all code references to use new role constant

4. Remove old role following "How to Remove a Protected System Role"

5. Delete old role from `AspNetRoles` table

## How to Check if a Role is Protected

Use helper method in code:
```csharp
if (ConstantsHelper.IdentityRoleConstants.IsProtectedSystemRole(roleName))
{
    // Role is protected
}
```

## Tenant vs Host Roles

Protected system roles apply to **both tenant and host role management**:

- **Tenant Roles**: Protected roles cannot be modified/deleted via `AuthorizationController` endpoints
- **Host Roles**: Protected roles cannot be modified/deleted via `HostRolesController` endpoints

Both use the same `ConstantsHelper.IdentityRoleConstants.IsProtectedSystemRole()` method, ensuring consistent protection rules across contexts.

## Files to Modify

- `Alforqan.Shared/HelperClasses/ConstantsHelper.cs` - Add/remove roles from `ProtectedSystemRoles` list
