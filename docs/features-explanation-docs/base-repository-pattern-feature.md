# Base Repository Pattern Feature - Detailed Explanation

## Overview

The Base Repository Pattern in Alforqan provides a generic, reusable abstraction layer over Entity Framework Core's `DbContext`. It standardizes CRUD operations, transaction management, and soft delete functionality across all repositories, eliminating code duplication and ensuring consistent data access patterns throughout the application.

## Problem Statement

Without a base repository pattern, each repository would need to:

1. **Duplicate CRUD Logic**: Every repository would reimplement common operations like `GetByIdAsync`, `AddAsync`, `UpdateAsync`, `DeleteAsync`.

2. **Inconsistent Transaction Management**: Each repository would handle transactions differently, leading to bugs and inconsistent behavior.

3. **DbContext Type Coupling**: Repositories would be tightly coupled to specific `DbContext` types (`AppDbContext` vs `HostIdentityDbContext`), making it difficult to reuse code across different database contexts.

4. **No Standardized Soft Delete**: Each repository would implement soft delete differently, or not at all.

5. **Inconsistent SaveChanges Behavior**: Some repositories might call `SaveChangesAsync()` immediately, others might defer it, leading to unpredictable behavior.

## Architecture & Design

### Components

1. **BaseRepo<TEntity, TPrimaryKeyType, TDbContext>** (`Alforqan.Infrastructure/Implementation/Base/BaseRepo.cs`)
   - Generic base class accepting any `DbContext` type
   - Implements `IBaseRepo<TEntity, TPrimaryKeyType>`
   - Provides all standard CRUD operations
   - Manages transaction lifecycle
   - Supports soft delete via `ISoftDelete` interface

2. **AppBaseRepo<TEntity, TPrimaryKeyType>** (`Alforqan.Infrastructure/Implementation/Base/AppBaseRepo.cs`)
   - Convenience class for tenant database operations
   - Inherits from `BaseRepo<TEntity, TPrimaryKeyType, AppDbContext>`
   - Used for repositories accessing tenant-specific data

3. **HostBaseRepo<TEntity, TPrimaryKeyType>** (`Alforqan.Infrastructure/Implementation/Base/HostBaseRepo.cs`)
   - Convenience class for host database operations
   - Inherits from `BaseRepo<TEntity, TPrimaryKeyType, HostIdentityDbContext>`
   - Used for repositories accessing host/shared data (tenants, modules, host identity)

4. **IBaseRepo<TEntity, TPrimaryKeyType>** (`Alforqan.Infrastructure/Abstractions/Base/IRepositories/IBaseRepo.cs`)
   - Interface contract defining all repository operations
   - Enables dependency injection and testability
   - Provides clear contract for repository implementations

## How It Works

### Flow Diagram

```
1. Service/Handler Needs Data Access
   ↓
2. Injects IBaseRepo<TEntity, TPrimaryKeyType> (or specific repo interface)
   ↓
3. Repository Implementation (e.g., StudentRepo)
   ├─ Inherits from AppBaseRepo<Student, Guid>
   ├─ AppBaseRepo inherits from BaseRepo<Student, Guid, AppDbContext>
   └─ BaseRepo provides all CRUD methods
   ↓
4. Service Calls Repository Method
   ├─ GetByIdAsync(id)
   │  └─ BaseRepo.GetByIdAsync()
   │     └─ _dbContext.Set<TEntity>().FindAsync(id)
   ├─ AddAsync(entity)
   │  └─ BaseRepo.AddAsync()
   │     ├─ _dbContext.Set<TEntity>().AddAsync(entity)
   │     └─ _dbContext.SaveChangesAsync() (automatic)
   └─ UpdateAsync(entity)
      └─ BaseRepo.UpdateAsync()
         ├─ _dbContext.Set<TEntity>().Update(entity)
         └─ _dbContext.SaveChangesAsync() (automatic)
   ↓
5. Changes Persisted to Database
```

### Repository Inheritance Hierarchy

```
BaseRepo<TEntity, TPrimaryKeyType, TDbContext>
    ├─ AppBaseRepo<TEntity, TPrimaryKeyType>
    │   └─ StudentRepo : AppBaseRepo<Student, Guid>, IStudentRepo
    │   └─ TeacherRepo : AppBaseRepo<Teacher, Guid>, ITeacherRepo
    │
    └─ HostBaseRepo<TEntity, TPrimaryKeyType>
        └─ TenantRepo : HostBaseRepo<Tenant, Guid>, ITenantRepo
        └─ RefreshTokenRepo : HostBaseRepo<RefreshToken, Guid>, IRefreshTokenRepo
```

### Available Base Methods

