# Tenant-Aware Logging Setup Guide

## Step 1: Register Logging Services

In `Alforqan.Api/ModuleAPIDependencies.cs`:

```csharp
// TenantLogContextEnricher requires IHttpContextAccessor
services.AddHttpContextAccessor();
services.AddSingleton<TenantLogContextEnricher>();
```

## Step 2: Configure Serilog with Centralized Host DB Sink

In `Alforqan.Api/Program.cs`:

```csharp
builder.Host.UseSerilog((context, services, configuration) =>
{
    configuration
        .ReadFrom.Configuration(context.Configuration);
    
    // Get TenantLogContextEnricher from service provider (requires dependencies)
    var tenantLogEnricher = services.GetRequiredService<TenantLogContextEnricher>();
    configuration.Enrich.With(tenantLogEnricher);
    
    // Get HostDb connection string for centralized logging
    var hostConnectionString = context.Configuration.GetConnectionString("HostDb");

    if (!string.IsNullOrEmpty(hostConnectionString))
    {
        var sinkOptions = new MSSqlServerSinkOptions
        {
            TableName = "SystemLogs",
            AutoCreateSqlTable = false, // Table created via EF migration
            BatchPostingLimit = 50,
            BatchPeriod = TimeSpan.FromSeconds(5)
        };

        var columnOptions = new ColumnOptions();
        // Remove MessageTemplate (not needed)
        columnOptions.Store.Remove(StandardColumn.MessageTemplate);

        // Add custom columns for tenant context
        columnOptions.AdditionalColumns = new Collection<SqlColumn>
        {
            new SqlColumn
            {
                ColumnName = "TenantId",
                PropertyName = "TenantId",
                DataType = SqlDbType.UniqueIdentifier,
                AllowNull = true
            },
            new SqlColumn
            {
                ColumnName = "TenantCode",
                PropertyName = "TenantCode",
                DataType = SqlDbType.NVarChar,
                DataLength = 64,
                AllowNull = true
            },
            new SqlColumn
            {
                ColumnName = "IsHostRequest",
                PropertyName = "IsHostRequest",
                DataType = SqlDbType.Bit,
                AllowNull = false
            },
            new SqlColumn
            {
                ColumnName = "RequestPath",
                PropertyName = "RequestPath",
                DataType = SqlDbType.NVarChar,
                DataLength = 512,
                AllowNull = true
            }
        };

        configuration.WriteTo.MSSqlServer(
            connectionString: hostConnectionString,
            sinkOptions: sinkOptions,
            columnOptions: columnOptions);
    }
});
```

## Step 3: Update appsettings.json

Ensure the `WriteTo` array does not include a hardcoded MSSqlServer sink (the sink is configured in code):

```json
"WriteTo": [
  {
    "Name": "Console"
  },
  {
    "Name": "File",
    "Args": {
      "path": "Logs\\log-.txt",
      "rollingInterval": "Day"
    }
  }
]
```

## Step 4: Update TenantResolutionMiddleware

Ensure tenant is stored in HttpContext.Items for the enricher to read:

```csharp
// After resolving tenant
context.Items["CurrentTenant"] = tenant;
await _next(context);
```

For host endpoints, no tenant is stored (enricher will detect host request from path).

## Step 5: Create SystemLogs Table Migration

The `SystemLogs` table must be created via EF Core migration in the Host DB:

```bash
dotnet ef migrations add AddSystemLogsTableToHostDb --project Alforqan.Infrastructure --startup-project Alforqan.Api --context HostDbContext
dotnet ef database update --project Alforqan.Infrastructure --startup-project Alforqan.Api --context HostDbContext
```

## Step 6: Verify Log Table Schema

The `SystemLogs` table in Host DB includes:

**Standard Serilog Columns**:
- `Id` (bigint, identity) - Primary key
- `Message` (nvarchar(max)) - Log message
- `Level` (varchar(128)) - Log level
- `TimeStamp` (datetime2) - When the log was created
- `Exception` (nvarchar(max)) - Exception details (if any)
- `Properties` (nvarchar(max)) - JSON of all log properties

**Custom Tenant Context Columns**:
- `TenantCode` (nvarchar(64), nullable) - Tenant code
- `TenantId` (uniqueidentifier, nullable) - Tenant ID
- `IsHostRequest` (bit, not null) - True for host/system admin operations
- `RequestPath` (nvarchar(512), nullable) - HTTP request path

**Indexes**:
- `IX_SystemLogs_TenantId_TimeStamp` - For tenant-specific queries
- `IX_SystemLogs_IsHostRequest_TimeStamp` - For host request queries

## Step 7: Test Logging

1. Make a host endpoint request → Check Host DB `SystemLogs` table
   - Filter: `WHERE IsHostRequest = 1`
   - Verify `TenantId` and `TenantCode` are NULL

2. Make a tenant endpoint request → Check Host DB `SystemLogs` table
   - Filter: `WHERE TenantCode = 'TENANT_CODE'` or `WHERE TenantId = 'TENANT_ID'`
   - Verify `IsHostRequest = 0`
   - Verify tenant context columns are populated

3. Verify logs include tenant context properties in the database columns

