# Controller & API Architecture

## Overview

The controller layer in Alforqan follows a thin, delegation-based architecture that integrates seamlessly with the CQRS pattern using MediatR. Controllers act as HTTP endpoints that delegate all business logic to handlers, services, and builders, ensuring clear separation of concerns and maintainability.

## Architecture Overview

### Controller Layer Responsibilities

The controller layer has minimal responsibilities:

1. **HTTP Request/Response Handling**: Receives HTTP requests and returns HTTP responses
2. **Authorization Enforcement**: Applies authorization policies and roles
3. **Request Delegation**: Sends commands/queries to MediatR
4. **Response Mapping**: Maps handler responses to appropriate HTTP status codes

**What controllers should NOT do:**
- Business logic
- Data validation (handled by FluentValidation)
- Data access (handled by repositories/services)
- Response creation (handled by handlers/services)

### Integration with CQRS Pattern

Controllers integrate with the CQRS pattern through MediatR:

```
HTTP Request → Controller → MediatR → Handler → Service/Builder → Repository → Database
                                                                    ↓
HTTP Response ← Controller ← MediatR ← Handler ← Service/Builder ← Repository
```

**Flow:**
1. Controller receives HTTP request
2. Controller creates command/query object
3. Controller sends to MediatR via `Mediator.Send()`
4. MediatR routes to appropriate handler
5. Handler executes business logic (via services/builders)
6. Handler returns `GenericResponse<T>`
7. Controller maps response to HTTP status code via `ReturnControllerResponse()`
8. Controller returns HTTP response

### Response Handling via GenericResponse<T>

All handlers return `GenericResponse<T>` which contains:
- `Succeeded` - Boolean indicating success/failure
- `StatusCode` - HTTP status code (OK, Created, BadRequest, NotFound, etc.)
- `Message` - Human-readable message
- `Data` - Response data (if successful)
- `Errors` - List of error messages
- `ErrorsBag` - Dictionary of field-specific errors

Controllers use `ReturnControllerResponse()` to map `GenericResponse<T>` to appropriate HTTP responses:

```csharp
protected IActionResult ReturnControllerResponse<TDataType>(GenericResponse<TDataType> handlerResponse)
{
    switch (handlerResponse.StatusCode)
    {
        case HttpStatusCode.OK:
            return Ok(handlerResponse);
        case HttpStatusCode.Created:
            return Created(String.Empty, handlerResponse);
        case HttpStatusCode.BadRequest:
            return BadRequest(handlerResponse);
        case HttpStatusCode.NotFound:
            return NotFound(handlerResponse);
        // ... etc
    }
}
```

### Authorization Strategy

Alforqan uses a **claims-based + role-based** authorization strategy:

1. **Roles**: Broad access control (Admin, User, etc.)
2. **Claims/Policies**: Fine-grained permissions (CreateUserClaim, TenantManagementClaim, etc.)

**Policy Registration:**
Policies are registered in `ModuleSharedDependencies.cs`:

```csharp
services.AddAuthorization(options =>
{
    foreach (var claim in PoliciesHelper.ClaimsList)
    {
        options.AddPolicy(claim.Type, policy => policy.RequireClaim(claim.Type, claim.Value));
    }
});
```

**Authorization Pattern:**
Controllers use separate `[Authorize]` attributes for roles and policies:

```csharp
[Authorize(Roles = ConstantsHelper.IdentityRoleConstants.AdminRole)]
[Authorize(Policy = PoliciesHelper.TenantManagementClaim)]
```

This ensures users must have BOTH the role AND the policy claim.

## Design Decisions

### Why AppControllerBase for CRUD?

**Problem**: Standard CRUD operations (List, GetById, Create, Edit, Delete) are repetitive across controllers.

**Solution**: `AppControllerBase<TDto, TKey, TCreateUpdateDto, TOrderingEnum>` provides:
- **Reduced boilerplate**: No need to implement standard CRUD endpoints
- **Consistency**: All CRUD operations follow the same pattern
- **Type safety**: Generic parameters ensure correct types
- **Automatic routing**: Uses `CustomRouter.BaseRouting` constants

