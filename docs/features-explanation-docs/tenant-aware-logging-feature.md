# Tenant-Aware Logging Feature

## Problem Statement

In a multi-tenant system, logging errors and events with proper tenant identification is critical for:
- **Centralized Management**: All logs stored in one location for easier monitoring and analysis
- **Tenant Identification**: Each log entry must clearly identify which tenant it belongs to
- **Host Operations**: System admin operations must be distinguishable from tenant operations
- **Troubleshooting**: Developers need to quickly identify which tenant an error belongs to
- **Cross-Tenant Analysis**: Ability to query and analyze logs across all tenants from a single database
- **Compliance**: Support tenant-specific log access policies through query filters

The original implementation logged all errors to a single tenant database, causing:
- Host errors logged to tenant DB (wrong location)
- All tenant errors mixed in one database (no isolation)
- No tenant identification in logs (hard to troubleshoot)
- Difficult to perform cross-tenant analysis or centralized monitoring

## Architecture Overview

The Tenant-Aware Logging System uses a **two-layer approach**:

1. **LogContext Enricher** - Adds tenant context to log events (lightweight, in-memory)
2. **Direct Database Sink** - Writes all logs to centralized Host DB with tenant context columns

### Component Diagram

```
HTTP Request
    ↓
TenantResolutionMiddleware
    ├─ Resolves Tenant
    └─ Stores Tenant in HttpContext.Items
         ↓
TenantLogContextEnricher
    ├─ Reads Tenant from HttpContext.Items
    ├─ Detects Host vs Tenant Request
    └─ Enriches LogEvent with Context Properties:
       ├─ IsHostRequest: bool
       ├─ TenantCode: string (nullable)
       ├─ TenantId: Guid (nullable)
       └─ RequestPath: string
         ↓
MSSqlServerSink (Host DB)
    └─ Writes to SystemLogs table in Host DB (async, batched)
       ├─ All logs (host + tenant) stored in same table
       └─ Tenant context stored as columns for filtering
```

## Three Databases = Three DbContexts

The logging system operates in a multi-tenant architecture with three distinct DbContexts, each serving a specific purpose:

### HostDbContext

**Purpose**: Manages host-level data in the Host database.

**Connection**: Static connection string from configuration (`ConnectionStrings:HostDb`).

**Used For**:
- Storing tenants list (Tenants table)
- Storing which modules each tenant has (TenantModules table)
- Storing centralized logs (SystemLogs table)
- Host-level business entities

**Configuration**:
```csharp
builder.Services.AddDbContext<HostDbContext>(options =>
{
    options.UseSqlServer(builder.Configuration.GetConnectionString("HostDb"));
    // Suppress pending model changes warning - migrations handled manually
    options.ConfigureWarnings(warnings =>
        warnings.Ignore(RelationalEventId.PendingModelChangesWarning));
});
```

**Why suppress pending model changes warning?** To avoid EF Core spamming logs about "model changed but migrations not applied" - you handle migrations manually via `dotnet ef database update`.

### HostIdentityDbContext

**Purpose**: Manages ASP.NET Identity tables for platform admins in the Host database.

**Connection**: Same physical database as HostDbContext, but a different DbContext.

**Used For**:
- ASP.NET Identity tables for host-level users (SystemAdmin, etc.)
- Tables like `AspNetUsers`, `AspNetRoles`, `AspNetUserRoles` - but for host-level users

**Configuration**:
```csharp
builder.Services.AddDbContext<HostIdentityDbContext>(options =>
{
    options.UseSqlServer(builder.Configuration.GetConnectionString("HostDb"));
});
```

**Why separate from HostDbContext?**
- `HostDbContext` = tenants & modules (business entities)
- `HostIdentityDbContext` = identity/authentication for host-level users
- Same database, different "areas" - separation of concerns

### AppDbContext (Tenant DB)

**Purpose**: Manages tenant-specific data in tenant databases.

**Connection**: Dynamic connection string resolved per request from `ITenantContext.CurrentTenant.ConnectionString`.

**Used For**:
- Tenant-specific entities (Students, Teachers, Halaqat, etc.)
- Tenant users and roles (via Identity)
- All tenant business data

**Configuration**:
```csharp
builder.Services.AddDbContext<AppDbContext>((sp, options) =>
{
    try
    {
        var tenantContext = sp.GetRequiredService<ITenantContext>();
        
        if (!tenantContext.HasTenant)
        {
            throw new InvalidOperationException(
                "Tenant-scoped DbContext requested but no tenant is resolved. " +
                "Use HostDbContext for host-level operations.");
        }
        
        var connectionString = tenantContext.CurrentTenant!.ConnectionString;
        options.UseSqlServer(connectionString);
    }
    catch (InvalidOperationException)
    {
        // Design-time / tools / migrations fallback
        var defaultConnectionString = builder.Configuration.GetConnectionString("TenantDb");
        options.UseSqlServer(defaultConnectionString);
    }
});
```

**How it works**:
- For normal requests: `ITenantContext` is already filled by `TenantResolutionMiddleware`, so it reads `CurrentTenant.ConnectionString` and uses it for `AppDbContext`
- For design-time / tools / migrations: `ITenantContext` isn't available, so it falls back to `"TenantDb"` connection string from configuration

**Why throw if no tenant?** To avoid mistakes like accidentally using `AppDbContext` in host-level code. Host-level logic should use `HostDbContext` instead.

**Summary**:
- **Host-level**: `HostDbContext` + `HostIdentityDbContext` (both use Host DB)
- **Tenant-level**: `AppDbContext` (tenant-scoped connection string, per-tenant DB)

## Design Decisions

### Why LogContext Enricher Instead of Direct Database Lookup?

**LogContext Enricher** reads tenant information from HttpContext and adds it as log properties. It's:
- **Lightweight**: Just reads from HttpContext.Items (already populated by middleware)
- **Thread-Safe**: Serilog handles async/background operations automatically
- **No Performance Impact**: Happens in-memory, never blocks requests
- **Automatic**: All logs automatically include tenant context without code changes

**Alternative (Rejected)**: Looking up tenant in the sink would require:
- Database queries on every log write (slow)
- HttpContext access from background threads (not reliable)
- Complex thread synchronization

### Why Centralized Logging in Host DB?

