# Controller & API Guide

## How to Create a Standard CRUD Controller

```csharp
using Microsoft.AspNetCore.Mvc;
using Alforqan.Core.Features.Students.Shared.InputDtos;
using Alforqan.Core.Features.Students.Shared.ResponseDtos;
using Alforqan.Data.AppMetaData;
using Alforqan.Shared.Enums.Students;
using Alforqan.Shared.Responses;

namespace Alforqan.Api.Controllers.Students
{
    [Route($"{CustomRouter._rule}/[controller]")]
    public class StudentsController
        : AppControllerBase<StudentDto, int, CreateUpdateStudentDto, enStudentOrdering>
    {
    }
}
```

Automatically provides:
- `POST /List` - Get paginated, sorted, filtered list
- `GET /{id}` - Get entity by ID
- `POST /create` - Create new entity
- `PUT /{id}` - Update entity
- `DELETE /{id}` - Delete entity

## How to Create a Custom Controller

```csharp
using MediatR;
using Microsoft.AspNetCore.Mvc;
using Alforqan.Api.Controllers.Shared;
using Alforqan.Core.Features.Identity.Authentication.Commands.Models;
using Alforqan.Data.AppMetaData;

namespace Alforqan.Api.Controllers.Identity.Authentication
{
    [Route($"{CustomRouter._rule}/[controller]")]
    [ApiController]
    public class AuthenticationController : ControllerResponse
    {
        #region private fields
        private readonly IMediator mediator;
        #endregion

        #region constructors
        public AuthenticationController(IMediator mediator)
        {
            this.mediator = mediator;
        }
        #endregion

        #region end points
        [HttpPost($"{CustomRouter.AuthRouting.Login}")]
        public async Task<IActionResult> Login([FromBody] LoginCommand command)
        {
            var response = await mediator.Send(command);
            return ReturnControllerResponse(response);
        }
        #endregion
    }
}
```

## How to Add Custom Endpoints to AppControllerBase

```csharp
[Route($"{CustomRouter._rule}/[controller]")]
public class TenantsController
    : AppControllerBase<TenantDto, Guid, CreateUpdateTenantDto, enTenantOrdering>
{
    #region end points
    
    [HttpGet("by-domain/{domain}")]
    public async Task<IActionResult> GetByDomain([FromRoute] string domain)
    {
        var response = await Mediator.Send(new GetTenantByDomainQuery(domain));
        return ReturnControllerResponse(response);
    }
    
    #endregion
}
```

## Controller Structure

Use regions to organize code:

```csharp
#region private fields
private readonly IService _service;
#endregion

#region constructors
public MyController(IService service)
{
    _service = service;
}
#endregion

#region end points
// Endpoints here
#endregion
```

## How to Set Route

Always use:
```csharp
[Route($"{CustomRouter._rule}/[controller]")]
```

This ensures consistent routing: `API/V01/ControllerName`

## How to Add Authorization

Use separate `[Authorize]` attributes - one for Roles, one for Policy:

```csharp
[Authorize(Roles = ConstantsHelper.IdentityRoleConstants.AdminRole)]
[Authorize(Policy = PoliciesHelper.TenantManagementClaim)]
[Route($"{CustomRouter._rule}/[controller]")]
public class TenantsController : AppControllerBase<...>
{
}
```

DO NOT combine them:
```csharp
// ❌ WRONG
[Authorize(Policy = PoliciesHelper.TenantManagementClaim, Roles = ConstantsHelper.IdentityRoleConstants.AdminRole)]

// ✅ CORRECT
[Authorize(Roles = ConstantsHelper.IdentityRoleConstants.AdminRole)]
[Authorize(Policy = PoliciesHelper.TenantManagementClaim)]
```

## How to Add Controller-Level Authorization

```csharp
[Authorize(Roles = ConstantsHelper.IdentityRoleConstants.AdminRole)]
[Authorize(Policy = PoliciesHelper.TenantManagementClaim)]
[Route($"{CustomRouter._rule}/[controller]")]
public class TenantsController : AppControllerBase<...>
{
}
```

## How to Add Action-Level Authorization

```csharp
[Authorize(Roles = ConstantsHelper.IdentityRoleConstants.AdminRole)]
[Authorize(Policy = PoliciesHelper.CreateUserClaim)]
[HttpPost($"{CustomRouter.BaseRouting.Create}")]
public async Task<IActionResult> Create([FromBody] CreateDto dto)
{
    var response = await Mediator.Send(new CreateCommand(dto));
    return ReturnControllerResponse(response);
}
```

## How to Implement Endpoints

1. Use `Mediator.Send()` to send commands/queries:
   ```csharp
   var response = await Mediator.Send(new GetTenantByDomainQuery(domain));
   ```

2. Use `ReturnControllerResponse()` to return responses:
   ```csharp
   return ReturnControllerResponse(response);
   ```

DO NOT create `GenericResponse` directly in controllers.

## How to Use Route Parameters

- `[FromRoute]` - For route parameters: `{id}`, `{domain}`, etc.
- `[FromBody]` - For JSON request bodies
- `[FromForm]` - For form data (multipart/form-data)

```csharp
[HttpGet("by-domain/{domain}")]
public async Task<IActionResult> GetByDomain([FromRoute] string domain)
{
    // ...
}

[HttpPost("create")]
public async Task<IActionResult> Create([FromBody] CreateDto dto)
{
    // ...
}

[HttpPost("upload")]
public async Task<IActionResult> Upload([FromForm] UploadCommand command)
{
    // ...
}
```

## How to Define Custom Routes

1. Add route constants to `Alforqan.Data/AppMetaData/CustomRouter.cs`:
   ```csharp
   public static class HostRouting
   {
       public static class HostTenantsRouting
       {
           public const string GetByDomain = "by-domain/{domain}";
           public const string GetByCode = "by-code/{code}";
           public const string EnableModule = "{tenantId}/modules/{moduleKey}/enable";
       }
   }
   ```

2. Use route constants in controller:
   ```csharp
   // ✅ CORRECT
   [HttpGet(CustomRouter.HostRouting.HostTenantsRouting.GetByDomain)]
   public async Task<IActionResult> GetByDomain([FromRoute] string domain)
   {
       // ...
   }

   // ❌ WRONG
   [HttpGet("by-domain/{domain}")]
   public async Task<IActionResult> GetByDomain([FromRoute] string domain)
   {
       // ...
   }
   ```

## Authorization Constants

- **Policies**: Use `PoliciesHelper.ClaimName` (e.g., `PoliciesHelper.TenantManagementClaim`)
- **Roles**: Use `ConstantsHelper.IdentityRoleConstants.RoleName` (e.g., `ConstantsHelper.IdentityRoleConstants.AdminRole`)

## Files to Reference

- Base Classes: `Alforqan.Api/Controllers/Shared/AppControllerBase.cs`, `ControllerResponse.cs`
- Router: `Alforqan.Data/AppMetaData/CustomRouter.cs`
- Examples: `StudentsController.cs`, `TenantsController.cs`, `AuthenticationController.cs`
