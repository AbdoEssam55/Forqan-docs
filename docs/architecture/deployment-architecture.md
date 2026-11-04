# Alforqan Deployment Architecture

## Overview

Alforqan uses a **single codebase, multi-instance deployment** model with a centralized host database and distributed tenant databases for security and scalability.

---

## Architecture Model

### High-Level Structure

```
┌──────────────────────────────────────┐
│   ONE Central Host Database          │
│   - Tenant metadata (all clients)    │
│   - TenantModules                    │
│   - SystemAdmin users                │
│   - Centralized logs                 │
└──────────────────────────────────────┘
            ↑
            │ Single connection (secure)
            │
┌───────────┴──────────────────────────┐
│   Central Admin Server               │
│   - Host endpoints enabled           │
│   - SystemAdmin portal               │
│   - Tenant management                │
│   - Host user images (/Host/)        │
└──────────────────────────────────────┘

         All clients share host DB
    ┌────────┬────────┬────────┬────────┐
    │        │        │        │        │
Client A  Client B  Client C  Client D
Backend   Backend   Backend   Backend
    │        │        │        │
    ↓        ↓        ↓        ↓
Tenant   Tenant   Tenant   Tenant
DB A     DB B     DB C     DB D
```

---

## Deployment Model

### Central Admin Server

**Purpose**: System administration and tenant management

**Responsibilities**:

- ✅ Host endpoint operations (`/API/V01/Host/*`)
- ✅ Tenant metadata management via Host DB
- ✅ SystemAdmin authentication
- ✅ Cross-tenant monitoring and analytics
- ✅ Host user file storage

**Database Access**:

- **Host DB**: Read/Write (full access)
- **Tenant DBs**: Read-only or no access (depends on use case)

**File Storage**:

```
/wwwroot/
  └── Host/
      └── Images/
          └── Profile/
              └── {systemAdminUserId}.jpg
```

---

### Client Tenant Servers

**Purpose**: Serve individual tenant applications

**Responsibilities**:

- ✅ Tenant endpoint operations (`/API/V01/*`)
- ✅ Tenant user authentication
- ✅ Tenant-specific business logic
- ✅ Tenant user file storage

**Database Access**:

- **Host DB**: ❌ NO ACCESS (security isolation)
- **Tenant DB**: Read/Write (own database only)

**File Storage**:

```
/wwwroot/
  └── Tenant/
      └── Images/
          └── Profile/
              └── {tenantUserId}.jpg
```

**Security Note**: Client servers should NOT have host DB connection strings in configuration.

---

## One Codebase Strategy

### Git Repository Structure

```
Alforqan/ (Single Git Repo)
  ├── Alforqan.Api/
  │   ├── Controllers/
  │   │   ├── Host/           ← Host endpoints (admin server only)
  │   │   └── Identity/       ← Tenant endpoints (all servers)
  │   └── appsettings.json
  ├── Alforqan.Core/
  ├── Alforqan.Infrastructure/
  └── docs/
```

### Deployment Process

1. **Developer** commits to Git repository
2. **CI/CD Pipeline** builds single artifact
3. **Deploy** same artifact to:
   - Central admin server (with host DB connection)
   - All client servers (with tenant DB connections only)

### Configuration Differences

**Central Admin Server** (`appsettings.Production.json`):

```json
{
  "ConnectionStrings": {
    "HostDb": "Server=central-host-db;...",
    "TenantDb": "Server=default-tenant-db;..." // Optional
  },
  "HostEndpointsEnabled": true
}
```

**Client Server** (`appsettings.Production.json`):

```json
{
  "ConnectionStrings": {
    "TenantDb": "Server=tenant-db-client-a;..."
    // No HostDb connection string
  },
  "HostEndpointsEnabled": false // Optional security
}
```

---

## Security Model

### Defense in Depth

**Layer 1: Network Isolation**

- Client servers: Block `/API/V01/Host/*` at firewall/reverse proxy
- Admin server: Accessible only from trusted networks

**Layer 2: Database Access Control**

- Client servers: No host DB credentials
- Admin server: Host DB credentials stored securely

**Layer 3: Authentication**

- Host endpoints: Require `HostScheme` JWT with `SystemAdminRole`
- SystemAdmin users: Only exist in central host DB
- Even if client has host endpoints, no SystemAdmin users in their DB

**Layer 4: Authorization**

- `[Authorize(AuthenticationSchemes = "HostScheme", Roles = "SystemAdminRole")]`
- Tenant admins cannot access host endpoints (different role)

### Attack Scenario Protection

**If Client Server Compromised**:

- ❌ Attacker cannot access host DB (no credentials)
- ❌ Attacker cannot create SystemAdmin users (no host DB access)
- ❌ Attacker cannot access other tenants (database isolation)
- ✅ Damage limited to single tenant

