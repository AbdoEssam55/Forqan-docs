# FluentValidation Localization Feature - Detailed Explanation

## Overview

The FluentValidation Localization Feature provides **automatic localized validation messages** for FluentValidation validators in Alforqan. This ensures that validation error messages are displayed in the user's preferred language (currently Arabic and English) without requiring explicit `.WithMessage()` calls for standard validations.

## Problem Statement

By default, FluentValidation validators display error messages in English only, and developers must explicitly use `.WithMessage()` to provide localized messages. For a multi-lingual application like Alforqan, we need:

1. **Automatic localization** of default FluentValidation messages
2. **Consistent message format** across all validators
3. **Reduced boilerplate code** - no need for explicit `.WithMessage()` for standard validations
4. **Easy maintenance** of validation messages in JSON files
5. **Integration** with the existing localization system

## Architecture & Design

### Components

1. **LocalizedFluentValidationLanguageManager** (`Alforqan.Core/Localization/LocalizedFluentValidationLanguageManager.cs`)
   - Implements FluentValidation's `ILanguageManager` interface
   - Intercepts FluentValidation's default message resolution
   - Maps FluentValidation message keys to localization keys
   - Retrieves localized messages from `IStringLocalizer`

2. **SharedResourcesKeys.Validation** (`Alforqan.Shared/Resources/SharedResourcesKeys.cs`)
   - Contains constants for all FluentValidation message keys (prefixed with `FV_`)
   - Prevents hardcoded strings in code
   - Organized under `Validation` nested class

3. **Localization Files**
   - `Alforqan.Api/Localization/en.json` - English messages
   - `Alforqan.Api/Localization/ar.json` - Arabic messages

4. **Registration** (`Alforqan.Core/ModuleCoreDependencies.cs`)
   - Configures `ValidatorOptions.Global.LanguageManager` to use `LocalizedFluentValidationLanguageManager`

## How It Works

### Flow Diagram

```
1. FluentValidation Validator Execution
   ↓
2. Validator Calls Default Message Resolution
   ↓
3. LocalizedFluentValidationLanguageManager.GetString()
   ├─ Receives FluentValidation key (e.g., "NotEmptyValidator")
   ├─ Maps key to localization key (e.g., "FV_NotEmpty")
   ├─ Retrieves IStringLocalizer via LocalizerHelper.GetLocalizer()
   ├─ Gets localized message from JSON files
   └─ Returns localized message template
   ↓
4. FluentValidation Formats Message
   ├─ Replaces placeholders ({PropertyName}, {MinLength}, etc.)
   ├─ Inserts property name, values, and constraints
   └─ Returns final localized error message
   ↓
5. Validation Error Returned
   ↓
6. Error Message Displayed in User's Language
```

### Interception Mechanism

The feature intercepts FluentValidation's message resolution by:

1. **Implementing `ILanguageManager`**: The `LocalizedFluentValidationLanguageManager` implements FluentValidation's `ILanguageManager` interface, which is responsible for providing default validation messages.

2. **Global Registration**: In `ModuleCoreDependencies.cs`, we set the global language manager:
   ```csharp
   ValidatorOptions.Global.LanguageManager = new LocalizedFluentValidationLanguageManager();
   ```

3. **Automatic Message Resolution**: When a FluentValidation validator doesn't have an explicit `.WithMessage()`, it calls `ILanguageManager.GetString()` to get the default message. Our implementation intercepts this call and returns a localized message instead.

### Message Key Mapping

The system maps FluentValidation's default message keys to our localization keys:

| FluentValidation Key | Localization Key | Example Message |
|---------------------|------------------|-----------------|
| `NotEmptyValidator` | `FV_NotEmpty` | `'{PropertyName}' must not be empty.` |
| `NotNullValidator` | `FV_NotNull` | `'{PropertyName}' must not be null.` |
| `MinimumLengthValidator` | `FV_MinimumLength` | `'{PropertyName}' must be at least {MinLength} characters.` |
| `MaximumLengthValidator` | `FV_MaximumLength` | `'{PropertyName}' must not exceed {MaxLength} characters.` |
| `EmailValidator` | `FV_Email` | `'{PropertyName}' is not a valid email address.` |
| And more... | | |

### Placeholder Handling

FluentValidation uses its own placeholder syntax (e.g., `{PropertyName}`, `{MinLength}`, `{MaxLength}`, `{TotalLength}`). Our localization messages use these same placeholders, and FluentValidation replaces them automatically when formatting the error message.

**Example:**
```json
{
  "FV_MinimumLength": "'{PropertyName}' must be at least {MinLength} characters. You entered {TotalLength} characters."
}
```

When FluentValidation resolves this message:
- `{PropertyName}` → "Name"
- `{MinLength}` → "2"
- `{TotalLength}` → "1"

Result: `'Name' must be at least 2 characters. You entered 1 characters.`

## Integration with Existing Systems

### Localization System