**Centralized Approach**:
- **Simplified Infrastructure**: Single database connection for all logging
- **Easier Management**: One table to monitor, backup, and maintain
- **Cross-Tenant Analysis**: Query logs across all tenants from a single location
- **Unified Retention Policies**: Apply log retention and cleanup policies centrally
- **Tenant Identification**: Tenant context stored as columns (TenantId, TenantCode) for filtering
- **No Configuration Changes**: Supports unlimited tenants without sink configuration

**Alternative (Rejected)**: Per-tenant database logging would require:
- Multiple database connections and sinks
- Complex routing logic
- Difficult cross-tenant analysis
- More infrastructure to manage

## How It Works

### Request Flow

1. **Request Arrives**
   - HTTP request comes in with host header (e.g., `alnoor.alforqan.com` or `admin.alforqan.com`)

2. **Tenant Resolution** (`TenantResolutionMiddleware`)
   - Checks if request path starts with host endpoint prefix (e.g., `/API/V01/Tenants`)
   - If host endpoint: Does not resolve tenant, leaves `HttpContext.Items["CurrentTenant"]` null
   - If tenant endpoint: Resolves tenant from Host DB by PrimaryDomain and stores in `HttpContext.Items["CurrentTenant"]`
   - Middleware runs early in the pipeline (before authentication/authorization)

3. **Log Event Creation**
   - Any `Log.Error()`, `Log.Warning()`, `ILogger.LogError()`, etc. call creates a `LogEvent`
   - Serilog automatically calls all registered enrichers, including `TenantLogContextEnricher`
   - `TenantLogContextEnricher.Enrich()` method is called for each log event:
     - Reads `HttpContext` via `IHttpContextAccessor`
     - Extracts request path
     - Determines if it's a host request by checking if path starts with host endpoint prefix
     - Reads tenant from `HttpContext.Items["CurrentTenant"]` (set by middleware)
     - Adds properties: `IsHostRequest`, `TenantCode`, `TenantId`, `RequestPath`

4. **Database Write** (MSSqlServerSink)
   - Writes to `SystemLogs` table in **Host DB** (centralized)
   - Tenant context properties are written as columns:
     - `IsHostRequest` → bit column
     - `TenantCode` → nvarchar(64) column (nullable)
     - `TenantId` → uniqueidentifier column (nullable)
     - `RequestPath` → nvarchar(512) column (nullable)
   - Async, batched (50 logs per batch, 5-second intervals)
   - Non-blocking (happens in background thread)

5. **Querying Logs**
   - Filter by tenant: `WHERE TenantId = @tenantId`
   - Filter by tenant code: `WHERE TenantCode = @tenantCode`
   - Filter host requests: `WHERE IsHostRequest = 1`
   - Cross-tenant analysis: Query all logs without tenant filter

### Detailed Request Flow Examples

#### Example 1: Tenant User Hits `/API/V01/Students/Get`

1. Request comes in with host `alnoor.alforqan.com`
2. `TenantResolutionMiddleware` runs:
   - Resolves tenant from HostDbContext by PrimaryDomain
   - Checks tenant is Active
   - Stores it in: `context.Items["CurrentTenant"] = tenant`
3. Some service throws an exception → `ErrorHandlerMiddleware` logs:
   ```csharp
   Log.Error(ex, "Unhandled exception");
   ```
4. Serilog creates a `LogEvent`
5. Serilog calls `TenantLogContextEnricher.Enrich(logEvent, factory)`:
   - Gets path: `/API/V01/Students/Get`
   - `isHostRequest = false` (path does not start with HostEndpointPrefix)
   - Reads tenant from `HttpContext.Items["CurrentTenant"]`
   - Adds properties:
     - `"IsHostRequest" = false`
     - `"TenantCode" = "ALNOOR"`
     - `"TenantId" = "guid-here"`
     - `"RequestPath" = "/API/V01/Students/Get"`
6. Serilog sink writes to `SystemLogs` in HostDb with those columns filled

**Result in SystemLogs table:**
- `IsHostRequest = 0` (false)
- `TenantCode = "ALNOOR"`
- `TenantId = <guid>`
- `RequestPath = "/API/V01/Students/Get"`

#### Example 2: System Admin Hits `/API/V01/Tenants/Create`

1. Request host could be `admin.alforqan.com`
2. Path `/API/V01/Tenants/Create` starts with `HostEndpointPrefix`
3. `TenantResolutionMiddleware` sees it's a host endpoint → does not resolve tenant, leaves `CurrentTenant` null
4. Something logs:
   ```csharp
   Log.Information("Creating new tenant {Name}", dto.Name);
   ```
5. Enricher:
   - `isHostRequest = true`
   - `tenant = null`
   - Adds properties:
     - `"IsHostRequest" = true`
     - `"TenantCode" = null`
     - `"TenantId" = null`
     - `"RequestPath" = "/API/V01/Tenants/Create"`

**Result in SystemLogs table:**
- `IsHostRequest = 1` (true)
- `TenantCode = NULL`
- `TenantId = NULL`
- `RequestPath = "/API/V01/Tenants/Create"`

## TenantLogContextEnricher Deep Dive

The `TenantLogContextEnricher` is a small helper that runs for every log entry Serilog creates. Its job is to attach tenant context information (`IsHostRequest`, `TenantCode`, `TenantId`, `RequestPath`) to the log event before Serilog writes to the database.

### How It Gets Used

The enricher is wired in `Program.cs`:

```csharp
builder.Host.UseSerilog((context, services, configuration) =>
{
    configuration.ReadFrom.Configuration(context.Configuration);
    
    // Resolve the enricher from DI
    var tenantLogEnricher = services.GetRequiredService<TenantLogContextEnricher>();
    
    // Attach it to Serilog
    configuration.Enrich.With(tenantLogEnricher);
    
    // Configure MSSqlServer sink...
});
```

**Meaning**: Serilog is told "For every log event, call this enricher first." So whenever you do `Log.Error(...)` or `ILogger.LogError(...)`, Serilog creates a `LogEvent`, then runs all enrichers (including this one) before sending the event to the SQL sink.

### Dependencies

```csharp
private readonly IHttpContextAccessor _httpContextAccessor;
private const string TenantKey = "CurrentTenant";

public TenantLogContextEnricher(IHttpContextAccessor httpContextAccessor)
{
    _httpContextAccessor = httpContextAccessor;
}
```

