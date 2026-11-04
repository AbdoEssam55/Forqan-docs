# Custom Validation Feature - Detailed Explanation

## Overview

The Custom Validation Feature provides **localized server-side validation messages** for data annotation attributes in ASP.NET Core. This ensures that validation error messages are displayed in the user's preferred language (currently Arabic and English) instead of the default English-only messages.

## Problem Statement

By default, ASP.NET Core's data annotation validation system displays error messages in English only. For a multi-lingual application like Alforqan, we need:

1. **Localized error messages** based on the user's language preference
2. **Consistent message format** across all validation attributes
3. **Easy maintenance** of validation messages in JSON files
4. **Integration** with the existing localization system

## Architecture & Design

### Components

1. **LocalizedDataAnnotationsModelValidatorProvider** (`Alforqan.Api/Localization/LocalizedDataAnnotationsModelValidatorProvider.cs`)
   - Implements `IModelValidatorProvider`
   - Intercepts the validation pipeline during model binding
   - Creates custom validators for each validation attribute

2. **LocalizedDataAnnotationsModelValidator** (same file)
   - Implements `IModelValidator`
   - Performs actual validation logic
   - Retrieves localized error messages from `IStringLocalizer`

3. **SharedResourcesKeys.Validation** (`Alforqan.Shared/Resources/SharedResourcesKeys.cs`)
   - Contains constants for all validation message keys
   - Prevents hardcoded strings in code

4. **Localization Files**
   - `Alforqan.Api/Localization/en.json` - English messages
   - `Alforqan.Api/Localization/ar.json` - Arabic messages

5. **Registration** (`Alforqan.Api/ModuleAPIDependencies.cs`)
   - Registers the custom validator provider in MVC options

## How It Works

### Flow Diagram

```
1. Client Request
   ↓
2. Model Binding (ASP.NET Core MVC)
   ↓
3. Validation Pipeline Triggered
   ↓
4. LocalizedDataAnnotationsModelValidatorProvider.CreateValidators()
   ├─ Iterates through all ValidationAttribute instances on model properties
   ├─ Creates LocalizedDataAnnotationsModelValidator for each attribute
   └─ Adds validators to context.Results
   ↓
5. LocalizedDataAnnotationsModelValidator.Validate()
   ├─ Checks if attribute validation fails (_attribute.IsValid(value))
   ├─ If invalid: GetLocalizedErrorMessage()
   │  ├─ Checks if custom ErrorMessage is set (override)
   │  ├─ Maps attribute type to SharedResourcesKeys.Validation constant
   │  ├─ Retrieves localized message from IStringLocalizer
   │  └─ Formats message with display name and parameters
   └─ Returns ModelValidationResult with localized error message
   ↓
6. Validation Results Returned to Controller
   ↓
7. ModelState Contains Localized Error Messages
```

### Interception Mechanism

The feature intercepts ASP.NET Core's validation pipeline by:

1. **Replacing default validators**: In `ModuleAPIDependencies.cs`, we clear the default `ModelValidatorProviders` and add our custom provider:
   ```csharp
   options.ModelValidatorProviders.Clear();
   options.ModelValidatorProviders.Add(new LocalizedDataAnnotationsModelValidatorProvider());
   ```

2. **IModelValidatorProvider Contract**: ASP.NET Core calls `CreateValidators()` for each model property during the validation phase. Our provider:
   - Filters for `ValidationAttribute` types
   - Wraps each attribute in our custom `LocalizedDataAnnotationsModelValidator`

3. **IModelValidator Contract**: When validation runs, ASP.NET Core calls `Validate()` on each validator. Our validator:
   - Uses the underlying attribute's validation logic (`_attribute.IsValid()`)
   - Provides localized error messages instead of default English messages

### Attribute Mapping

Each validation attribute type is mapped to a specific localization key:

| Attribute Type | Localization Key | Parameters |
|---------------|-----------------|------------|
| `RequiredAttribute` | `Validation_Required` | `{0}` = display name |
| `StringLengthAttribute` | `Validation_StringLength` or `Validation_StringLength_Minimum` | `{0}` = display name, `{1}` = max length, `{2}` = min length |
| `MaxLengthAttribute` | `Validation_MaxLength` | `{0}` = display name, `{1}` = max length |
| `MinLengthAttribute` | `Validation_MinLength` | `{0}` = display name, `{1}` = min length |
| `RangeAttribute` | `Validation_Range` | `{0}` = display name, `{1}` = min, `{2}` = max |
| `EmailAddressAttribute` | `Validation_EmailAddress` | `{0}` = display name |
| `PhoneAttribute` | `Validation_Phone` | `{0}` = display name |
| `UrlAttribute` | `Validation_Url` | `{0}` = display name |
| `RegularExpressionAttribute` | `Validation_RegularExpression` | `{0}` = display name |
| `CompareAttribute` | `Validation_Compare` | `{0}` = display name, `{1}` = other property |
| `CreditCardAttribute` | `Validation_CreditCard` | `{0}` = display name |
| `DataTypeAttribute` | `Validation_DataType` | `{0}` = display name, `{1}` = data type |