- Uses the same `IStringLocalizer` implementation (`JsonStringLocalizer`)
- Uses the same `LocalizerHelper.GetLocalizer()` pattern as data annotation validators
- Respects the current request culture (set via `Accept-Language` header or default culture)
- Falls back to Arabic if translation is missing (configured in `JSONStringLocalizer`)

### Service Provider Access

Since `LocalizerHelper.GetLocalizer()` accesses `ServiceProviderHelper.Provider`, the service provider must be set before validators run. This is done in `Program.cs`:

```csharp
ServiceProviderHelper.Provider = app.Services;
```

### FluentValidation vs Data Annotations

The project uses **both** validation approaches:

- **FluentValidation**: Complex business rules, custom validations, async validation, dependency injection in validators
  - Now with automatic localization via `ILanguageManager`
  
- **Data Annotations**: Simple property-level validation, entity-level constraints
  - Uses `IModelValidatorProvider` for localization

Both systems now provide automatic localization without requiring explicit message specification for standard validations.

## Benefits

1. **Reduced Code**: No need for explicit `.WithMessage()` calls for standard validations
   ```csharp
   // Before: Required explicit messages
   RuleFor(x => x.Name)
       .NotEmpty().WithMessage("{PropertyName} can't be empty")
       .NotNull().WithMessage("{PropertyName} can't be empty");
   
   // After: Automatic localization
   RuleFor(x => x.Name)
       .NotEmpty()
       .NotNull();
   ```

2. **Consistency**: All validators use the same message format and localization system

3. **Maintainability**: Validation messages are centralized in JSON files, easy to update

4. **Flexibility**: Still supports explicit `.WithMessage()` for custom business rules

5. **Multilingual Support**: Messages automatically adapt to user's language preference

## Limitations

1. **Only Default Messages**: Only messages without explicit `.WithMessage()` are localized automatically. Custom messages still need manual localization.

2. **Service Provider Dependency**: Requires `ServiceProviderHelper.Provider` to be set before validators run.

3. **Placeholder Format**: FluentValidation placeholders must be used in localization messages (not .NET format `{0}`, `{1}`).

## Usage Examples

### Standard Validations (Automatic Localization)

```csharp
public class AddTenantValidator : AbstractValidator<AddEntityCommand<TenantDto, CreateUpdateTenantDto>>
{
    public AddTenantValidator()
    {
        // No .WithMessage() needed - messages are automatically localized
        RuleFor(input => input._input.Name)
            .NotEmpty()
            .NotNull()
            .MinimumLength(2)
            .MaximumLength(100);
        
        RuleFor(input => input._input.Email)
            .EmailAddress();
    }
}
```

### Custom Business Rules (Explicit Localization)

```csharp
public class AddTenantValidator : AbstractValidator<AddEntityCommand<TenantDto, CreateUpdateTenantDto>>
{
    private readonly IStringLocalizer localizer;
    
    public AddTenantValidator(HostDbContext hostDbContext)
    {
        localizer = LocalizerHelper.GetLocalizer();
        
        // Standard validation - automatic localization
        RuleFor(input => input._input.Code)
            .NotEmpty()
            .MinimumLength(2);
        
        // Custom business rule - explicit localized message
        RuleFor(input => input._input.Code)
            .MustAsync(async (value, context, ct) => 
                !await hostDbContext.Tenants
                    .AnyAsync(t => t.Code == value._input.Code && !t.IsDeleted, ct))
            .WithMessage(localizer[SharedResourcesKeys.Tenant.CodeExists]);
    }
}
```

## Comparison with Data Annotation Validation

| Feature | FluentValidation Localization | Data Annotation Localization |
|---------|------------------------------|------------------------------|
| **Interception Mechanism** | `ILanguageManager` | `IModelValidatorProvider` |
| **Registration Location** | `ModuleCoreDependencies.cs` | `ModuleAPIDependencies.cs` |
| **Placeholder Format** | FluentValidation (`{PropertyName}`) | .NET (`{0}`, `{1}`) |
| **Default Messages** | ✅ Automatic | ✅ Automatic |
| **Custom Messages** | Requires explicit `.WithMessage()` | Requires explicit `ErrorMessage` |
| **Async Support** | ✅ Yes | ❌ No |
| **Dependency Injection** | ✅ Yes (in validators) | ⚠️ Limited (via service locator) |

## Future Enhancements

Potential improvements for the future:

1. **Support More Validators**: Add mappings for additional FluentValidation validators as needed
2. **Context-Aware Messages**: Use validation context to provide more specific messages
3. **Message Templates**: Allow custom message templates per validator type
4. **Validation Rules Library**: Create a library of common validation rules with localization

## Related Files

- **Implementation**: `Alforqan.Core/Localization/LocalizedFluentValidationLanguageManager.cs`
- **Registration**: `Alforqan.Core/ModuleCoreDependencies.cs`
- **Constants**: `Alforqan.Shared/Resources/SharedResourcesKeys.cs`
- **Localization Files**: 
  - `Alforqan.Api/Localization/en.json`
  - `Alforqan.Api/Localization/ar.json`
- **Technical Guide**: `docs/technical-docs/fluentvalidation-localization-guide.md`

