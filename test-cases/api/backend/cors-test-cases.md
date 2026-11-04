# CORS — Backend Test Cases

## Metadata
- Project: `alforqan`
- Feature: `api`
- Layer: `backend`
- Routes: `-`
- APIs: `OPTIONS /*`, `POST /API/V01/Authentication/SignIn` (or current login route)
- Related code: `Alforqan/Alforqan.Api/Program.cs`
- Related docs: `docs/technical-docs/cors-guide.md`

## Scope

Browser CORS for the deployed frontend calling the Alforqan API (AllowAll policy with credentials).

## Test pages

- Frontend app origin (Coolify / production / localhost)
- Backend API base URL

## Test scenarios

- Preflight OPTIONS from the frontend origin succeeds with CORS headers.
- Credentialed login request from the frontend origin is not blocked by the browser as a CORS error.
- A genuine API 500 still returns a readable status in the Network tab (do not confuse with CORS-only failures).

## Test cases

### Wave 1 — CORS allow

| ID | Key | Type | Case | Preconditions | Steps | Expected | Requires | Related code | Test data |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| TC-01 | ALFQ-API-CORS-001 | Happy path | Verify OPTIONS preflight from the frontend origin returns CORS allow headers | 1. Backend is redeployed with the `AllowAll` CORS policy.<br>2. Frontend and API are on different origins. | 1. Open DevTools → Network on the frontend.<br>2. Trigger a credentialed API call (e.g. login).<br>3. Select the `OPTIONS` preflight for that request. | 1. Preflight status is 200 or 204.<br>2. Response includes `Access-Control-Allow-Origin` equal to the frontend origin (not `*`).<br>3. Response allows the requested method and headers. | - | `Alforqan/Alforqan.Api/Program.cs` | Frontend origin + API base URL |
| TC-02 | ALFQ-API-CORS-002 | Happy path | Verify a credentialed login request is not blocked by the browser as a CORS error | `ALFQ-API-CORS-001` succeeded | 1. Submit a valid login from the frontend against the deployed API.<br>2. Watch the console and the login response in Network. | 1. The browser console does not show a CORS blocked error for the login request.<br>2. The login response status is visible in Network (200 or a normal API 4xx). | ALFQ-API-CORS-001 | `Alforqan/Alforqan.Api/Program.cs` | Valid tenant/host login credentials |
| TC-03 | ALFQ-API-CORS-003 | Edge | Verify a server 500 is visible in Network and is not mistaken for CORS-only failure | Backend can be forced to throw (or an existing bad request path returns 500) | 1. Trigger an API call that returns 500 from the frontend origin.<br>2. Open the failed request in Network.<br>3. Check console text. | 1. Network shows HTTP 500 (or the real server status).<br>2. If CORS headers are present, treat the failure as a server error, not a CORS misconfiguration. | - | `Alforqan/Alforqan.Api/Program.cs` | Endpoint known to 500, or temporary fault injection |