**Trade-offs:**
- Less flexibility for non-standard CRUD operations
- Requires entities to follow standard patterns

**When to use**: When you have standard CRUD operations that match the base class pattern.

### Why ControllerResponse for Custom?

**Problem**: Not all controllers need standard CRUD operations. Some need custom endpoints (authentication, authorization, reporting, etc.).

**Solution**: `ControllerResponse` provides:
- **Full flexibility**: Complete control over endpoints
- **No assumptions**: Doesn't assume CRUD pattern
- **Lightweight**: Minimal base class functionality

**Trade-offs:**
- More code to write for each endpoint
- Less consistency across controllers

**When to use**: When you need custom endpoints that don't follow CRUD pattern.

### Why Separate [Authorize] Attributes?

**Problem**: ASP.NET Core allows combining Roles and Policy in one attribute, but this can be less readable.

**Solution**: Use separate attributes:
- **Readability**: Clear separation of role and policy requirements
- **Consistency**: Matches existing codebase pattern
- **Flexibility**: Easy to add/remove role or policy independently

**Example:**
```csharp
// ✅ Clear and consistent
[Authorize(Roles = ConstantsHelper.IdentityRoleConstants.AdminRole)]
[Authorize(Policy = PoliciesHelper.TenantManagementClaim)]

// ❌ Less readable
[Authorize(Policy = PoliciesHelper.TenantManagementClaim, Roles = ConstantsHelper.IdentityRoleConstants.AdminRole)]
```

### Why CustomRouter?

**Problem**: Hardcoded route strings are error-prone and difficult to refactor.

**Solution**: `CustomRouter` provides:
- **Centralized management**: All routes in one place
- **Type safety**: Compile-time checking of route constants
- **Easy refactoring**: Change route in one place, affects all usages
- **Consistency**: Ensures consistent route patterns

**Structure:**
```csharp
public class CustomRouter
{
    public const string _rule = "API/V01";
    
    public static class BaseRouting
    {
        public const string List = "List";
        public const string GetById = "{id}";
        // ...
    }
    
    public static class HostRouting
    {
        public static class HostTenantsRouting
        {
            public const string GetByDomain = "by-domain/{domain}";
            // ...
        }
    }
}
```

## Pattern Flow

### Request Flow

```
1. HTTP Request arrives at Controller
   ↓
2. Controller validates authorization (if [Authorize] present)
   ↓
3. Controller creates Command/Query object from request
   ↓
4. Controller sends to MediatR: Mediator.Send(command/query)
   ↓
5. MediatR routes to appropriate Handler
   ↓
6. FluentValidation validates command/query (automatic via pipeline)
   ↓
7. Handler executes business logic:
   - Calls Service methods
   - Or uses Builder pattern for complex operations
   ↓
8. Service/Builder calls Repository for data access
   ↓
9. Repository queries/updates Database
   ↓
10. Response flows back: Repository → Service/Builder → Handler
   ↓
11. Handler returns GenericResponse<T>
   ↓
12. Controller maps GenericResponse to HTTP response via ReturnControllerResponse()
   ↓
13. HTTP Response returned to client
```

### Error Handling Flow

```
1. Error occurs (validation, business rule, database, etc.)
   ↓
2. Handler/Service/Builder creates GenericResponse with:
   - Succeeded = false
   - StatusCode = appropriate HTTP code (BadRequest, NotFound, etc.)
   - Message = error message
   - Errors/ErrorsBag = detailed error information
   ↓
3. GenericResponse flows back through the chain
   ↓
4. Controller receives GenericResponse with error
   ↓
5. ReturnControllerResponse() maps StatusCode to HTTP response
   ↓
6. Client receives HTTP error response with error details
```

### Authorization Check Flow

```
1. HTTP Request arrives at Controller
   ↓
2. ASP.NET Core Authorization Middleware checks [Authorize] attributes
   ↓
3. For Roles: Checks if user has required role
   ↓
4. For Policy: Checks if user has required claim
   ↓
5. If both pass: Request proceeds to controller action
   ↓
6. If either fails: Returns 401 Unauthorized or 403 Forbidden
```

