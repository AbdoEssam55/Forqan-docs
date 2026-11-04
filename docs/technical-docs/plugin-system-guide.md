# Plugin System – Technical Guide

## Pattern Summary

1. **Tenant.PluginName** (Host DB): Optional string (e.g. "Default", "Formal"). Used as the DI key when resolving plugin services.
2. **IPluginFactory.GetService&lt;T&gt;():** Returns the implementation of T for the current tenant’s PluginName, with fallback to "Default".
3. **Keyed registration:** Register each implementation with a string key: `services.AddKeyedScoped<IMyService, DefaultImpl>("Default"); services.AddKeyedScoped<IMyService, FormalImpl>("Formal");`

## Adding a New Plugin Interface

1. **Define interface** in Service/Abstractions/Plugins (e.g. IMyFeatureService).
2. **Implement** one or more concrete classes (e.g. DefaultMyFeatureService, FormalMyFeatureService) in Service/Implementation/Plugins.
3. **Register** in ModuleServiceDependencies.cs with keyed scoped services:
   ```csharp
   services.AddKeyedScoped<IMyFeatureService, DefaultMyFeatureService>("Default");
   services.AddKeyedScoped<IMyFeatureService, FormalMyFeatureService>("Formal");
   ```
4. **Resolve** in handlers/builders via IPluginFactory:
   ```csharp
   var service = _pluginFactory.GetService<IMyFeatureService>();
   ```
5. **Assign** PluginName to tenants in the Host DB (e.g. "Formal") so they get the right implementation.

## Plugin Keys for Host Admin

To show a dropdown of "available plugin names" when editing a tenant:

- **GET /API/V01/Host/Tenants/available-plugin-keys** returns a list of known plugin keys (e.g. ["Default", "Formal"]). Requires host auth and ViewTenantsClaim. The list is maintained in GetAvailablePluginKeysQueryHandler (KnownPluginKeys); add new keys there when registering new plugin implementations.

## Fallback Behavior

- If tenant.PluginName is null or empty, "Default" is used.
- If no implementation is registered for the tenant’s key, IPluginFactory falls back to "Default".
- If no implementation is registered for "Default", GetService throws.

## References

- **Explanation:** [plugin-system-explained.md](../features-explanation-docs/plugin-system-explained.md).
- **Interface:** Alforqan.Service/Abstractions/Plugins/IPluginFactory.cs.
- **Implementation:** Alforqan.Service/Implementation/Plugins/PluginFactory.cs.
- **How-to (builders):** [how-to-add-tenant-plugin.md](how-to-add-tenant-plugin.md).
