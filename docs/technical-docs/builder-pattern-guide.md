# Builder Pattern Guide

## How to Create an AppDbContext Builder

```csharp
public class MyCommandBuilder
    : AppBuilder<MyCommand, MyResponseDto>, IMyCommandBuilder
{
    public MyCommandBuilder(IServiceProvider serviceProvider)
        : base(serviceProvider)
    {
        // UseTransaction = true (default for AppBuilder)
    }
}
```

## How to Create a HostDbContext Builder

```csharp
public class MyQueryBuilder
    : HostBuilder<MyQuery, MyResponseDto>, IMyQueryBuilder
{
    public MyQueryBuilder(IServiceProvider serviceProvider)
        : base(serviceProvider)
    {
        // UseTransaction = false (default for HostBuilder)
    }
}
```

## How to Enable Transaction for HostDbContext Commands

```csharp
public class MyHostCommandBuilder
    : HostBuilder<MyCommand, MyResponseDto>, IMyHostCommandBuilder
{
    public MyHostCommandBuilder(IServiceProvider serviceProvider)
        : base(serviceProvider)
    {
        UseTransaction = true; // Override default
    }
}
```

## How to Override Transaction Behavior

Set `UseTransaction` property in constructor:

```csharp
public MyBuilder(IServiceProvider serviceProvider)
    : base(serviceProvider)
{
    UseTransaction = true;  // Enable transaction
    // or
    UseTransaction = false; // Disable transaction
}
```

## How to Inject Dependencies

Do NOT inject DbContext directly:

```csharp
// ❌ Bad
public MyBuilder(IServiceProvider serviceProvider, HostDbContext hostDbContext)
    : base(serviceProvider)
{
    _hostDbContext = hostDbContext;
}
```

DO inject services or repositories:

```csharp
// ✅ Good
public MyBuilder(IServiceProvider serviceProvider, ITenantService tenantService)
    : base(serviceProvider)
{
    _tenantService = tenantService;
}
```

## How to Handle Validation

Do NOT validate in builders:

```csharp
// ❌ Bad
public async Task<GenericResponse<MyDto>> A_ValidateInput()
{
    if (string.IsNullOrWhiteSpace(_input.Code))
    {
        return BadRequest<MyDto>(localizer[SharedResourcesKeys.Required]);
    }
    return null;
}
```

DO use FluentValidation validators:

Create validator in `Alforqan.Core/Features/<Area>/Queries|Commands/Validators/`:

```csharp
public class MyQueryValidator : AbstractValidator<MyQuery>
{
    public MyQueryValidator()
    {
        RuleFor(x => x.Code)
            .NotEmpty()
            .NotNull();
    }
}
```

Validation happens automatically via FluentValidation pipeline.

## Builder Step Methods

- Return `null` to continue to next step
- Return `GenericResponse<T>` to stop execution and return that response
- Use helper methods: `_ReturnNullTask()`, `_ReturnSuccessTask(data)`, `_ReturnNotFoundTask(message)`, etc.

**Important Transaction Behavior:**
- When `UseTransaction = true`, the transaction commits **only** if the final response has `Succeeded == true`
- Failed responses (BadRequest, NotFound, UnprocessableEntity, etc. where `Succeeded == false`) automatically rollback the transaction
- Exceptions also trigger rollback
- Transactions are wrapped in execution strategy for SQL Server retry support
- Transactions are properly disposed using `await using`

## Query Builder Example

```csharp
public class GetEntityQueryBuilder
    : HostBuilder<GetEntityQuery, EntityDto>, IGetEntityQueryBuilder
{
    private readonly IEntityService _entityService;
    private readonly IMapper _mapper;
    private Entity? _entity;
    private EntityDto? _entityDto;

    public GetEntityQueryBuilder(IServiceProvider serviceProvider, 
        IEntityService entityService, IMapper mapper)
        : base(serviceProvider)
    {
        _entityService = entityService;
        _mapper = mapper;
    }

    public async Task<GenericResponse<EntityDto>> B_GetEntity()
    {
        var response = await _entityService.GetByIdAsync(_input.Id);
        if (!response.IsSuccess)
        {
            return response.IsNotFound 
                ? NotFound<EntityDto>(localizer[SharedResourcesKeys.NotFound])
                : BadRequest<EntityDto>(response.Message ?? localizer[SharedResourcesKeys.Fail]);
        }
        _entity = response.Data;
        return null;
    }

    public Task<GenericResponse<EntityDto>> C_MapToDto()
    {
        _entityDto = _mapper.Map<EntityDto>(_entity);
        return _ReturnNullTask();
    }

    public Task<GenericResponse<EntityDto>> D_ReturnResponse()
    {
        return _ReturnSuccessTask(_entityDto!);
    }
}
```

## Command Builder Example

```csharp
public class CreateEntityCommandBuilder
    : AppBuilder<CreateEntityCommand, EntityDto>, ICreateEntityCommandBuilder
{
    private readonly IEntityService _entityService;
    private readonly IMapper _mapper;
    private Entity? _entity;
    private EntityDto? _entityDto;

    public CreateEntityCommandBuilder(IServiceProvider serviceProvider,
        IEntityService entityService, IMapper mapper)
        : base(serviceProvider)
    {
        _entityService = entityService;
        _mapper = mapper;
        // UseTransaction = true (default for AppBuilder)
    }

    public async Task<GenericResponse<EntityDto>> B_CreateEntity()
    {
        _entity = _mapper.Map<Entity>(_input);
        var response = await _entityService.AddAsync(_entity);
        if (!response.IsSuccess)
        {
            return BadRequest<EntityDto>(response.Message ?? localizer[SharedResourcesKeys.Fail]);
        }
        return null;
    }

    public Task<GenericResponse<EntityDto>> C_MapToDto()
    {
        _entityDto = _mapper.Map<EntityDto>(_entity);
        return _ReturnNullTask();
    }

    public Task<GenericResponse<EntityDto>> D_ReturnResponse()
    {
        return _ReturnSuccessTask(_entityDto!);
    }
}
```

## Files to Reference

- Base Classes: `Alforqan.Core/Builders/Implementations/Base/Builder.cs`, `AppBuilder.cs`, `HostBuilder.cs`
- Interface: `Alforqan.Core/Builders/Abstractions/Base/IBuilder.cs`