### Custom Error Message Override

If a validation attribute has a custom `ErrorMessage` set:
```csharp
[Required(ErrorMessage = "This field is mandatory")]
```
The custom message takes precedence over localized messages. This allows developers to override messages for specific properties when needed.

## Integration with Existing Systems

### Localization System

- Uses the same `IStringLocalizer` implementation (`JsonStringLocalizer`)
- Uses the same `LocalizerHelper.GetLocalizer()` pattern as FluentValidation validators
- Respects the current request culture (set via `Accept-Language` header or default culture)
- Falls back to Arabic if translation is missing (configured in `JSONStringLocalizer`)

### FluentValidation vs Data Annotations

The project uses **both** validation approaches:

- **FluentValidation**: Complex business rules, custom validations, dependency injection in validators
- **Data Annotations**: Simple property-level constraints (required, length, format) on entities

This feature ensures **data annotation validation messages are localized** to match FluentValidation's localized messages.

## Code Examples

### Basic Usage

```csharp
// Entity with localized validation
public class Student : FullAuditedEntity
{
    [Required]
    [StringLength(100, MinimumLength = 2)]
    public string Name { get; set; } = string.Empty;

    [Required]
    [EmailAddress]
    [StringLength(200)]
    public string Email { get; set; } = string.Empty;

    [Range(1, 100)]
    public int Age { get; set; }
}
```

When validation fails:
- **English**: "The Name field is required."
- **Arabic**: "حقل الاسم مطلوب."

### API Response Example

```json
{
  "errors": {
    "Name": [
      "The Name field is required."
    ],
    "Email": [
      "The Email field is not a valid email address."
    ]
  }
}
```

With Arabic culture:
```json
{
  "errors": {
    "Name": [
      "حقل الاسم مطلوب."
    ],
    "Email": [
      "حقل البريد الإلكتروني ليس عنوان بريد إلكتروني صحيح."
    ]
  }
}
```

## Benefits

1. **User Experience**: Users see error messages in their preferred language
2. **Consistency**: All validation messages follow the same localization pattern
3. **Maintainability**: Messages are centralized in JSON files, easy to update
4. **Type Safety**: Constants in `SharedResourcesKeys` prevent typos in key names
5. **Flexibility**: Custom error messages can override localized messages when needed
6. **No Breaking Changes**: Works seamlessly with existing data annotation attributes

## Limitations & Considerations

1. **Server-Side Only**: This feature only localizes server-side validation. Client-side validation (if using jQuery Validation or similar) needs separate configuration.

2. **Attribute-Specific**: Only works with standard data annotation attributes. Custom validation attributes need to be added to the switch statement in `GetLocalizedErrorMessage()`.

3. **Performance**: Minimal overhead - validator provider runs once per request, validators are created per property with validation attributes.

4. **Error Message Format**: Messages must follow the format expected by `string.Format()` with positional parameters (`{0}`, `{1}`, etc.).

## Future Enhancements

Possible improvements:

1. **Client-Side Localization**: Integrate with jQuery Validation or similar to localize client-side validation messages
2. **More Attribute Support**: Add support for additional validation attributes as needed
3. **Custom Validation Attributes**: Create project-specific validation attributes with built-in localization
4. **Caching**: Cache validation messages for performance (though current implementation is already efficient)

## Related Files

- `Alforqan.Api/Localization/LocalizedDataAnnotationsModelValidatorProvider.cs` - Core implementation
- `Alforqan.Api/ModuleAPIDependencies.cs` - Registration
- `Alforqan.Shared/Resources/SharedResourcesKeys.cs` - Validation constants
- `Alforqan.Api/Localization/en.json` - English validation messages
- `Alforqan.Api/Localization/ar.json` - Arabic validation messages
- `Alforqan.Shared/HelperClasses/LocalizerHelper.cs` - Localizer access helper
- `Alforqan.Api/Localization/JSONStringLocalizer.cs` - Localization implementation

## References

- [ASP.NET Core Model Validation](https://learn.microsoft.com/en-us/aspnet/core/mvc/models/validation)
- [Data Annotations](https://learn.microsoft.com/en-us/dotnet/api/system.componentmodel.dataannotations)
- [IModelValidatorProvider Interface](https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.modelbinding.validation.imodelvalidatorprovider)
- [IModelValidator Interface](https://learn.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.mvc.modelbinding.validation.imodelvalidator)

