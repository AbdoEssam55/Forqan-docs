# FluentValidation Localization Guide

## How to Use Default Localized Messages

Use standard FluentValidation validators without explicit `.WithMessage()` calls:

```csharp
RuleFor(x => x.Name)
    .NotEmpty()           // Uses localized FV_NotEmpty message
    .NotNull()            // Uses localized FV_NotNull message
    .MinimumLength(2)     // Uses localized FV_MinimumLength message
    .MaximumLength(100);  // Uses localized FV_MaximumLength message
```

## Supported Validators

| Validator | Localization Key | Placeholders |
|-----------|-----------------|--------------|
| `NotEmpty()` | `FV_NotEmpty` | `{PropertyName}` |
| `NotNull()` | `FV_NotNull` | `{PropertyName}` |
| `Length(min, max)` | `FV_Length` | `{PropertyName}`, `{MinLength}`, `{MaxLength}`, `{TotalLength}` |
| `MinimumLength(n)` | `FV_MinimumLength` | `{PropertyName}`, `{MinLength}`, `{TotalLength}` |
| `MaximumLength(n)` | `FV_MaximumLength` | `{PropertyName}`, `{MaxLength}`, `{TotalLength}` |
| `EmailAddress()` | `FV_Email` | `{PropertyName}` |
| `Equal(value)` | `FV_Equal` | `{PropertyName}`, `{ComparisonValue}` |
| `NotEqual(value)` | `FV_NotEqual` | `{PropertyName}`, `{ComparisonValue}` |
| `GreaterThan(value)` | `FV_GreaterThan` | `{PropertyName}`, `{ComparisonValue}` |
| `GreaterThanOrEqual(value)` | `FV_GreaterThanOrEqual` | `{PropertyName}`, `{ComparisonValue}` |
| `LessThan(value)` | `FV_LessThan` | `{PropertyName}`, `{ComparisonValue}` |
| `LessThanOrEqual(value)` | `FV_LessThanOrEqual` | `{PropertyName}`, `{ComparisonValue}` |
| `InclusiveBetween(from, to)` | `FV_Between` | `{PropertyName}`, `{From}`, `{To}`, `{Value}` |
| `RegularExpression(pattern)` | `FV_RegularExpression` | `{PropertyName}` |
| `CreditCard()` | `FV_CreditCard` | `{PropertyName}` |
| `Url()` | `FV_Url` | `{PropertyName}` |
| `Phone()` | `FV_Phone` | `{PropertyName}` |

## How to Add Custom Validation Messages

For custom business rules, use explicit `.WithMessage()` with localized messages:

```csharp
using Alforqan.Shared.HelperClasses;
using Alforqan.Shared.Resources;
using Microsoft.Extensions.Localization;

public class AddTenantValidator : AbstractValidator<AddEntityCommand<TenantDto, CreateUpdateTenantDto>>
{
    private readonly IStringLocalizer localizer;
    
    public AddTenantValidator(HostDbContext hostDbContext)
    {
        localizer = LocalizerHelper.GetLocalizer();
        
        RuleFor(input => input._input.Code)
            .MustAsync(async (value, context, ct) => 
                !await hostDbContext.Tenants
                    .AnyAsync(t => t.Code == value._input.Code && !t.IsDeleted, ct))
            .WithMessage(localizer[SharedResourcesKeys.Tenant.CodeExists]);
    }
}
```

## How to Add New Localization Keys

1. Add constants to `Alforqan.Shared/Resources/SharedResourcesKeys.cs`:
   ```csharp
   public static class Validation
   {
       public const string FV_YourNewValidator = "FV_YourNewValidator";
   }
   ```

2. Add English message to `Alforqan.Api/Localization/en.json`:
   ```json
   {
     "FV_YourNewValidator": "'{PropertyName}' must meet custom requirement."
   }
   ```

3. Add Arabic message to `Alforqan.Api/Localization/ar.json`:
   ```json
   {
     "FV_YourNewValidator": "'{PropertyName}' يجب أن يلبي المتطلب المخصص."
   }
   ```

4. Map the key in `LocalizedFluentValidationLanguageManager.cs`:
   ```csharp
   private string MapFluentValidationKeyToLocalizationKey(string fluentValidationKey)
   {
       return fluentValidationKey switch
       {
           "YourNewValidator" => SharedResourcesKeys.Validation.FV_YourNewValidator,
           _ => string.Empty
       };
   }
   ```

## How to Format Placeholders

Use FluentValidation placeholders in localization messages:

```json
{
  "FV_MinimumLength": "'{PropertyName}' must be at least {MinLength} characters. You entered {TotalLength} characters."
}
```

## Files to Reference

- Implementation: `Alforqan.Core/Localization/LocalizedFluentValidationLanguageManager.cs`
- Registration: `Alforqan.Core/ModuleCoreDependencies.cs`
- Constants: `Alforqan.Shared/Resources/SharedResourcesKeys.cs`
- Localization Files: `Alforqan.Api/Localization/en.json`, `ar.json`
