# Builder Pattern Feature - Detailed Explanation

## Overview

The Builder Pattern in Alforqan provides a structured, step-by-step approach to executing complex business operations. It allows breaking down operations into discrete, testable steps while managing database transactions and error handling automatically.

## Problem Statement

Before the refactoring, the Builder pattern had several limitations:

1. **Hardcoded DbContext**: The base `Builder` class was hardcoded to use `AppDbContext`, making it impossible to use `HostDbContext` without duplicating code.

2. **Mandatory Transactions**: All builders used transactions, even for read-only queries where transactions are unnecessary and add overhead.

3. **Code Duplication**: Builders using `HostDbContext` had to override `BuildAsync()` to avoid using the wrong `DbContext` and to skip transactions.

4. **Direct DbContext Injection**: Builders were injecting `DbContext` directly instead of using the service/repository layer, violating separation of concerns.

5. **Validation in Builders**: Validation logic was mixed into builder steps instead of using the centralized FluentValidation system.

## Architecture & Design

### Components

1. **Builder<TInputType, TResponseType, TDbContext>** (`Alforqan.Core/Builders/Implementations/Base/Builder.cs`)
   - Generic base class accepting any `DbContext` type
   - Supports optional transactions via `UseTransaction` property
   - Extracts step execution to `ExecuteStepsAsync()` method
   - Manages transaction lifecycle when enabled

2. **AppBuilder<TInputType, TResponseType>** (`Alforqan.Core/Builders/Implementations/Base/AppBuilder.cs`)
   - Convenience class for `AppDbContext` builders
   - Transaction enabled by default (for commands)
   - Inherits from `Builder<TInputType, TResponseType, AppDbContext>`

3. **HostBuilder<TInputType, TResponseType>** (`Alforqan.Core/Builders/Implementations/Base/HostBuilder.cs`)
   - Convenience class for `HostDbContext` builders
   - Transaction disabled by default (for queries)
   - Inherits from `Builder<TInputType, TResponseType, HostDbContext>`
   - Can enable transactions for commands by setting `UseTransaction = true`

4. **FluentValidation Validators**
   - Validation moved from builder steps to dedicated validators
   - Automatic validation via MediatR pipeline
   - Consistent with existing validation patterns

## How It Works

### Flow Diagram

```
1. Handler Receives Request
   ↓
2. FluentValidation Validates Request (automatic)
   ↓
3. Handler Creates Builder and Sets Input
   ↓
4. Handler Adds Steps to Builder Chain
   ↓
5. Handler Calls BuildAsync()
   ↓
6. Builder.BuildAsync()
   ├─ If UseTransaction == true:
   │  ├─ Create Execution Strategy (for SQL Server retry support)
   │  ├─ Begin Transaction (with await using for disposal)
   │  ├─ ExecuteStepsAsync()
   │  │  ├─ Execute Step 1
   │  │  ├─ Execute Step 2
   │  │  └─ ... (until step returns non-null or all steps complete)
   │  ├─ Commit Transaction (only if result.Succeeded == true)
   │  └─ Rollback Transaction (if result.Succeeded == false or exception)
   └─ If UseTransaction == false:
      └─ ExecuteStepsAsync() (no transaction)
   ↓
7. Return GenericResponse<T>
```

### Step Execution Logic

Each step method:
- Returns `null` to continue to the next step
- Returns `GenericResponse<T>` to stop execution and return that response
- Can access `_input` (the input object) and `_dbContext` (the appropriate DbContext)

### Transaction Management

**With Transaction (UseTransaction = true)**:
```csharp
var strategy = _dbContext.Database.CreateExecutionStrategy();

return await strategy.ExecuteAsync(async () =>
{
    await using var transaction = await _dbContext.Database.BeginTransactionAsync(cancellationToken);
    try
    {
        var result = await ExecuteStepsAsync();

        // Null safety: never commit/rollback on null
        if (result is null)
        {
            await transaction.RollbackAsync(cancellationToken);
            throw new InvalidOperationException(
                "Builder execution completed without returning a response. " +
                "Last step must return a GenericResponse (e.g., ReturnResponse()).");
        }

        // Only commit on success
        if (result.Succeeded)
            await transaction.CommitAsync(cancellationToken);
        else
            await transaction.RollbackAsync(cancellationToken);

        return result;
    }
    catch
    {
        await transaction.RollbackAsync(cancellationToken);
        throw;
    }
});
```