- **IHttpContextAccessor**: Allows code outside controllers/middleware to access the current `HttpContext`. Under the hood, ASP.NET Core uses `AsyncLocal` to keep a per-request `HttpContext`, making logging code request-aware.
- **TenantKey = "CurrentTenant"**: This is the same key used in `TenantResolutionMiddleware` when it stores the tenant: `context.Items["CurrentTenant"] = tenant;`

**Connection**: Middleware resolves the Tenant and stores it in `HttpContext.Items["CurrentTenant"]`. This enricher later reads from this same place.

### The Enrich Method - Step by Step

This method is called once for each log event:

```csharp
public void Enrich(LogEvent logEvent, ILogEventPropertyFactory propertyFactory)
```

#### Step 1: Get Current HttpContext, Path, and Determine if Host Request

```csharp
var httpContext = _httpContextAccessor.HttpContext;
string path = string.Empty;
bool isHostRequest = false;

if (httpContext != null)
{
    path = httpContext.Request.Path.Value ?? string.Empty;
    isHostRequest = path.StartsWith(
        Alforqan.Data.AppMetaData.CustomRouter.HostRouting.HostEndpointPrefix,
        StringComparison.OrdinalIgnoreCase
    );
}
```

**What's happening:**
- **httpContext**: If there is an active HTTP request, this returns it. If the log comes from a background job, timed hosted service, etc., this will be `null`.
- **path**: If `HttpContext` exists → path is the requested URL path (e.g., `/API/V01/Students/Get`, `/API/V01/Tenants/Create`). If no `HttpContext` → it stays `""`.
- **isHostRequest**: We decide whether this log is for a host endpoint or a tenant endpoint. It uses `HostRouting.HostEndpointPrefix` (probably something like `/API/V01/Tenants`). If path starts with this prefix, we treat it as a host operation (system admin managing tenants, etc.).

#### Step 2: Attach IsHostRequest Property

```csharp
logEvent.AddPropertyIfAbsent(propertyFactory.CreateProperty("IsHostRequest", isHostRequest));
```

We create a Serilog property called `"IsHostRequest"` with bool value. `AddPropertyIfAbsent` → if some previous code already set `IsHostRequest`, we don't override it.

This property will later be written to the `IsHostRequest` bit column in `SystemLogs` via the sink config in `Program.cs`:

```csharp
new SqlColumn
{
    ColumnName = "IsHostRequest",
    PropertyName = "IsHostRequest",
    DataType = SqlDbType.Bit,
    AllowNull = false
}
```

#### Step 3: Try to Read Tenant from HttpContext.Items

```csharp
Tenant? tenant = null;
if (httpContext != null &&
    httpContext.Items.TryGetValue(TenantKey, out var tenantObj) &&
    tenantObj is Tenant resolvedTenant)
{
    tenant = resolvedTenant;
}
```

`HttpContext.Items` is a per-request key/value bag. The `TenantResolutionMiddleware` earlier did: `context.Items["CurrentTenant"] = tenant;`

This enricher now tries to retrieve that `Tenant` object:
- `TryGetValue(TenantKey, out var tenantObj)` - tries to get the value
- Check it is actually a `Tenant` (`tenantObj is Tenant resolvedTenant`)

If everything is OK: `tenant` now references the resolved `Tenant` entity for this request.

If not: `tenant` remains `null` → meaning this is either a host request (Tenants API), a background job, or tenant resolution failed somehow.

#### Step 4: If Tenant Exists → Attach Tenant Properties

```csharp
if (tenant != null)
{
    logEvent.AddPropertyIfAbsent(propertyFactory.CreateProperty("TenantCode", tenant.Code));
    logEvent.AddPropertyIfAbsent(propertyFactory.CreateProperty("TenantId", tenant.Id.ToString()));
    logEvent.AddPropertyIfAbsent(propertyFactory.CreateProperty("RequestPath", path));
}
```

For tenant requests, we enrich the log with:
- **"TenantCode"**: e.g., `"ALNOOR"`, `"DAR_ALFURQAN"` - maps to `nvarchar(64)` column
- **"TenantId"**: the Guid of the tenant, but as a string in the log event: `tenant.Id.ToString()`. The sink will parse this into `uniqueidentifier` because `DataType = UniqueIdentifier`
- **"RequestPath"**: the URL path we computed earlier

When the log is written to `SystemLogs`, one row will contain:
- `TenantId = <that tenant's GUID>`
- `TenantCode = <that tenant's code>`
- `IsHostRequest = false`
- `RequestPath = <endpoint path>`

This allows you to query:
```sql
SELECT TOP 100 *
FROM SystemLogs
WHERE TenantCode = 'ALNOOR'
ORDER BY TimeStamp DESC;
```

#### Step 5: If No Tenant → Still Log Something Meaningful

```csharp
else
{
    // No tenant resolved - this is a host request, background job, or tenant resolution failed
    logEvent.AddPropertyIfAbsent(propertyFactory.CreateProperty("TenantCode", (string?)null));
    logEvent.AddPropertyIfAbsent(propertyFactory.CreateProperty("TenantId", (string?)null));
    logEvent.AddPropertyIfAbsent(propertyFactory.CreateProperty("RequestPath", path));
}
```

If `tenant` is `null`, we still add the same properties, but:
- `TenantCode = null`
- `TenantId = null`
- `RequestPath = URL path` (if any)

**Why?** **Consistency**: Every log event has the same set of properties (`IsHostRequest`, `TenantCode`, `TenantId`, `RequestPath`). That means every row in `SystemLogs` has columns filled or null.

**Meaning**:
- If `IsHostRequest = true` and `TenantCode` is null → host-level log
- If `IsHostRequest = false` and `TenantCode` is null → probably a background job or some early error before tenant resolution

So host/system logs look like:
- `TenantId = null`
- `TenantCode = null`
- `IsHostRequest = true`

You can filter host logs easily:
```sql
SELECT *
FROM SystemLogs
WHERE IsHostRequest = 1;
```

### How It Connects to the Database Table

In `Program.cs`, you configured the sink like this:

```csharp
columnOptions.AdditionalColumns = new Collection<SqlColumn>
{
    new SqlColumn { ColumnName = "TenantId", PropertyName = "TenantId", ... },
    new SqlColumn { ColumnName = "TenantCode", PropertyName = "TenantCode", ... },
    new SqlColumn { ColumnName = "IsHostRequest", PropertyName = "IsHostRequest", ... },
    new SqlColumn { ColumnName = "RequestPath", PropertyName = "RequestPath", ... }
};
```

