# Connection String Security Feature

## Overview

Tenant connection strings are stored in the Host database and used at runtime to configure the tenant's `AppDbContext`. To reduce risk of exposure (e.g. via backups, DB dumps, or compromised DB access), connection strings can be **encrypted at rest** in the Host DB and **decrypted only when needed** for the request (e.g. in middleware). This is implemented via a shared **secrets protection** abstraction (`ISecretsProtector`); the same abstraction can later be extended to use Key Vault or key reference.

## Why Encrypt at Rest?

- **Backups and dumps**: Plain-text connection strings in backups or exports can be misused.
- **DB access**: Anyone with read access to the Host DB could see all tenant connection strings.
- **Compliance**: Some standards require sensitive configuration to be protected at rest.

Encrypting at rest means the Host DB stores ciphertext (optionally with a small prefix like `enc:`); only the application, with the encryption key, can decrypt for use per request.

## Flow

**Write path (create/update tenant)**  
When a tenant is created or updated (e.g. in `TenantService.AddAsync` / `EditAsync`), if `EncryptTenantConnectionStrings` is enabled and the connection string is non-empty and not already encrypted (no prefix), the service calls `ISecretsProtector.Encrypt(tenant.ConnectionString)` and sets `tenant.ConnectionString` to the prefix (e.g. `enc:`) plus the ciphertext before persisting. Only this encrypted value is stored.

**Read path (per request)**  
After loading the tenant (e.g. in `TenantResolutionMiddleware`), if encryption is enabled and the connection string is non-empty, the middleware calls `ISecretsProtector.Decrypt(tenant.ConnectionString)`. The implementation strips the prefix if present, decrypts, and sets `tenant.ConnectionString` to the decrypted value before storing the tenant in `HttpContext.Items`. For the rest of the request, the tenant has a plain connection string for use (e.g. by `AppDbContext` registration); it is never written back to the DB in that form.

## Abstraction

- **ISecretsProtector**: `string Encrypt(string plainText)`, `string Decrypt(string cipherText)`.
- **Implementations**:
  - **AesSecretsProtector** (default): Symmetric encryption (AES) with a key from `EncryptionSettings.Key`. IV is generated per encryption and prepended to the ciphertext so decryption can recover it.
  - **KeyVaultSecretsProtector** (optional): Resolves the encryption key from Azure Key Vault. When `EncryptionSettings.KeyVaultUri` and `KeyVaultSecretName` are set, the app registers this implementation instead; the secret value is used as the passphrase (same key derivation as AES). Stored ciphertext format is identical, so values encrypted with either implementation are interchangeable.

## Key Vault (optional)

When both `KeyVaultUri` and `KeyVaultSecretName` are set in `EncryptionSettings`, the application uses **KeyVaultSecretsProtector** instead of **AesSecretsProtector**. The encryption key (passphrase) is fetched from the specified Azure Key Vault secret at startup; the same AES encrypt/decrypt format is used, so connection strings encrypted with the config key or the Key Vault key are compatible. This allows production to avoid storing the key in configuration; use managed identity or a service principal with access to the Key Vault secret. See the [technical guide](../technical-docs/connection-string-security-guide.md#key-vault-optional) for setup steps.

## Backward Compatibility

- **Encryption off**: When `EncryptTenantConnectionStrings` is false, no encrypt/decrypt is applied; connection strings are stored and read as plain text (current behavior).
- **Existing tenants**: Values already stored without the prefix are treated as plain text: decrypt is only applied when the configured prefix (e.g. `enc:`) is present; otherwise the value is used as-is. New or updated tenants get the prefix + ciphertext when encryption is on.

## References

- **Technical guide**: [connection-string-security-guide.md](../technical-docs/connection-string-security-guide.md) – how to enable encryption, migrate existing tenants, key management, and optional Key Vault.
- **Tenant entity**: [tenant-entity.md](../DB%20entities/tenant-entity.md) – ConnectionString Security section.
- **Roadmap**: `docs/multi-tenant-platform-scope-and-roadmap.md` § 8.2.

**Future enhancement:** Storing connection strings themselves in Key Vault (with only a reference in the Host DB) is a larger change and documented as a future enhancement.
