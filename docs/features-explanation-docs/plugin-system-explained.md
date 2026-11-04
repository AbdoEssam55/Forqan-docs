# Plugin System (Platform)

## Overview

The platform provides a **plugin resolution** mechanism so each tenant can use a different implementation of a service (e.g. report generator, greeting) based on **Tenant.PluginName**. The platform does not define product-specific plugin types; products define their own plugin interfaces and register keyed implementations. The platform provides **Tenant.PluginName**, **IPluginFactory**, and keyed service registration only.

## Why Plugins?

- **Tenant-specific behavior:** Same feature (e.g. "report") can have different implementations (simple vs formal) per tenant.
- **No if/tenant checks:** Business code asks for a service via **IPluginFactory.GetService&lt;T&gt;()**; resolution is by tenant’s PluginName.
- **Generic:** Plugin keys are strings (e.g. "Default", "Formal"); any product can add its own interfaces and keys.

## How It Works

1. **Tenant.PluginName** (Host DB): Each tenant has an optional PluginName (string). If null or empty, "Default" is used.
2. **Keyed services:** Plugin implementations are registered in DI with a key (e.g. `AddKeyedScoped<IReportGeneratorService, DefaultReportGeneratorService>("Default")`).
3. **IPluginFactory.GetService&lt;T&gt;():** Resolves the implementation for the current tenant’s PluginName. If no implementation is found for that key, falls back to "Default". If still not found, throws.
4. **Usage:** Handlers/builders inject IPluginFactory and call `GetService<IMyPluginInterface>()`; they do not depend on a specific implementation.

## When to Use

- Use **modules** (TenantModule) to gate access to whole features (on/off per tenant).
- Use **plugins** when the same feature must behave differently per tenant (e.g. different report format, different greeting style).

## References

- **Technical guide:** [plugin-system-guide.md](../technical-docs/plugin-system-guide.md) – how to add a new plugin interface, register keyed implementations, resolve via IPluginFactory, and list plugin keys for host admin.
- **Existing feature doc:** [tenant-plugins-and-reports-feature.md](tenant-plugins-and-reports-feature.md) – modules vs plugins, report example.
- **How-to:** [how-to-add-tenant-plugin.md](../technical-docs/how-to-add-tenant-plugin.md).
