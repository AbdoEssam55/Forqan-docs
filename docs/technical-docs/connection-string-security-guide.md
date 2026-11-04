# Connection String Security – Technical Guide

This guide explains how to enable tenant connection string encryption, migrate existing tenants, and manage the encryption key.

## Enabling Encryption

1. **Configuration**  
   In `appsettings.json` (or environment):

   ```json
   "EncryptionSettings": {
     "Key": "your-base64-or-hex-key-or-passphrase",
     "EncryptTenantConnectionStrings": true,
     "EncryptedValuePrefix": "enc:"
   }
   ```

2. **EncryptTenantConnectionStrings**  
   Set to `true` to encrypt connection strings on save and decrypt on read. Default is `false` (backward compatible).

3. **EncryptedValuePrefix**  
   Optional prefix (e.g. `enc:`) for stored encrypted values. Only values with this prefix are decrypted; others are treated as plain text (for existing tenants). Default: `"enc:"`.

4. **Key**  
   Used by `AesSecretsProtector` when Key Vault is not configured. Should be a strong secret. Do not commit the key in source control for production; use environment variables or a secrets store.

## Key Vault (optional)

When you want the encryption key to come from Azure Key Vault instead of configuration:

1. **Create a secret in Azure Key Vault**  
   Store the encryption passphrase (same as you would use for `EncryptionSettings.Key`) as a secret, e.g. named `TenantConnectionStringsEncryptionKey`.

2. **Configuration**  
   Set both `KeyVaultUri` and `KeyVaultSecretName` (omit or leave empty `Key` when using Key Vault):

   ```json
   "EncryptionSettings": {
     "EncryptTenantConnectionStrings": true,
     "EncryptedValuePrefix": "enc:",
     "KeyVaultUri": "https://your-vault.vault.azure.net/",
     "KeyVaultSecretName": "TenantConnectionStringsEncryptionKey"
   }
   ```

3. **Authentication to Key Vault**  
   The app uses **DefaultAzureCredential**, which tries (in order) environment variables, managed identity, Azure CLI, etc. For production:
   - **Recommended**: Enable **managed identity** for the app (e.g. Azure App Service or the VM/container identity) and grant that identity **Get** and **List** on the Key Vault secrets.
   - Alternatively: set `AZURE_CLIENT_ID`, `AZURE_CLIENT_SECRET`, and `AZURE_TENANT_ID` for a service principal that has access to the secret.

4. **Registration**  
   When both `KeyVaultUri` and `KeyVaultSecretName` are set, `AddSharedDependencies` does not register `AesSecretsProtector`; `AddKeyVaultSecretsProtectorIfConfigured` (called from the API after `AddSharedDependencies`) registers `KeyVaultSecretsProtector`. The secret is fetched once at startup when the protector is first resolved.

5. **Format compatibility**  
   Ciphertext produced with Key Vault (same AES + key derivation) is compatible with `AesSecretsProtector` and vice versa; you can switch from config key to Key Vault without re-encrypting existing connection strings.

## Flow Summary

- **Save**: `TenantService.AddAsync` / `EditAsync` call `ISecretsProtector.Encrypt` when encryption is enabled and the value is not already prefixed; then set `tenant.ConnectionString = prefix + encrypted` and persist.
- **Read**: `TenantResolutionMiddleware` after loading the tenant calls `ISecretsProtector.Decrypt(tenant.ConnectionString)` when encryption is enabled; the implementation strips the prefix if present, decrypts, and sets `tenant.ConnectionString` to the decrypted value before storing in `HttpContext.Items`.

## Migrating Existing Tenants

If you turn on encryption for a database that already has plain-text connection strings:

1. **Option A – Re-save with encryption on**  
   Run a job (or use admin UI) that loads each tenant, and call the existing tenant update path (e.g. update and save). With `EncryptTenantConnectionStrings` enabled, the service will encrypt and store `prefix + ciphertext`. Existing plain values are left as-is until re-saved.

2. **Option B – Backward compatibility**  
   Values without the prefix are not decrypted; they are used as plain text. So existing tenants continue to work; only new or updated tenants get encrypted values.

## Key Management

- **Development**: You can use a fixed key in configuration (e.g. in `appsettings.Development.json`), but do not commit production keys.
- **Production**: Use environment variables (e.g. `EncryptionSettings__Key`) or Azure Key Vault (set `KeyVaultUri` and `KeyVaultSecretName`; see [Key Vault (optional)](#key-vault-optional)).
- **Key change**: Changing the key invalidates existing ciphertext. You would need to decrypt all connection strings with the old key and re-encrypt with the new key (e.g. via a migration job).

## Interface and Registration

- **ISecretsProtector**: `Alforqan.Shared/Abstractions/ISecretsProtector.cs` – `Encrypt(string)`, `Decrypt(string)`.
- **Implementations**:
  - **AesSecretsProtector**: `Alforqan.Shared/Implementation`; registered in `ModuleSharedDependencies` when Key Vault is not configured (singleton with `EncryptionSettings`).
  - **KeyVaultSecretsProtector**: `Alforqan.Infrastructure/Implementation/Secrets`; registered by `AddKeyVaultSecretsProtectorIfConfigured` (in `ModuleServiceDependencies`) when `KeyVaultUri` and `KeyVaultSecretName` are set. The API calls this after `AddSharedDependencies`.

## References

- **Explanation**: [connection-string-security-explained.md](../features-explanation-docs/connection-string-security-explained.md)
- **Tenant entity**: [tenant-entity.md](../DB%20entities/tenant-entity.md)
