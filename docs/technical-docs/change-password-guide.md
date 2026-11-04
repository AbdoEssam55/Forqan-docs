# Change password (tenant and host)

## Behavior

`PATCH /API/V01/Authentication/ChangePassword` (tenant) and host `Auth/ChangePassword` update the password and clear `MustChangePassword` when set. **No notification email** is sent — the API returns immediately after the password update.

Reset-password flow (`ResetPassword`) may still send email separately when configured.

## Code

- `Alforqan.Core/Builders/Implementations/Identity/Base/Authentication/Commands/ChangePasswordCommandBuilderBase.cs`
- Handlers: `AuthenticationCommandHandler`, `HostAuthenticationCommandHandler`