**Key point**: `PropertyName` in the sink config must match the name used in the enricher:
- `"TenantId"` → `"TenantId"`
- `"TenantCode"` → `"TenantCode"`
- `"IsHostRequest"` → `"IsHostRequest"`
- `"RequestPath"` → `"RequestPath"`

The enricher creates the properties; the sink maps them to columns.

### Performance: Why It's "Lightweight"

The enricher does:
- A single `HttpContextAccessor.HttpContext` lookup (AsyncLocal)
- One string compare: `StartsWith(...)`
- One lookup in `HttpContext.Items` dictionary
- Creating 3–4 small objects (log properties)

There is:
- **No database call**
- **No network call**
- Just reading from stuff that is already in memory

Compared to your normal request handling, this is tiny work.

## Serilog Configuration Deep Dive

The Serilog configuration in `Program.cs` is the heart of the logging system. Let's break it down step by step:

### Step 1: Load Configuration

```csharp
configuration.ReadFrom.Configuration(context.Configuration);
```

Reads logging configuration from `appsettings.json` (Serilog section if present). This allows you to configure log levels, other sinks, etc. from configuration files.

### Step 2: Enrich Logs with Tenant Info

```csharp
var tenantLogEnricher = services.GetRequiredService<TenantLogContextEnricher>();
configuration.Enrich.With(tenantLogEnricher);
```

`TenantLogContextEnricher` is a class that:
- Reads `ITenantContext` and/or `HttpContext`
- Adds properties like `TenantId`, `TenantCode`, `IsHostRequest`, `RequestPath`

Every log entry that goes through Serilog gets these fields attached automatically.

### Step 3: Configure SQL Sink to Host DB

```csharp
var hostConnectionString = context.Configuration.GetConnectionString("HostDb");

var sinkOptions = new MSSqlServerSinkOptions
{
    TableName = "SystemLogs",
    AutoCreateSqlTable = false,  // Table created via EF migration
    BatchPostingLimit = 50,
    BatchPeriod = TimeSpan.FromSeconds(5)
};
```

**Key Configuration Options**:

- **TableName**: `"SystemLogs"` - The table in Host DB where all logs are written
- **AutoCreateSqlTable = false**: **Critical setting** - Serilog's SQL sink can create the table for you, but we turned that off on purpose because:
  - You treat `SystemLogs` like any other table in your system
  - You have an EF Core entity `SystemLog` with indexes
  - You manage it with migrations (versioned with the rest of your DB)
  - If Serilog auto-created the table, it would create a basic table with no extra indexes, and you'd lose control over column types, lengths, indexes, and constraints
  - EF migrations and Serilog's auto-created schema could conflict
  - **Solution**: Let EF migration create `SystemLogs` exactly how you want (with indexes, correct types, etc.), then Serilog just writes to it

- **BatchPostingLimit = 50**: Serilog collects log entries in memory (in a small buffer) and does not hit the database for every `Log.Error()` call. Instead, it waits until it has 50 log events, then sends that batch to SQL in one go.

- **BatchPeriod = TimeSpan.FromSeconds(5)**: If 5 seconds pass before reaching 50 logs, it sends whatever it has collected so far.

**Why batching is good**:
- **Requests stay fast**: When your code does `Log.Error("...")`, it just adds to an in-memory buffer. It doesn't wait for SQL.
- **Actual DB write happens in a background thread**: Serilog has an internal worker that wakes up every few seconds and flushes logs.
- **Less DB pressure**: Instead of 1 insert per log line, it does bulk inserts in batches.

**In simple words**: When your API throws an error and calls `Log.Error`, the request does not slow down waiting for the log to hit SQL. The log is written later, in the background, together with other logs. That's what "async, non-blocking" means here.

### Step 4: Map Serilog Properties to SQL Columns

```csharp
var columnOptions = new ColumnOptions();
columnOptions.Store.Remove(StandardColumn.MessageTemplate);

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
```

**Key Points**:

- **StandardColumn.MessageTemplate removed**: To save space. You mostly care about final `Message`, `Level`, `Exception`, etc.
- **AdditionalColumns**: Define extra columns in your SQL table that should be filled from log properties
- **PropertyName must match**: The `PropertyName` in the sink config must match the name used in the enricher:
  - `"TenantId"` → `"TenantId"`
  - `"TenantCode"` → `"TenantCode"`
  - `"IsHostRequest"` → `"IsHostRequest"`
  - `"RequestPath"` → `"RequestPath"`

The enricher creates the properties; the sink maps them to columns.

### Complete Sink Configuration

```csharp
configuration.WriteTo.MSSqlServer(
    connectionString: hostConnectionString,
    sinkOptions: sinkOptions,
    columnOptions: columnOptions);
```

This configures Serilog to write all logs to the `SystemLogs` table in Host DB with the custom tenant context columns.

## Database Schema

The `SystemLogs` table is located in the **Host DB** and is created via EF Core migration (not auto-created by the sink). It includes:

**Standard Serilog Columns**:
- `Id` (bigint, identity) - Primary key
- `Message` (nvarchar(max)) - Log message
- `Level` (varchar(128)) - Log level (Error, Warning, Information, etc.)
- `TimeStamp` (datetime2) - When the log was created
- `Exception` (nvarchar(max)) - Exception details (if any)
- `Properties` (nvarchar(max)) - JSON of all log properties

**Custom Tenant Context Columns**:
- `TenantCode` (nvarchar(64), nullable) - Tenant code (e.g., "ALNOOR")
- `TenantId` (uniqueidentifier, nullable) - Tenant ID
- `IsHostRequest` (bit, not null) - True for host/system admin operations
- `RequestPath` (nvarchar(512), nullable) - HTTP request path

**Indexes** (for performance):
- `IX_SystemLogs_TenantId_TimeStamp` - For tenant-specific log queries
- `IX_SystemLogs_IsHostRequest_TimeStamp` - For host request log queries

## SystemLog Entity - Property by Property

The `SystemLog` entity is a normal EF Core entity that maps to a table called `SystemLogs` in HostDb. Its main purpose is to let EF create and manage the `SystemLogs` table via migrations. You will not usually query it from code, and Serilog does not use EF to write logs. Serilog writes directly to `SystemLogs` using the SQL sink, but EF is used to define the schema and create/update the table. Think of it like a "table blueprint" for EF migrations.

