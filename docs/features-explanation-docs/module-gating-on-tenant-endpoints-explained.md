# Module Gating on Tenant Endpoints – Explained

This document explains why we use `[RequiresModule]` on tenant API endpoints, how the filter works, and how it fits with frontend module gating.

---

## Problem

Tenants can have different **modules** enabled (e.g. Core, Reports, PaymentsBasic). A tenant without the Reports module must not be able to call Reports-related APIs; otherwise they could access features they are not licensed for. The backend must enforce this so that:

- Tenants without a module get a clear **403 Forbidden** (and optionally a localized message).
- The frontend can hide nav and routes for disabled modules, but direct URL or API calls are still blocked on the server.

---

## Architecture

- **`[RequiresModule(TenantModuleKey)]`** is an attribute applied to a controller or action. It declares that the endpoint requires the tenant to have that module enabled.
- **`ModuleAuthorizationFilter`** is a global async authorization filter. It runs before the action. It reads the attribute from the endpoint metadata; if no attribute is present, it does nothing. If the attribute is present, it:
  1. Ensures tenant context is available (otherwise returns 404).
  2. Checks whether the current tenant has the required module enabled (via `CurrentTenant.Modules`).
  3. If the module is not enabled, returns 403 with a localized message (e.g. "This feature requires the 'Reports' module, which is not enabled for your tenant.").
- **TenantModuleKey** (enum): Core, Reports, PaymentsBasic. Module keys are product-specific; the same mechanism applies to other products (e-commerce, clinical) with their own module enums.

---

## How It Fits with Frontend

- The frontend fetches tenant config (`GET /API/V01/TenantConfig`) which returns enabled modules. It uses `hasModule(key)` to show/hide nav items and wraps gated routes with `ModuleGate`. So the UI does not offer links to disabled features.
- The backend `[RequiresModule]` is the **authoritative** check: even if someone calls the API directly (e.g. bypassing the UI), they get 403 if the tenant does not have the module. End-to-end: frontend gates UI, backend gates API.

---

## When to Use It

- **Core** features (e.g. students list, get/create/update/delete) do **not** require the attribute; they are available to all tenants with Core (or no module restriction).
- **Reports**-scoped endpoints (e.g. `GET /API/V01/Students/{id}/Report`) must have `[RequiresModule(TenantModuleKey.Reports)]`.
- **PaymentsBasic**-scoped endpoints (e.g. payment create, list) must have `[RequiresModule(TenantModuleKey.PaymentsBasic)]`.
- When adding a **new** tenant API that belongs to a module, add the attribute so tenants without that module get 403. See the technical guide for steps and the endpoint→module reference table.

---

## Benefits and Trade-offs

- **Benefits:** Consistent enforcement; one place (filter) to check module; clear 403 and message; aligns with frontend gating.
- **Trade-offs:** Every new module-gated endpoint must have the attribute applied (and documented); the convention must be followed by all developers.
