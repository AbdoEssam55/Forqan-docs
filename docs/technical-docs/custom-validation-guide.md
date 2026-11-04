# Custom Validation Guide

## How to Use Data Annotations with Localized Messages

Apply standard attributes to entity properties:

```csharp
[Required]
[StringLength(100, MinimumLength = 2)]
public string Name { get; set; } = string.Empty;
```

Messages are automatically localized using keys from `SharedResourcesKeys.Validation`.

## Supported Validation Attributes

- `RequiredAttribute` → `Validation_Required`
- `StringLengthAttribute` → `Validation_StringLength` / `Validation_StringLength_Minimum`
- `MaxLengthAttribute` → `Validation_MaxLength`
- `MinLengthAttribute` → `Validation_MinLength`
- `RangeAttribute` → `Validation_Range`
- `EmailAddressAttribute` → `Validation_EmailAddress`
- `PhoneAttribute` → `Validation_Phone`
- `UrlAttribute` → `Validation_Url`
- `RegularExpressionAttribute` → `Validation_RegularExpression`
- `CompareAttribute` → `Validation_Compare`
- `CreditCardAttribute` → `Validation_CreditCard`
- `DataTypeAttribute` → `Validation_DataType`

## How to Add New Validation Message Key

1. Add key-value pair to `Alforqan.Api/Localization/en.json`:
   ```json
   {
     "Validation_YourNewAttribute": "The {0} field is invalid."
   }
   ```

2. Add same key to `Alforqan.Api/Localization/ar.json`:
   ```json
   {
     "Validation_YourNewAttribute": "حقل {0} غير صحيح."
   }
   ```

3. Add constant to `Alforqan.Shared/Resources/SharedResourcesKeys.cs`:
   ```csharp
   public static class Validation
   {
       public const string YourNewAttribute = "Validation_YourNewAttribute";
   }
   ```

4. Update `Alforqan.Api/Localization/LocalizedDataAnnotationsModelValidatorProvider.cs`:
   ```csharp
   YourNewAttributeAttribute => _localizer[SharedResourcesKeys.Validation.YourNewAttribute, displayName],
   ```

## How to Customize Validation Message for Specific Property

Use `ErrorMessage` parameter in the attribute:

```csharp
[Required(ErrorMessage = "This specific field is required")]
public string CustomField { get; set; }
```

Custom error messages override localized messages.

## How to Disable Custom Validator

In `Alforqan.Api/ModuleAPIDependencies.cs`, remove or comment out:

```csharp
options.ModelValidatorProviders.Clear();
options.ModelValidatorProviders.Add(new LocalizedDataAnnotationsModelValidatorProvider());
```

System will fall back to default ASP.NET Core validation messages (English only).

## Files to Reference

- Registration: `Alforqan.Api/ModuleAPIDependencies.cs`
- Implementation: `Alforqan.Api/Localization/LocalizedDataAnnotationsModelValidatorProvider.cs`
- Constants: `Alforqan.Shared/Resources/SharedResourcesKeys.Validation`
