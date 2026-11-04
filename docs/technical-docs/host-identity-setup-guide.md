# Host Identity Setup Guide

## Step 1: Register HostIdentityDbContext in Program.cs

```csharp
builder.Services.AddDbContext<HostIdentityDbContext>(options =>
{
    options.UseSqlServer(builder.Configuration.GetConnectionString(ConstantsHelper.ConnectionStringConstants.HostDb));
});
```

## Step 2: Register Host Identity services using AddIdentityCore

In `Alforqan.Infrastructure/ModuleInfrastructureDependencies.cs`:

```csharp
#region Host Identity registeration
services.AddIdentityCore<AppUser>(option =>
{
    option.SignIn.RequireConfirmedEmail = true;
})
.AddEntityFrameworkStores<HostIdentityDbContext>()
.AddDefaultTokenProviders();
#endregion
```

## Step 3: Configure dual JWT authentication schemes

In `Alforqan.Shared/ModuleSharedDependencies.cs`:

```csharp
services.AddAuthentication(options =>
{
    options.DefaultAuthenticateScheme = "TenantScheme";
    options.DefaultChallengeScheme = "TenantScheme";
})
.AddJwtBearer("TenantScheme", options =>
{
    options.TokenValidationParameters = new TokenValidationParameters
    {
        ValidateIssuer = JwtSettings.ValidateIssuer,
        ValidateAudience = JwtSettings.ValidateAudience,
        ValidateLifetime = JwtSettings.ValidateLifeTime,
        ValidateIssuerSigningKey = JwtSettings.ValidateIssuerSigningKey,
        RequireExpirationTime = true,
        ValidIssuer = JwtSettings.Issuer,
        ValidAudience = JwtSettings.Audience,
        IssuerSigningKey = new SymmetricSecurityKey(Encoding.ASCII.GetBytes(JwtSettings.SecretKey)),
    };
})
.AddJwtBearer("HostScheme", options =>
{
    options.TokenValidationParameters = new TokenValidationParameters
    {
        ValidateIssuer = JwtSettings.ValidateIssuer,
        ValidateAudience = JwtSettings.ValidateAudience,
        ValidateLifetime = JwtSettings.ValidateLifeTime,
        ValidateIssuerSigningKey = JwtSettings.ValidateIssuerSigningKey,
        RequireExpirationTime = true,
        ValidIssuer = JwtSettings.Issuer,
        ValidAudience = JwtSettings.Audience,
        IssuerSigningKey = new SymmetricSecurityKey(Encoding.ASCII.GetBytes(JwtSettings.SecretKey)),
    };
});
```

## Step 4: Create HostIdentityDbContextFactory for migrations

Create `Alforqan.Infrastructure/Data/HostIdentityDbContextFactory.cs`:

```csharp
using Microsoft.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore.Design;
using Microsoft.Extensions.Configuration;
using System.IO;

namespace Alforqan.Infrastructure.Data
{
    public class HostIdentityDbContextFactory : IDesignTimeDbContextFactory<HostIdentityDbContext>
    {
        public HostIdentityDbContext CreateDbContext(string[] args)
        {
            var configuration = new ConfigurationBuilder()
                .SetBasePath(Path.Combine(Directory.GetCurrentDirectory(), "../Alforqan.Api"))
                .AddJsonFile("appsettings.json", optional: false, reloadOnChange: true)
                .AddJsonFile("appsettings.Development.json", optional: true, reloadOnChange: true)
                .Build();

            var connectionString = configuration.GetConnectionString(ConstantsHelper.ConnectionStringConstants.HostDb);

            var optionsBuilder = new DbContextOptionsBuilder<HostIdentityDbContext>();
            optionsBuilder.UseSqlServer(connectionString);

            return new HostIdentityDbContext(optionsBuilder.Options);
        }
    }
}
```

## Step 5: Generate and apply migration

```bash
dotnet ef migrations add HostIdentityInitial --context HostIdentityDbContext --project Alforqan.Infrastructure --startup-project Alforqan.Api
dotnet ef database update --context HostIdentityDbContext --project Alforqan.Infrastructure --startup-project Alforqan.Api
```

## Step 6: Create host authentication endpoints

Create `Alforqan.Api/Controllers/Host/Auth/HostAuthController.cs`:

```csharp
[Route($"{CustomRouter.HostRouting.HostEndpointPrefix}/Auth")]
[ApiController]
public class HostAuthController : ControllerResponse
{
    [AllowAnonymous]
    [HttpPost(CustomRouter.HostRouting.HostAuthRouting.Login)]
    public async Task<IActionResult> Login([FromBody] HostLoginCommand command)
    {
        var response = await mediator.Send(command);
        return ReturnControllerResponse(response);
    }
}
```

## Step 7: Update TenantResolutionMiddleware to skip host routes

In `Alforqan.Api/Middlewares/TenantResolutionMiddleware.cs`:

```csharp
var path = context.Request.Path.Value ?? string.Empty;
var isHostEndpoint = path.StartsWith(CustomRouter.HostRouting.HostEndpointPrefix, StringComparison.OrdinalIgnoreCase);

if (isHostEndpoint)
{
    await _next(context);
    return;
}
```

## Step 8: Seed host users (SystemAdmin)

In `Alforqan.Api/Program.cs`:

```csharp
var userManagerFactory = scope.ServiceProvider.GetRequiredService<IUserManagerFactory>();
var hostIdentityDbContext = scope.ServiceProvider.GetRequiredService<HostIdentityDbContext>();
var hostUserManager = userManagerFactory.GetHostUserManager(hostIdentityDbContext);
var hostRoleManagerFactory = scope.ServiceProvider.GetRequiredService<IHostRoleManagerFactory>();
var hostRoleManager = hostRoleManagerFactory.CreateHostRoleManager(hostIdentityDbContext);

await AppUserSeeding.SeedHostUsersAsync(hostUserManager, hostRoleManager);
```