### Property Details

#### Id
```csharp
[Key]
[DatabaseGenerated(DatabaseGeneratedOption.Identity)]
public long Id { get; set; }
```
- **Primary key**
- **Type**: `long` (bigint in SQL)
- **DatabaseGenerated.Identity** → SQL auto-increments it (`IDENTITY(1,1)`)
- Every log row gets a unique `Id`

#### TimeStamp
```csharp
[Required]
public DateTime TimeStamp { get; set; }
```
- When the log was created
- Marked as `[Required]` → cannot be null in DB
- Serilog automatically fills it when writing logs
- Use this in queries like:
  ```sql
  SELECT TOP 100 *
  FROM SystemLogs
  WHERE TenantCode = 'ALNOOR'
  ORDER BY TimeStamp DESC;
  ```

#### Level
```csharp
[Required]
[StringLength(128)]
public string Level { get; set; } = string.Empty;
```
- The log level: `"Error"`, `"Information"`, `"Warning"`, `"Fatal"`, etc.
- `[Required]` → cannot be null
- `StringLength(128)` → SQL column will be `nvarchar(128)`
- Useful when filtering only errors:
  ```sql
  SELECT *
  FROM SystemLogs
  WHERE Level = 'Error';
  ```

#### Message
```csharp
[Column(TypeName = "nvarchar(max)")]
public string? Message { get; set; }
```
- The final rendered message, e.g.: `"Error while creating student 123 in tenant ALNOOR"`
- Can be null (just in case)
- `nvarchar(max)` → can be long

#### Exception
```csharp
[Column(TypeName = "nvarchar(max)")]
public string? Exception { get; set; }
```
- Full exception details (stack trace) if any
- `null` for non-error logs
- Also `nvarchar(max)` because stack traces can be long

#### Properties
```csharp
[Column(TypeName = "nvarchar(max)")]
public string? Properties { get; set; }
```
- JSON blob with all remaining Serilog properties that are not mapped to columns
- Example content (simplified):
  ```json
  {
    "SourceContext": "Alforqan.Api.Middlewares.ErrorHandlerMiddleware",
    "UserId": "abc123",
    "CorrelationId": "d7d9c..."
  }
  ```
- This is useful when you want to see raw data, but for fast queries, you rely on indexed columns like `TenantId`, `TenantCode`, `Level`, `TimeStamp`

#### TenantId
```csharp
/// <summary>
/// Tenant ID for tenant-specific logs. Null for host/system admin logs.
/// </summary>
public Guid? TenantId { get; set; }
```
- `Guid?` → nullable
- If the log came from a tenant request, this is that tenant's `Id`
- If it's a host request (system admin, tenants API) or background job: This is `null`
- Example query:
  ```sql
  SELECT TOP 50 *
  FROM SystemLogs
  WHERE TenantId = 'GUID-OF-TENANT-HERE'
  ORDER BY TimeStamp DESC;
  ```

#### TenantCode
```csharp
/// <summary>
/// Tenant code for tenant-specific logs. Null for host/system admin logs.
/// </summary>
[StringLength(64)]
public string? TenantCode { get; set; }
```
- Human-friendly identifier for tenant like `"ALNOOR"`, `"DAR_ALFURQAN"`
- Nullable
- `nvarchar(64)`
- Example queries:
  ```sql
  SELECT COUNT(*)
  FROM SystemLogs
  WHERE TenantCode = 'ALNOOR'
    AND Level = 'Error';
  ```

#### IsHostRequest
```csharp
/// <summary>
/// Indicates if this log is from a host/system admin request.
/// True for host requests, false for tenant requests.
/// </summary>
[Required]
public bool IsHostRequest { get; set; }
```
- `true` → log came from host endpoints (e.g. `/API/V01/Tenants/...`), or a part of the system considered "host"
- `false` → log came from tenant endpoints
- Never null (`[Required]` → `bit NOT NULL` in SQL)
- This works together with the enricher:
  - **Tenant requests**: `IsHostRequest = false`, `TenantCode`, `TenantId` filled
  - **Host requests**: `IsHostRequest = true`, `TenantCode`, `TenantId = null`
- You can get all host-level logs easily:
  ```sql
  SELECT *
  FROM SystemLogs
  WHERE IsHostRequest = 1;
  ```

#### RequestPath
```csharp
/// <summary>
/// HTTP request path that generated this log entry.
/// </summary>
[StringLength(512)]
public string? RequestPath { get; set; }
```
- The URL path of the request: `/API/V01/Students/Get`, `/API/V01/Tenants/Create`, `/health`, etc.
- Helps you know which endpoint was involved when an error happened
- Nullable in case there's no `HttpContext` (background job)
- Example query:
  ```sql
  SELECT TOP 50 *
  FROM SystemLogs
  WHERE RequestPath LIKE '/API/V01/Students%'
  ORDER BY TimeStamp DESC;
  ```

### How EF and Serilog Work Together

**Important**: EF does NOT write logs. Serilog does.

**Process**:
1. You define `SystemLog` as an entity
2. Add `DbSet<SystemLog>` in `HostDbContext`
3. Run `dotnet ef migrations add AddSystemLogsTableToHostDb`
4. Run `dotnet ef database update --context HostDbContext`
5. **Result**: EF generates the `SystemLogs` table with the columns that match this class

**Then**:
- Serilog is configured in `Program.cs` with the `MSSqlServer` sink
- It writes directly into `HostDb.SystemLogs` using ADO.NET
- It uses the column definitions you gave in `ColumnOptions` so it knows:
  - Which Serilog property goes into which column
  - Which SQL type to use

**The names match**:
- Serilog property `"TenantId"` → column `TenantId`
- `"TenantCode"` → column `TenantCode`
- `"IsHostRequest"` → `IsHostRequest`
- `"RequestPath"` → `RequestPath`

**The `SystemLog` class is mainly there so**:
- EF can create the table and indexes
- You have strong typing if you ever want to query logs via EF (for admin UI, etc.)

### Why This Design is Good

