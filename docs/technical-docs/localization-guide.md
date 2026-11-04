# Localization Guide

## How to Change Default Language (When No Language Header is Sent)

- Go to `Alforqan.Api/ModuleAPIDependencies.cs`
- Change `options.SetDefaultCulture("ar");` to desired culture code (e.g., "en", "ar")

## How to Change Fallback Language (When Translation is Missing)

- Go to `Alforqan.Api/Localization/JSONStringLocalizer.cs`
- In `GetTranslation` method, change line 39:
  ```csharp
  if (_localizations.TryGetValue("ar", out var fallback) && fallback.TryGetValue(key, out var fallbackValue))
  ```
  Replace `"ar"` with desired fallback culture code

## How to Add New Localization Keys

1. Add key-value pair to `Alforqan.Api/Localization/en.json`:
   ```json
   {
     "YourNewKey": "Your English message"
   }
   ```

2. Add same key to `Alforqan.Api/Localization/ar.json`:
   ```json
   {
     "YourNewKey": "رسالتك بالعربية"
   }
   ```

3. (Optional) Add constant to `Alforqan.Shared/Resources/SharedResourcesKeys.cs`:
   ```csharp
   public const string YourNewKey = "YourNewKey";
   ```

## How to Use Localization in FluentValidation Validators

1. Inject `IStringLocalizer` in constructor:
   ```csharp
   private readonly IStringLocalizer localizer;
   
   public YourValidator(IStringLocalizer localizer)
   {
       this.localizer = localizer;
   }
   ```

2. Use in validation rules:
   ```csharp
   RuleFor(x => x.Property)
       .NotEmpty()
       .WithMessage(localizer[SharedResourcesKeys.Required]);
   ```

3. With parameters:
   ```csharp
   RuleFor(x => x.Property)
       .MaximumLength(100)
       .WithMessage(localizer[SharedResourcesKeys.ExceedsMaximumLengthOf] + 100);
   ```

## How to Use Localization in Data Annotations

- Data annotations are automatically localized
- Use standard attributes: `[Required]`, `[StringLength(100)]`, `[MaxLength(50)]`, etc.
- Messages are automatically retrieved from JSON files using keys like:
  - `Validation_Required`
  - `Validation_StringLength`
  - `Validation_MaxLength`
  - etc.

## How to Use Localization in Code (Non-Validation)

1. Get localizer using helper:
   ```csharp
   var localizer = LocalizerHelper.GetLocalizer();
   ```

2. Use simple lookup:
   ```csharp
   var message = localizer["YourKey"];
   ```

3. Use with parameters:
   ```csharp
   var message = localizer["YourKey", param1, param2];
   ```

## Supported Cultures

- English: "en"
- Arabic: "ar"

To add more cultures:
1. Create new JSON file: `Alforqan.Api/Localization/{culture}.json`
2. Add culture to `Alforqan.Api/ModuleAPIDependencies.cs`:
   ```csharp
   var supportedCultures = new[] { "en", "ar", "fr" }; // Add new culture
   ```

## Localization File Structure

- Location: `Alforqan.Api/Localization/`
- Format: JSON files named by culture code (e.g., `en.json`, `ar.json`)
- Structure: Key-value pairs
  ```json
  {
    "KeyName": "Translated Message",
    "KeyWithParams": "Message with {0} parameter"
  }
  ```