**Query Methods:**
- `GetByIdAsync(TPrimaryKeyType id)` - Retrieves entity by primary key
- `GetTableNoTracking()` - Returns `IQueryable<TEntity>` with `AsNoTracking()` for read-only queries
- `GetTableAsTracking()` - Returns `IQueryable<TEntity>` with tracking enabled for updates

**Create Methods:**
- `AddAsync(TEntity entity)` - Adds single entity and saves changes
- `AddRangeAsync(ICollection<TEntity> entities)` - Adds multiple entities and saves changes

**Update Methods:**
- `UpdateAsync(TEntity entity)` - Updates single entity and saves changes
- `UpdateRangeAsync(ICollection<TEntity> entities)` - Updates multiple entities and saves changes

**Delete Methods:**
- `DeleteAsync(TEntity entity)` - Hard deletes entity and saves changes
- `DeleteRangeAsync(ICollection<TEntity> entities)` - Hard deletes multiple entities and saves changes
- `SoftDeleteAsync(ISoftDelete entity)` - Soft deletes entity (sets `IsDeleted = true`, `DeletedAt = DateTime.UtcNow`) and saves changes

**Transaction Methods:**
- `BeginTransactionAsync(CancellationToken cancellationToken = default)` - Begins a database transaction asynchronously
- `CommitAsync(CancellationToken cancellationToken = default)` - Commits the current transaction asynchronously
- `RollBackAsync(CancellationToken cancellationToken = default)` - Rolls back the current transaction asynchronously

**Utility Methods:**
- `SaveChangesAsync()` - Explicitly saves changes to database (though most methods save automatically)

### Automatic SaveChanges Behavior

Most repository methods automatically call `SaveChangesAsync()` after performing operations:

```csharp
public virtual async Task<TEntity> AddAsync(TEntity entity)
{
    await _dbContext.Set<TEntity>().AddAsync(entity);
    await _dbContext.SaveChangesAsync(); // Automatic save
    return entity;
}
```

This ensures data is persisted immediately, but can be overridden by custom repository methods that need to batch operations.

### Soft Delete Support

The repository supports soft delete for entities implementing `ISoftDelete`:

```csharp
public async Task SoftDeleteAsync(ISoftDelete entity)
{
    entity.IsDeleted = true;
    entity.DeletedAt = DateTime.UtcNow;
    _dbContext.Update(entity);
    await _dbContext.SaveChangesAsync();
}
```

This allows entities to be "deleted" without actually removing them from the database, enabling audit trails and recovery.

## Design Decisions

### Why Generic DbContext?

- **Flexibility**: Supports both `AppDbContext` (tenant DB) and `HostIdentityDbContext` (host DB) without code duplication
- **Type Safety**: Compile-time checking ensures correct `DbContext` usage
- **Extensibility**: Easy to add support for additional `DbContext` types in the future
- **Testability**: Can inject mock `DbContext` for unit testing

### Why Convenience Classes (AppBaseRepo/HostBaseRepo)?

- **Ease of Use**: Developers don't need to specify `DbContext` type every time
- **Clear Intent**: `AppBaseRepo` vs `HostBaseRepo` makes database context obvious
- **Reduced Boilerplate**: Eliminates repetitive generic type parameters
- **Consistency**: Ensures all tenant repos use `AppDbContext` and all host repos use `HostIdentityDbContext`

### Why Automatic SaveChanges?

- **Immediate Persistence**: Data is saved immediately, reducing risk of data loss
- **Consistency**: All repositories behave the same way
- **Simplicity**: Developers don't need to remember to call `SaveChangesAsync()`

**Trade-off**: For batch operations, custom repository methods can override this behavior by using `GetTableAsTracking()` and calling `SaveChangesAsync()` manually.

### Why Separate NoTracking and Tracking Methods?

- **Performance**: `AsNoTracking()` improves query performance for read-only operations
- **Explicit Intent**: Makes it clear when tracking is needed (for updates) vs not needed (for reads)
- **Memory Efficiency**: No tracking reduces memory usage for large result sets

### Why Transaction Methods?

- **Complex Operations**: Some operations require multiple repository calls within a single transaction
- **Consistency**: Ensures all-or-nothing behavior for multi-step operations
- **Control**: Allows services to manage transaction boundaries explicitly

## Integration with Existing Systems

### Service Layer

Repositories are used by services, which provide business logic and orchestration:

```csharp
public class StudentService : BaseService<Student, Guid, StudentSortingOptions>, IStudentService
{
    private readonly IStudentRepo studentRepo;
    
    public StudentService(IStudentRepo studentRepo, IServiceProvider serviceProvider)
        : base(studentRepo, serviceProvider)
    {
        this.studentRepo = studentRepo;
    }
    
    public async Task<GenericResponse<StudentDto>> GetStudentByIdAsync(Guid id)
    {
        var student = await studentRepo.GetByIdAsync(id);
        // Business logic here...
    }
}
```