## Integration Points

### MediatR Integration

Controllers integrate with MediatR through the `Mediator` property (inherited from `ControllerResponse`):

```csharp
protected IMediator Mediator => _mediatorInstance ??= HttpContext.RequestServices.GetService<IMediator>();
```

**Usage:**
```csharp
var response = await Mediator.Send(new GetTenantByDomainQuery(domain));
```

**Benefits:**
- Loose coupling: Controller doesn't know about handler implementation
- Testability: Easy to mock MediatR
- Pipeline support: Automatic validation, logging, etc.

### FluentValidation Integration

FluentValidation is integrated via MediatR pipeline behavior:

```csharp
services.AddScoped(typeof(IPipelineBehavior<,>), typeof(ValidationBehavior<,>));
```

**Flow:**
1. Controller sends command/query to MediatR
2. ValidationBehavior intercepts before handler
3. Validator validates command/query
4. If validation fails: Returns `GenericResponse` with validation errors
5. If validation passes: Proceeds to handler

**Controllers don't need to handle validation** - it's automatic.

### AutoMapper Integration

AutoMapper is used in handlers and builders, not controllers:

```csharp
// In Handler or Builder
_tenantDto = _mapper.Map<TenantDto>(_tenant);
```

**Why not in controllers?**
- Keeps controllers thin
- Business logic (mapping rules) belongs in handlers/builders
- Easier to test

### Service Layer Integration

Controllers don't directly call services. Instead:
1. Controller sends command/query to MediatR
2. Handler receives command/query
3. Handler calls service methods
4. Service returns `GenericResponse<T>`
5. Handler returns response to controller

**Benefits:**
- Controllers stay thin
- Business logic encapsulated in services
- Easy to test (mock services in handlers)

## Best Practices

### Keep Controllers Thin

Controllers should only:
- Receive HTTP requests
- Validate authorization
- Delegate to MediatR
- Return HTTP responses

**Don't:**
- Perform business logic
- Access database directly
- Create entities/DTOs manually
- Handle validation logic

### Use DTOs for Input/Output

Always use DTOs (Data Transfer Objects) for:
- Request bodies (`[FromBody]`)
- Response data
- Route parameters (when complex)

**Never expose entities directly** - this breaks encapsulation and can cause security issues.

### Follow Naming Conventions

- **Controllers**: Plural noun + "Controller" (e.g., `StudentsController`, `TenantsController`)
- **Actions**: Verb + noun (e.g., `GetByDomain`, `EnableModule`)
- **Routes**: kebab-case (e.g., `by-domain`, `enable-module`)

### Security Considerations

1. **Always authorize**: Use `[Authorize]` on protected endpoints
2. **Principle of least privilege**: Only grant minimum required permissions
3. **Validate input**: Let FluentValidation handle validation
4. **Never trust client**: Always validate IDs, tenant codes, etc.
5. **Use HTTPS**: In production, always use HTTPS
6. **Log security events**: Log authorization failures, suspicious activity

### Error Handling

Controllers don't need try-catch blocks for business logic errors:
- Handlers/services return `GenericResponse` with error information
- `ReturnControllerResponse()` maps errors to appropriate HTTP status codes
- Only catch exceptions for unexpected errors (logging, graceful degradation)

### Testing Considerations

Controllers are easy to test because they're thin:
- Mock `IMediator` to test request/response flow
- Test authorization attributes
- Test route mappings
- Business logic is tested in handlers/services

## Related Files

- `Alforqan.Api/Controllers/Shared/AppControllerBase.cs` - Base class for CRUD controllers
- `Alforqan.Api/Controllers/Shared/ControllerResponse.cs` - Base class for custom controllers
- `Alforqan.Data/AppMetaData/CustomRouter.cs` - Route constants
- `Alforqan.Shared/HelperClasses/PoliciesHelper.cs` - Policy constants
- `Alforqan.Shared/HelperClasses/ConstantsHelper.cs` - Role constants
- `Alforqan.Shared/Responses/GenericResponse.cs` - Response wrapper
- `Alforqan.Shared/ModuleSharedDependencies.cs` - Policy registration