**Key Transaction Behaviors:**
- **Commit on Success**: Transaction only commits when `result.Succeeded == true`
- **Rollback on Failure**: Transaction rolls back when `result.Succeeded == false` (BadRequest, NotFound, etc.)
- **Rollback on Exception**: Any exception triggers rollback and rethrow
- **Execution Strategy**: Wrapped in `CreateExecutionStrategy().ExecuteAsync()` for SQL Server retry support
- **Proper Disposal**: Uses `await using` to ensure transaction is always disposed

**Without Transaction (UseTransaction = false)**:
```csharp
return await ExecuteStepsAsync();
```

## Design Decisions

### Why Generic DbContext?

- **Flexibility**: Supports both `AppDbContext` and `HostDbContext` without code duplication
- **Type Safety**: Compile-time checking ensures correct `DbContext` usage
- **Extensibility**: Easy to add support for additional `DbContext` types in the future

### Why Optional Transactions?

- **Performance**: Queries don't need transactions, avoiding unnecessary overhead
- **Flexibility**: Commands can opt-out of transactions if needed (rare cases)
- **Clarity**: Makes transaction requirements explicit in code

### Why Convenience Classes (AppBuilder/HostBuilder)?

- **Ease of Use**: Developers don't need to specify `DbContext` type every time
- **Default Behavior**: Sensible defaults (transactions for AppBuilder, no transactions for HostBuilder)
- **Backward Compatibility**: Existing code can migrate gradually

### Why Remove DbContext Injection from Builders?

- **Separation of Concerns**: Builders should orchestrate business logic, not access data directly
- **Testability**: Easier to mock services than `DbContext`
- **Consistency**: Aligns with repository/service pattern used throughout the project

### Why Move Validation to FluentValidation?

- **Consistency**: All validation uses the same system
- **Automatic Execution**: Validation happens before builder execution
- **Localization**: Automatic localization via `LocalizedFluentValidationLanguageManager`
- **Separation**: Validation logic separate from business logic

## Integration with Existing Systems

### CQRS Pattern

Builders are used in CQRS handlers:

```csharp
public class GetTenantByCodeQueryHandler
    : IRequestHandler<GetTenantByCodeQuery, GenericResponse<TenantDto>>
{
    private readonly IGetTenantByCodeQueryBuilder _builder;

    public async Task<GenericResponse<TenantDto>> Handle(...)
    {
        _builder.SetBuilderInput(request);
        return await _builder
            .AddStep(_builder.B_GetTenantByCode)
            .AddStep(_builder.C_CheckTenantExists)
            .AddStep(_builder.D_MapToDto)
            .AddStep(_builder.E_ReturnResponse)
            .BuildAsync();
    }
}
```

### Service Layer

Builders use services instead of direct `DbContext` access:

```csharp
public async Task<GenericResponse<TenantDto>> B_GetTenantByCode()
{
    var tenantResponse = await _tenantService.GetTenantByCodeAsync(_input.Code);
    // Handle response...
}
```

### FluentValidation

Validation happens automatically before builder execution:

```csharp
public class GetTenantByCodeQueryValidator : AbstractValidator<GetTenantByCodeQuery>
{
    public GetTenantByCodeQueryValidator()
    {
        RuleFor(x => x.Code)
            .NotEmpty()
            .NotNull();
    }
}
```

## Benefits

1. **No Code Duplication**: Single `BuildAsync()` implementation for all builders
2. **Type Safety**: Generic `DbContext` ensures correct usage
3. **Flexibility**: Optional transactions allow optimal performance
4. **Maintainability**: Changes to transaction logic only need to be made once
5. **Testability**: Services/repos are easier to mock than `DbContext`
6. **Consistency**: All validation uses FluentValidation
7. **Clear Intent**: `AppBuilder` vs `HostBuilder` makes usage obvious

## Usage Examples

### Query Builder (No Transaction)