- **Centralized**: All logs (host + all tenants) in one table in HostDb
- **Tenant-aware**: You can filter logs by tenant or host
- **Simple querying**: Common columns (`Level`, `TimeStamp`, `TenantCode`, `IsHostRequest`) are plain columns, not JSON
- **Flexible**: Extra log metadata still stored in `Properties` as JSON
- **Clean separation**: EF controls the schema, Serilog controls the writing

## Performance Characteristics

### Request Impact

- **LogContext Push**: < 0.001ms (in-memory operation)
- **Log Write**: 0ms (async, non-blocking, happens in background)
- **Total Request Impact**: **Negligible** (< 0.001ms)

### Background Thread Operations

- **Database Write**: Async, batched (50 logs per batch)
- **Connection Pooling**: Reuses existing database connections to Host DB

### Memory Usage

- **Single Sink Instance**: ~1KB (one MSSqlServerSink for all logs)
- **Total**: Minimal memory footprint

## Security Considerations

### Tenant Identification

- **Column-Based Identification**: Tenant context stored as columns (TenantId, TenantCode) in centralized table
- **Query Filtering**: Access control enforced through queries filtering by TenantId
- **Host Request Identification**: Host/system admin operations identified by `IsHostRequest = true`

### Log Data Sensitivity

- **No Secrets in Logs**: Passwords, tokens, connection strings are never logged
- **Tenant Context Only**: Logs include tenant identification, not sensitive data
- **Compliance Ready**: Supports tenant-specific log access policies through query filters
- **Centralized Storage**: All logs in Host DB, but tenant data can be isolated via queries

### Access Control

- **Database-Level Security**: Host DB access restricted to backend services
- **Query-Level Filtering**: Application code must filter by TenantId when querying logs
- **Audit Trail**: All logs include tenant identification for compliance and auditing

## Request Pipeline Order

The order of middleware in the request pipeline is critical for tenant-aware logging to work correctly:

```csharp
app.UseMiddleware<ErrorHandlerMiddleware>();  // 1. Wraps everything, catches exceptions
// ... other middleware ...
app.UseRequestLocalization(locOptions.Value);
app.UseHttpsRedirection();
app.UseRouting();

app.UseMiddleware<TenantResolutionMiddleware>();  // 2. Resolves tenant BEFORE auth

app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();
```

**Why this order matters:**

1. **ErrorHandlerMiddleware** wraps everything:
   - Catches exceptions and returns standard error responses
   - Also logs errors (which now go through Serilog to SystemLogs with tenant info)

2. **UseRouting** must come before TenantResolutionMiddleware:
   - Routing needs to be resolved so the middleware can check the request path

3. **TenantResolutionMiddleware** must come before Authentication/Authorization:
   - Reads Host header
   - Resolves which tenant it is
   - Puts `CurrentTenant` into `HttpContext.Items["CurrentTenant"]`
   - Enriches `LogContext` (via `LogContext.PushProperty`) with `TenantId`, `TenantCode`, `IsHostRequest`, `RequestPath`
   - This is what makes logs tenant-aware

4. **Authentication + Authorization**:
   - Now that tenant is known, if your handlers use `AppDbContext`, it already has the right connection string

5. **Controllers**:
   - Can now use tenant-aware `AppDbContext` and all logs automatically include tenant context

**Summary**: Tenant resolution happens before auth and controllers. Error handler wraps everything and logs errors with tenant info.

## Integration Points

### With Tenant Resolution

- `TenantResolutionMiddleware` resolves tenant and stores in `HttpContext.Items["CurrentTenant"]`
- `TenantLogContextEnricher` reads from HttpContext to enrich logs automatically
- The middleware runs early in the pipeline (after routing, before authentication) to ensure tenant context is available for all subsequent operations

### With Error Handling

- `ErrorHandlerMiddleware` logs errors with tenant context automatically
- No code changes needed - tenant context is automatically added by enricher
- Errors are logged to `SystemLogs` with full tenant identification

### With Application Logging

- All `Log.Error()`, `Log.Warning()`, etc. calls automatically include tenant context
- Works with dependency-injected `ILogger<T>` as well
- All logs written to centralized Host DB `SystemLogs` table
- Background jobs or code without HttpContext will have `TenantId = null`, `TenantCode = null`, but `IsHostRequest` will still be set appropriately

## Benefits

1. **Automatic Tenant Context**: No code changes needed - logs automatically include tenant identification
2. **Zero Performance Impact**: Enricher reads from HttpContext (already populated), database writes are async
3. **Centralized Management**: All logs in one database for easier monitoring, backup, and maintenance
4. **Easy Troubleshooting**: Logs include tenant identification (TenantCode, TenantId) for filtering
5. **Cross-Tenant Analysis**: Query logs across all tenants from a single location
6. **Scalable**: Supports unlimited tenants without configuration changes
7. **Simplified Infrastructure**: Single database connection for all logging
8. **Compliance Ready**: Supports tenant-specific log access policies through query filters

## LogCleanupBackgroundService Deep Dive

The `LogCleanupBackgroundService` is a background service that runs in the API process. Its job is to periodically delete old rows from `HostDb.SystemLogs` so the table doesn't grow forever. This implements automatic log retention based on the configured `LogRetentionDays` setting.

### What It Is

```csharp
public class LogCleanupBackgroundService : BackgroundService
```

It's a background service that runs in the API process. When your app starts, ASP.NET Core calls `ExecuteAsync` and the service runs continuously until the app shuts down.

### Dependencies

```csharp
private readonly IServiceScopeFactory _serviceScopeFactory;
private readonly LoggingSettings _loggingSettings;
private readonly ILogger<LogCleanupBackgroundService> _logger;
private const int BatchSize = 1000;
private const int HoursBetweenRuns = 24;
```

#### IServiceScopeFactory

**Why can't we inject a scoped service directly into a singleton?**

- **Singleton** = one instance for the whole app lifetime
- **Scoped** = one instance per request / per scope
- If you inject a scoped service into a singleton, you're basically saying: "Give this singleton one scoped instance and keep it forever"

That breaks the whole idea of "per request":
- The scoped service (like `HostDbContext`) is supposed to be created at the start of a request/scope and disposed at the end
- If the singleton holds onto it:
  - It might use it after it's disposed
  - It will reuse the same `DbContext` across multiple requests, which is unsafe (DbContext is not thread-safe)
  - You might get weird errors / race conditions
  - Tracking state gets messed up

**So the DI container blocks this by design**: You cannot inject scoped → singleton (it throws or behaves incorrectly).