---

## File Storage Strategy

### Folder Separation

**Purpose**: Prepare for multi-server deployment and clear organization

**Structure**:

```
/wwwroot/
  ├── Host/              ← Central admin server only
  │   └── Images/
  │       └── Profile/
  └── Tenant/            ← All servers
      └── Images/
          └── Profile/
```

### FileService Routing

**Host Operations**:

```csharp
var folderPath = fileService.GetFileDirectory(
    enFileType.Image,
    enFileOwner.eHostProfile);  // Routes to /Host/Images/Profile/
```

**Tenant Operations**:

```csharp
var folderPath = fileService.GetFileDirectory(
    enFileType.Image,
    enFileOwner.eProfile);  // Routes to /Tenant/Images/Profile/
```

### Future Cloud Storage

If migrating to cloud storage (Azure Blob, AWS S3):

- **Host files**: Single central container/bucket
- **Tenant files**: Per-tenant containers or shared with prefixes

---

## Database Architecture

### Host Database (Central)

**Location**: Single central database server

**Tables**:

- `Tenants` - All tenant metadata
- `TenantModules` - Feature flags per tenant
- `SystemLogs` - Centralized logging
- `AspNetUsers` - SystemAdmin users only
- `AspNetRoles` - System admin roles
- `RefreshTokens` - SystemAdmin sessions

**Access**:

- Central admin server: Full access
- Client servers: NO ACCESS

---

### Tenant Databases (Distributed)

**Location**: One database per client (can be on different servers)

**Tables**:

- `AspNetUsers` - Tenant users (Admin, Teacher, Student, Parent)
- `AspNetRoles` - Tenant roles
- `Students`, `Teachers`, `Parents`, `Halaqat`, etc.
- Tenant-specific business data

**Access**:

- Client server: Full access to own DB only
- Central admin server: Optional read-only access for analytics

---

## Scalability & Future Considerations

### Horizontal Scaling

**Current**: Single instance per client
**Future**: Multiple instances per client with load balancer

```
Client A:
  Load Balancer
      ├── Backend Instance 1
      ├── Backend Instance 2
      └── Backend Instance 3
           ↓ (all connect to)
      Tenant DB A
```

**File Storage**: Migrate to cloud storage (Azure Blob/S3) for multi-instance support

### Geographic Distribution

**Current**: All servers in one region
**Future**: Regional data centers

```
Region 1:                Region 2:
Central Admin Server     Read Replica (optional)
Clients A, B             Clients C, D
```

---

## Migration Path (Current → Target Architecture)

### Phase 1: Current State (Development)

- ✅ Single server
- ✅ Host + Tenant endpoints together
- ✅ Files in `/wwwroot/` (mixed)

### Phase 2: File Separation (In Progress)

- ✅ Separate `/Host/` and `/Tenant/` folders
- ✅ Prepare for multi-server deployment

### Phase 3: Multi-Server Deployment

- Deploy central admin server (with host DB)
- Deploy client servers (without host DB)
- Configure network isolation

### Phase 4: Production Hardening

- Cloud file storage
- Load balancers
- Database replication
- Enhanced monitoring

---

## Implementation Guidelines for Developers

### When Adding New Features

**Host Features** (SystemAdmin operations):

- Controllers in `Controllers/Host/`
- Routes use `HostRouting.HostEndpointPrefix`
- Authorization: `[Authorize(AuthenticationSchemes = "HostScheme", Roles = "SystemAdminRole")]`
- Database: Use `HostIdentityDbContext` or `HostDbContext`
- Files: Use `/Host/` folder path

**Tenant Features** (Client operations):

- Controllers in `Controllers/Identity/` or other tenant paths
- Routes use standard `CustomRouter._rule`
- Authorization: Standard tenant roles (`Admin`, `Teacher`, etc.)
- Database: Use `AppDbContext` (tenant-scoped)
- Files: Use `/Tenant/` folder path

### Testing Multi-Server Deployment Locally

1. Run two API instances with different ports
2. Instance 1: Admin server config (with host DB)
3. Instance 2: Client server config (without host DB)
4. Verify host endpoints blocked/fail on Instance 2

---

## Summary

**Key Principles**:

1. ✅ **One Codebase** - Deploy same code everywhere
2. ✅ **Security First** - Client servers never access host DB
3. ✅ **Clear Separation** - Host vs Tenant paths, files, databases
4. ✅ **Scalability Ready** - Architecture supports growth
5. ✅ **Simple Deployment** - Configuration-based differentiation

**Current Status**: Phase 2 (File Separation) in progress
**Next Steps**: Complete Phase 2, then prepare deployment scripts for Phase 3