```csharp
public class GetTenantByCodeQueryBuilder
    : HostBuilder<GetTenantByCodeQuery, TenantDto>, IGetTenantByCodeQueryBuilder
{
    private readonly ITenantService _tenantService;
    private readonly IMapper _mapper;

    public GetTenantByCodeQueryBuilder(IServiceProvider serviceProvider,
        ITenantService tenantService, IMapper mapper)
        : base(serviceProvider)
    {
        _tenantService = tenantService;
        _mapper = mapper;
        // UseTransaction = false (default for HostBuilder)
    }

    public async Task<GenericResponse<TenantDto>> B_GetTenantByCode()
    {
        var response = await _tenantService.GetTenantByCodeAsync(_input.Code);
        // ... handle response
    }
}
```

### Command Builder (With Transaction)

```csharp
public class EnableTenantModuleCommandBuilder
    : HostBuilder<EnableTenantModuleCommand, string>, IEnableTenantModuleCommandBuilder
{
    private readonly HostDbContext _hostDbContext;

    public EnableTenantModuleCommandBuilder(IServiceProvider serviceProvider,
        HostDbContext hostDbContext)
        : base(serviceProvider)
    {
        _hostDbContext = hostDbContext;
        UseTransaction = true; // Override default for commands
    }
}
```

## Migration from Old Pattern

### Before

```csharp
public class GetTenantByCodeQueryBuilder
    : Builder<GetTenantByCodeQuery, TenantDto>, IGetTenantByCodeQueryBuilder
{
    private readonly HostDbContext _hostDbContext;

    public GetTenantByCodeQueryBuilder(IServiceProvider serviceProvider,
        HostDbContext hostDbContext, IMapper mapper)
        : base(serviceProvider)
    {
        _hostDbContext = hostDbContext;
    }

    public async Task<GenericResponse<TenantDto>> A_ValidateCode()
    {
        if (string.IsNullOrWhiteSpace(_input.Code))
        {
            return BadRequest<TenantDto>(localizer[SharedResourcesKeys.RequiredField]);
        }
        return null;
    }

    public new async Task<GenericResponse<TenantDto>> BuildAsync()
    {
        // Override without transaction
        // ... duplicate code
    }
}
```

### After

```csharp
public class GetTenantByCodeQueryBuilder
    : HostBuilder<GetTenantByCodeQuery, TenantDto>, IGetTenantByCodeQueryBuilder
{
    private readonly ITenantService _tenantService;
    private readonly IMapper _mapper;

    public GetTenantByCodeQueryBuilder(IServiceProvider serviceProvider,
        ITenantService tenantService, IMapper mapper)
        : base(serviceProvider)
    {
        _tenantService = tenantService;
        _mapper = mapper;
        // No BuildAsync override needed
        // No validation step needed (handled by FluentValidation)
    }
}
```

## Limitations

1. **Single Transaction Scope**: All steps in a builder execute within a single transaction (if enabled). For complex scenarios requiring multiple transaction scopes, consider splitting into multiple builders.

2. **Sequential Execution**: Steps execute sequentially. For parallel execution, implement custom logic within a step.

3. **Error Handling**: 
   - Exceptions in steps cause transaction rollback (if enabled) and propagate to the handler. Handle expected errors by returning `GenericResponse<T>` instead of throwing.
   - Failed responses (BadRequest, NotFound, etc. where `Succeeded == false`) automatically trigger transaction rollback.
   - Only successful responses (`Succeeded == true`) commit the transaction.

## Future Enhancements

Potential improvements:

1. **Step Dependencies**: Allow steps to declare dependencies on other steps
2. **Parallel Steps**: Support parallel step execution where safe
3. **Step Retry Logic**: Automatic retry for transient failures
4. **Step Timeout**: Per-step timeout configuration
5. **Step Logging**: Enhanced logging for step execution

## Related Files

- **Base Classes**: 
  - `Alforqan.Core/Builders/Implementations/Base/Builder.cs`
  - `Alforqan.Core/Builders/Implementations/Base/AppBuilder.cs`
  - `Alforqan.Core/Builders/Implementations/Base/HostBuilder.cs`
- **Interface**: `Alforqan.Core/Builders/Abstractions/Base/IBuilder.cs`
- **Technical Guide**: `docs/technical-docs/builder-pattern-guide.md`

