# Base Repository Pattern Guide

## How to Create an AppDbContext Repository

1. Create repository interface:
   ```csharp
   // Alforqan.Infrastructure/Abstractions/YourEntity/IRepositories/IYourEntityRepo.cs
   public interface IYourEntityRepo : IBaseRepo<YourEntity, Guid>
   {
       // Add custom methods here if needed
   }
   ```

2. Create repository implementation:
   ```csharp
   // Alforqan.Infrastructure/Implementation/YourEntity/Repositories/YourEntityRepo.cs
   using Microsoft.EntityFrameworkCore;
   using Alforqan.Data.Entities.YourEntity;
   using Alforqan.Infrastructure.Abstractions.YourEntity.IRepositories;
   using Alforqan.Infrastructure.Data;
   using Alforqan.Infrastructure.Implementation.Base;

   namespace Alforqan.Infrastructure.Implementation.YourEntity.Repositories
   {
       public class YourEntityRepo
           : AppBaseRepo<YourEntity, Guid>, IYourEntityRepo
       {
           #region private fields
           private readonly AppDbContext context;
           #endregion

           #region life cycle
           public YourEntityRepo(AppDbContext context) : base(context)
           {
               this.context = context;
           }
           #endregion

           #region methods
           // Add custom methods here if needed
           public async Task<List<YourEntity>> GetCustomListAsync()
           {
               return await context.YourEntities
                   .Include(x => x.RelatedEntity)
                   .ToListAsync();
           }
           #endregion
       }
   }
   ```

3. Register in DI container:
   ```csharp
   // Alforqan.Infrastructure/ModuleInfrastructureDependencies.cs
   services.AddScoped<IYourEntityRepo, YourEntityRepo>();
   ```

## How to Create a HostDbContext Repository

1. Create repository interface:
   ```csharp
   // Alforqan.Infrastructure/Abstractions/YourTenantEntity/IRepositories/IYourTenantEntityRepo.cs
   public interface IYourTenantEntityRepo : IBaseRepo<YourTenantEntity, Guid>
   {
       // Add custom methods here if needed
   }
   ```

2. Create repository implementation:
   ```csharp
   // Alforqan.Infrastructure/Implementation/YourTenantEntity/Repositories/YourTenantEntityRepo.cs
   using Microsoft.EntityFrameworkCore;
   using Alforqan.Data.Entities.YourTenantEntity;
   using Alforqan.Infrastructure.Abstractions.YourTenantEntity.IRepositories;
   using Alforqan.Infrastructure.Data;
   using Alforqan.Infrastructure.Implementation.Base;

   namespace Alforqan.Infrastructure.Implementation.YourTenantEntity.Repositories
   {
       public class YourTenantEntityRepo
           : HostBaseRepo<YourTenantEntity, Guid>, IYourTenantEntityRepo
       {
           #region private fields
           private readonly HostDbContext context;
           #endregion

           #region life cycle
           public YourTenantEntityRepo(HostDbContext context) : base(context)
           {
               this.context = context;
           }
           #endregion

           #region methods
           // Add custom methods here if needed
           #endregion
       }
   }
   ```

3. Register in DI container:
   ```csharp
   // Alforqan.Infrastructure/ModuleInfrastructureDependencies.cs
   services.AddScoped<IYourTenantEntityRepo, YourTenantEntityRepo>();
   ```

## Available Base Methods

All repositories inherit these methods from `BaseRepo`:

**Query Methods:**
```csharp
Task<TEntity> GetByIdAsync(TPrimaryKeyType id)
IQueryable<TEntity> GetTableNoTracking()
IQueryable<TEntity> GetTableAsTracking()
```

**Create Methods:**
```csharp
Task<TEntity> AddAsync(TEntity entity)
Task AddRangeAsync(ICollection<TEntity> entities)
```

**Update Methods:**
```csharp
Task<TEntity> UpdateAsync(TEntity entity)
Task UpdateRangeAsync(ICollection<TEntity> entities)
```

**Delete Methods:**
```csharp
Task DeleteAsync(TEntity entity)
Task DeleteRangeAsync(ICollection<TEntity> entities)
Task SoftDeleteAsync(ISoftDelete entity)
```

**Transaction Methods:**
```csharp
Task<IDbContextTransaction> BeginTransactionAsync(CancellationToken cancellationToken = default)
Task CommitAsync(CancellationToken cancellationToken = default)
Task RollBackAsync(CancellationToken cancellationToken = default)
```

**Utility Methods:**
```csharp
Task SaveChangesAsync()
```

## How to Access DbContext in Custom Methods

Use the private `context` field:

```csharp
public async Task<List<Student>> GetWithDepartmentAsync()
{
    return await context.Students
        .Include(s => s.Department)
        .ToListAsync();
}
```

## Repository Structure Template

```csharp
public class YourEntityRepo
    : AppBaseRepo<YourEntity, TPrimaryKey>, IYourEntityRepo
{
    #region private fields
    private readonly AppDbContext context; // or HostDbContext for host repos
    #endregion

    #region life cycle
    public YourEntityRepo(AppDbContext context) : base(context)
    {
        this.context = context;
    }
    #endregion

    #region methods
    // Custom methods here
    #endregion
}
```

## Files to Reference

- Base Classes: `Alforqan.Infrastructure/Implementation/Base/AppBaseRepo.cs`, `HostBaseRepo.cs`
- Interface: `Alforqan.Infrastructure/Abstractions/Base/IRepositories/IBaseRepo.cs`
- Examples: `StudentRepo.cs`, `TenantRepo.cs`, `RefreshTokenRepo.cs`