### CQRS Handlers

Handlers use services (which use repositories), maintaining separation of concerns:

```csharp
public class GetStudentByIdQueryHandler : IRequestHandler<GetStudentByIdQuery, GenericResponse<StudentDto>>
{
    private readonly IStudentService studentService;
    
    public async Task<GenericResponse<StudentDto>> Handle(GetStudentByIdQuery request, CancellationToken cancellationToken)
    {
        return await studentService.GetStudentByIdAsync(request.Id);
    }
}
```

### Builders

Builders use services/repositories instead of direct `DbContext` access:

```csharp
public class CreateStudentBuilder : AppBuilder<CreateStudentCommand, StudentDto>, ICreateStudentBuilder
{
    private readonly IStudentService studentService;
    
    public async Task<GenericResponse<StudentDto>> B_CreateStudent()
    {
        var student = new Student { /* ... */ };
        var result = await studentService.AddAsync(student);
        // Handle response...
    }
}
```

### Custom Repository Methods

Repositories can extend base functionality with custom methods:

```csharp
public class StudentRepo : AppBaseRepo<Student, Guid>, IStudentRepo
{
    private readonly AppDbContext context;
    
    public StudentRepo(AppDbContext context) : base(context)
    {
        this.context = context;
    }
    
    public async Task<List<Student>> GetStudentsWithHalaqatAsync()
    {
        return await context.Students
            .Include(s => s.Halaqat)
            .Where(s => !s.IsDeleted)
            .ToListAsync();
    }
}
```

## Benefits

1. **Code Reuse**: All repositories inherit standard CRUD operations, eliminating duplication
2. **Consistency**: All repositories behave the same way, reducing bugs
3. **Type Safety**: Generic types ensure compile-time checking
4. **Maintainability**: Changes to base repository logic affect all repositories automatically
5. **Testability**: Interface-based design enables easy mocking
6. **Flexibility**: Supports multiple `DbContext` types through generics
7. **Performance**: Built-in support for `AsNoTracking()` for read-only queries
8. **Soft Delete**: Standardized soft delete support for audit trails

## Limitations & Considerations

1. **Automatic SaveChanges**: Base methods save immediately, which may not be desired for batch operations. Override with custom methods if needed.

2. **Single DbContext Per Repository**: Each repository is bound to one `DbContext` type. For operations spanning multiple contexts, use services to coordinate.

3. **No Built-in Filtering**: Base methods don't automatically filter soft-deleted entities. Use `GetTableNoTracking().Where(e => !e.IsDeleted)` explicitly.

4. **Transaction Scope**: Transaction methods operate on a single `DbContext`. For distributed transactions, consider using `UnitOfWork` pattern.

5. **Generic Constraints**: `TPrimaryKeyType` must match the entity's primary key type exactly. Mixed key types (Guid, int, string) require separate repository instances.

6. **No Built-in Pagination**: Base methods don't include pagination. Use `GetTableNoTracking()` with LINQ `Skip()` and `Take()`.

## Future Enhancements

Potential improvements:

1. **UnitOfWork Pattern**: Coordinate transactions across multiple repositories
2. **Specification Pattern**: Encapsulate complex queries as reusable specifications
3. **Automatic Soft Delete Filtering**: Option to automatically exclude soft-deleted entities
4. **Bulk Operations**: Optimized bulk insert/update methods
5. **Change Tracking**: Built-in change tracking and audit logging
6. **Query Caching**: Optional caching layer for frequently accessed data

## Related Files

- **Base Classes**: 
  - `Alforqan.Infrastructure/Implementation/Base/BaseRepo.cs`
  - `Alforqan.Infrastructure/Implementation/Base/AppBaseRepo.cs`
  - `Alforqan.Infrastructure/Implementation/Base/HostBaseRepo.cs`
- **Interface**: `Alforqan.Infrastructure/Abstractions/Base/IRepositories/IBaseRepo.cs`
- **Service Base**: `Alforqan.Service/Implementation/Base/Services/BaseService.cs`
- **Technical Guide**: `docs/technical-docs/base-repository-pattern.md`
- **Examples**: 
  - `Alforqan.Infrastructure/Implementation/Students/Repositories/StudentRepo.cs`
  - `Alforqan.Infrastructure/Implementation/Tenants/Repositories/TenantRepo.cs`
  - `Alforqan.Infrastructure/Implementation/Identity/Repositories/RefreshTokenRepo.cs`

