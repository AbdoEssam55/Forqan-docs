# How to Add a New Tenant Plugin

## 1. Define the Service Interface

Create an abstraction for your feature in `Alforqan.Service/Abstractions/Plugins/`.

```csharp
namespace Alforqan.Service.Abstractions.Plugins;

public interface IMyFeatureService
{
    Task<string> ExecuteFeatureAsync(int entityId);
}
```

## 2. Implement Concrete Services

Create multiple implementations (behaviors) in `Alforqan.Service/Implementation/Plugins/`.

**Default Implementation:**

```csharp
public class DefaultMyFeatureService : IMyFeatureService
{
    public Task<string> ExecuteFeatureAsync(int entityId) => Task.FromResult("Default Behavior");
}
```

**Custom Implementation:**

```csharp
public class CustomMyFeatureService : IMyFeatureService
{
    public Task<string> ExecuteFeatureAsync(int entityId) => Task.FromResult("Custom Behavior");
}
```

## 3. Register Services

Register the implementations in `Alforqan.Service/ModuleServiceDependencies.cs` using **keyed services** and the `PluginName` enum/constant.

```csharp
// ModuleServiceDependencies.cs
services.AddKeyedScoped<IMyFeatureService, DefaultMyFeatureService>(ConstantsHelper.PluginNames.Default);
services.AddKeyedScoped<IMyFeatureService, CustomMyFeatureService>(ConstantsHelper.PluginNames.Formal);
```

## 4. Create a Builder using `PluginBuilder`

Create a builder for your feature's request logic that inherits from `PluginBuilder<TInput, TResponse, TDbContext>`. This gives you access to `A_ResolvePluginService`.

```csharp
// Core/Builders/Implementations/Features/MyFeatureBuilder.cs
public class MyFeatureBuilder : PluginBuilder<MyRequest, string, AppDbContext>, IMyFeatureBuilder
{
    private IMyFeatureService _service;

    public MyFeatureBuilder(
        IServiceProvider serviceProvider,
        AppDbContext dbContext,
        IPluginFactory pluginFactory) : base(serviceProvider, dbContext, pluginFactory)
    {
        UseTransaction = false;
    }

    // Step to resolve the correct service based on Tenant setup
    public Task<GenericResponse<string>> A_ResolveService()
    {
        return A_ResolvePluginService(out _service);
    }

    // Step to execute logic
    public async Task<GenericResponse<string>> B_ExecuteLogic()
    {
         var result = await _service.ExecuteFeatureAsync(_input.Id);
         return await _ReturnSuccessTask(result);
    }
}
```

## 5. Use in MediatR Handler

Inject the builder interface into your handler and execute the chain.

```csharp
public class MyFeatureHandler : IRequestHandler<MyRequest, GenericResponse<string>>
{
    private readonly IMyFeatureBuilder _builder;

    public MyFeatureHandler(IMyFeatureBuilder builder)
    {
        _builder = builder;
    }

    public async Task<GenericResponse<string>> Handle(MyRequest request, CancellationToken cancellationToken)
    {
        _builder.SetBuilderInput(request);

        return await _builder
            .AddStep(_builder.A_ResolveService)
            .AddStep(_builder.B_ExecuteLogic)
            .BuildAsync(cancellationToken);
    }
}
```
