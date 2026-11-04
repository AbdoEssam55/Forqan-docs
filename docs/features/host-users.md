# Host Users

## Overview

Host Users are platform administrators (e.g. SystemAdmin, DesignAdmin) stored in the Host database. The Host Users API provides CRUD, password management, and activation for these users. All endpoints require host authentication (`HostScheme`) and appropriate host claims.

## Base route

`/API/V01/Host/Users`

## Main operations

- **List / search**: `GET Single/{search}`, `GET {id}`, `POST List` (with query body)
- **Create**: `POST create` (form)
- **Update**: `PUT EditUser` (form)
- **Deactivate**: `PATCH UnActivate`
- **Password**: `PATCH ChangePassword`, `PATCH {id}/SetPassword` (admin set password)

## Related documentation

- [Host identity system](../features-explanation-docs/host-identity-system.md) – architecture, Host vs Tenant identity, DbContext separation
- [Host identity setup guide](../technical-docs/host-identity-setup-guide.md) – seeding and configuration

## Controller

`Alforqan.Api/Controllers/Host/Users/HostUserController.cs`