**How does IServiceScopeFactory fix this?**

The background service is a singleton, but some of the work it does is like "fake request work" (it needs a `DbContext` as if it was a request). We solve this by creating our own mini-scope inside the singleton:

```csharp
using var scope = _serviceScopeFactory.CreateScope();
var hostDbContext = scope.ServiceProvider.GetRequiredService<HostDbContext>();
```

**What happens here:**
- `CreateScope()` creates a new DI scope, just like ASP.NET does per HTTP request
- Inside that scope, `HostDbContext` is scoped, so a fresh `DbContext` is created for that scope
- When the `using scope` ends, the scope is disposed, and the `HostDbContext` is also disposed

**So:**
- The background service stays singleton
- The `DbContext` stays scoped (per cleanup run, not per app)
- Lifetimes are respected, no cross-thread issues, no reuse of disposed `DbContexts`

You can think of it like: "The background service pretends to be a 'fake request' by creating a scoped container for the duration of the cleanup job."

#### LoggingSettings

Holds configuration from `appsettings.json`, e.g.:

```json
"LoggingSettings": {
  "LogRetentionDays": 90
}
```

`LogRetentionDays` = how many days of logs you want to keep.

#### ILogger<LogCleanupBackgroundService>

**Why ILogger<LogCleanupBackgroundService> specifically?**

That generic type is just the category of the logger. ASP.NET Core logging is category-based. The category is usually the class name, and is used for:
- Grouping logs
- Filtering logs in settings (e.g., "only show logs from `Alforqan.Api.BackgroundServices`")
- Seeing in your log output which class wrote this message

When you use `ILogger<LogCleanupBackgroundService>`, the logging system treats it as:
- Category name = `"Alforqan.Api.BackgroundServices.LogCleanupBackgroundService"`

So, in your logs you'll see:
```
[Information] Alforqan.Api.BackgroundServices.LogCleanupBackgroundService ...
[Error] Alforqan.Api.BackgroundServices.LogCleanupBackgroundService ...
```

**Why is that nice?** In Serilog / Kibana / Seq / whatever, you can filter by this category. It's standard ASP.NET Core convention: each class that logs gets `ILogger<ThatClass>`.

**Could you use ILogger or ILogger<object>?** Yes, but then all logs look the same category, and it's harder to know where they came from. Using the class type gives you free grouping + filtering + clarity.

#### Constants

- **BatchSize = 1000**: Delete logs in chunks of 1000 rows
- **HoursBetweenRuns = 24**: Run cleanup once every 24 hours

### How It Runs (ExecuteAsync)

```csharp
protected override async Task ExecuteAsync(CancellationToken stoppingToken)
{
    _logger.LogInformation("Log cleanup background service started. Retention period: {RetentionDays} days", 
        _loggingSettings.LogRetentionDays);

    while (!stoppingToken.IsCancellationRequested)
    {
        try
        {
            await CleanupOldLogsAsync(stoppingToken);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error occurred during log cleanup. Service will retry in {Hours} hours.", 
                HoursBetweenRuns);
        }

        // Wait 24 hours before next cleanup run
        await Task.Delay(TimeSpan.FromHours(HoursBetweenRuns), stoppingToken);
    }
}
```

**Step by step:**
1. When your app starts, ASP.NET Core calls `ExecuteAsync`
2. It logs: `"Log cleanup background service started. Retention period: X days"`
3. Enters a `while` loop that runs until app shuts down
4. Inside the loop:
   - It calls `CleanupOldLogsAsync`
   - If that throws an exception: It logs the error but does not crash the service
5. After each run, it waits 24 hours (`Task.Delay`) and then repeats

**So**: Once per day, it cleans logs.

**stoppingToken** is used so: If the app shuts down, the loop exits gracefully. No infinite loop on shutdown.

### The Real Work: CleanupOldLogsAsync

```csharp
private async Task CleanupOldLogsAsync(CancellationToken cancellationToken)
{
    using var scope = _serviceScopeFactory.CreateScope();
    var hostDbContext = scope.ServiceProvider.GetRequiredService<HostDbContext>();
    // ... rest of the method ...
} // 👈 HERE the `scope` gets disposed automatically
```

**Where does the scope end?**

With the new C# `using var` declaration syntax:
- There's no braces
- The variable is automatically disposed at the end of the current block (the method or the surrounding `{}`)

In this method:
- The lifetime of `scope` = the whole `CleanupOldLogsAsync` method body
- When the method finishes (returns), `scope.Dispose()` is called
- Disposing the scope → disposes `HostDbContext` and any other scoped services created from that scope

**That's exactly what we want**: One short-lived scope just for this cleanup run, with a clean `HostDbContext`, then throw it away. Next day, the background service creates another scope and does the same.

#### Step 1: Decide What to Delete

```csharp
var cutoffDate = DateTime.UtcNow.AddDays(-_loggingSettings.LogRetentionDays);
_logger.LogInformation("Starting log cleanup. Deleting logs older than {CutoffDate}", cutoffDate);
```

If `LogRetentionDays = 90`:
- `cutoffDate = now - 90 days`
- Any log with `TimeStamp < cutoffDate` is considered too old and should be deleted

**So**: "keep last 90 days, delete anything older"

#### Step 2: Batch Deletion Loop

```csharp
int totalDeleted = 0;

while (!cancellationToken.IsCancellationRequested)
{
    var oldLogs = await hostDbContext.SystemLogs
        .Where(l => l.TimeStamp < cutoffDate)
        .OrderBy(l => l.TimeStamp)
        .Take(BatchSize)
        .ToListAsync(cancellationToken);

    if (!oldLogs.Any())
    {
        break; // No more logs to delete
    }

    hostDbContext.SystemLogs.RemoveRange(oldLogs);
    await hostDbContext.SaveChangesAsync(cancellationToken);

    totalDeleted += oldLogs.Count;
    _logger.LogDebug("Deleted {BatchCount} log entries (total so far: {TotalDeleted})", 
        oldLogs.Count, totalDeleted);

    // Small delay between batches to reduce database load
    await Task.Delay(100, cancellationToken);
}
```

**Why a loop?**

You don't want to run:
```sql
DELETE FROM SystemLogs WHERE TimeStamp < cutoffDate
```
in one shot if there are millions of rows:
- It can lock the table for a long time
- It can cause timeouts
- It's harder on the database

