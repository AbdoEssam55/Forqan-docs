# CORS configuration

## Purpose

Allow the Next.js frontend (any deployed origin) to call the ASP.NET Core API with credentials.

## Steps

1. Register the policy in `Alforqan.Api/Program.cs` (`builder.Services.AddCors`):
   - Policy name: `AllowAll`
   - `SetIsOriginAllowed(_ => true)`
   - `AllowAnyMethod()`, `AllowAnyHeader()`, `AllowCredentials()`
2. Place middleware after routing and before auth:
   - `app.UseRouting();`
   - `app.UseCors("AllowAll");`
   - `app.UseAuthentication();`
   - `app.UseAuthorization();`
3. Redeploy the API after changing CORS (Coolify / container restart).

## Verify

1. From the browser on the frontend origin, call a login or health endpoint.
2. Response includes `Access-Control-Allow-Origin` matching the request `Origin` (not `*`).
3. Preflight `OPTIONS` returns 204/200 with CORS headers.
4. If the browser still reports CORS but the network tab shows 500, fix the server exception first — missing CORS headers on error responses can mask the real fault.