**So instead**, it:
1. Selects up to `BatchSize` rows (e.g., 1000)
2. Deletes them
3. Repeats until there's nothing left to delete

**Query:**
- Finds logs older than `cutoffDate`
- Orders them by `TimeStamp` ascending (oldest first)
- Takes 1000 rows max

**If no rows**: `if (!oldLogs.Any()) { break; }` → Then we're done. No more logs to clean.

**Deleting the batch:**
- `hostDbContext.SystemLogs.RemoveRange(oldLogs)` → Marks these 1000 entities for deletion
- `SaveChangesAsync()` → Sends a DELETE command to DB (for those specific rows)

**Tracking progress:**
- Keeps track of how many rows were deleted in total
- Writes debug logs so you can see what happened

**Small delay between batches:**
- 100ms pause between batches
- **Why?** To avoid hammering the DB in one continuous tight loop. Gives the DB "breathing room" if there are a ton of rows.

#### Step 3: Final Message

```csharp
if (totalDeleted > 0)
{
    _logger.LogInformation("Log cleanup completed. Deleted {TotalDeleted} log entries older than {CutoffDate}", 
        totalDeleted, cutoffDate);
}
else
{
    _logger.LogInformation("Log cleanup completed. No log entries found older than {CutoffDate}", cutoffDate);
}
```

If some logs were deleted: Logs how many and until which date. If none: Logs that there was nothing to delete. This is useful to confirm that the job is running.

### How This Fits in the Whole Logging Feature

Let's connect the dots:

1. **Serilog + MSSqlServer sink**: Writes all logs to `HostDb.SystemLogs`
2. **TenantLogContextEnricher**: Enriches each log event with `TenantId`, `TenantCode`, `IsHostRequest`, `RequestPath` → So every row in `SystemLogs` knows which tenant / host / endpoint produced it
3. **SystemLog entity**: EF model to create the `SystemLogs` table + indexes
4. **LogCleanupBackgroundService** (this file): Keeps `SystemLogs` small and healthy:
   - Deletes old logs based on `LogRetentionDays`
   - Runs once per day
   - Deletes in small batches (1000 rows) to avoid stress on DB

**So the overall idea**: "Log everything in one place (HostDb), label each row with tenant/host info, and clean up old data automatically."

## Practical SQL Query Examples

### Show Last 50 Errors for a Specific Tenant

```sql
SELECT TOP 50
    TimeStamp,
    Level,
    Message,
    Exception,
    RequestPath
FROM SystemLogs
WHERE TenantCode = 'ALNOOR'
  AND Level = 'Error'
ORDER BY TimeStamp DESC;
```

### Show All Host Errors for the Last 7 Days

```sql
SELECT
    TimeStamp,
    Level,
    Message,
    Exception,
    RequestPath
FROM SystemLogs
WHERE IsHostRequest = 1
  AND Level = 'Error'
  AND TimeStamp >= DATEADD(DAY, -7, GETUTCDATE())
ORDER BY TimeStamp DESC;
```

### Count Errors per Tenant (Last 30 Days)

```sql
SELECT
    TenantCode,
    COUNT(*) AS ErrorCount
FROM SystemLogs
WHERE Level = 'Error'
  AND TimeStamp >= DATEADD(DAY, -30, GETUTCDATE())
  AND TenantCode IS NOT NULL
GROUP BY TenantCode
ORDER BY ErrorCount DESC;
```

### Find All Logs for a Specific Endpoint

```sql
SELECT TOP 100
    TimeStamp,
    Level,
    Message,
    TenantCode,
    IsHostRequest
FROM SystemLogs
WHERE RequestPath LIKE '/API/V01/Students%'
ORDER BY TimeStamp DESC;
```

### Get Error Rate per Tenant (Last 24 Hours)

```sql
SELECT
    TenantCode,
    COUNT(CASE WHEN Level = 'Error' THEN 1 END) AS ErrorCount,
    COUNT(*) AS TotalLogs,
    CAST(COUNT(CASE WHEN Level = 'Error' THEN 1 END) * 100.0 / COUNT(*) AS DECIMAL(5,2)) AS ErrorPercentage
FROM SystemLogs
WHERE TimeStamp >= DATEADD(HOUR, -24, GETUTCDATE())
  AND TenantCode IS NOT NULL
GROUP BY TenantCode
ORDER BY ErrorCount DESC;
```

## Limitations & Future Enhancements

### Current Limitations

- **Query Performance**: Large log tables may require pagination for efficient queries
- **No Log Archiving**: Old logs are deleted, not archived to cold storage

### Future Enhancements

- **Real-time Monitoring**: Dashboard showing errors per tenant with filtering
- **Alerting**: Notifications when error rates exceed thresholds per tenant
- **Log Archiving**: Archive old logs to cold storage while maintaining query capability
- **Advanced Filtering**: UI for complex log queries with date ranges, tenant filters, level filters

## Related Files

### Core Logging Components

- `Alforqan.Api/Logging/TenantLogContextEnricher.cs` - Adds tenant context to log events (enriches LogEvent with TenantId, TenantCode, IsHostRequest, RequestPath)
- `Alforqan.Api/Program.cs` - Serilog configuration with MSSqlServerSink to Host DB (lines 124-200)
- `Alforqan.Data/Entities/Logging/SystemLog.cs` - Entity definition for SystemLogs table (EF Core model)
- `Alforqan.Infrastructure/Data/HostDbContext.cs` - DbContext with SystemLogs DbSet
- `Alforqan.Infrastructure/Migrations/Host/` - EF Core migrations creating SystemLogs table

### Background Services

- `Alforqan.Api/BackgroundServices/LogCleanupBackgroundService.cs` - Background service that periodically deletes old log entries based on retention policy

### Configuration & Settings

- `Alforqan.Shared/Settings/LoggingSettings.cs` - Configuration class for log retention settings (LogRetentionDays)
- `Alforqan.Api/appsettings.json` - Contains `LoggingSettings` section with retention configuration

### Integration Points

- `Alforqan.Api/Middlewares/TenantResolutionMiddleware.cs` - Resolves tenant and stores in HttpContext.Items["CurrentTenant"] (used by enricher)
- `Alforqan.Service/Implementation/Tenants/TenantContext.cs` - Provides ITenantContext interface for tenant resolution

