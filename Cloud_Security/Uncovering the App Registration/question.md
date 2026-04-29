# Azure App Registration Red Team Challenge

## Description

Welcome to Secure Corp’s Red Team mission.

Your assignment is to investigate an Azure App Registration and uncover sensitive details that may expose security risks inside the organization’s Microsoft Entra ID (Azure AD) environment.

Recent security audits flagged potential misconfigurations in:

* Service Principals
* API Permissions
* Azure RBAC Assignments
* Microsoft Graph Permissions

As part of the Red Team operation, your goal is to simulate an attacker, enumerate permissions, and identify the hidden flag.

---

# Objective

Your objectives are:

1. Discover the Azure Tenant ID
2. Authenticate using the provided credentials
3. Enumerate Microsoft Graph permissions
4. Identify the permission assigned to the application
5. Submit the permission name as the flag

---

# Initial Access

The following application credentials were discovered during reconnaissance:

```text
Client ID     : caaa28c5-b8da-4d29-b42e-95b1aba6b81c
Client Secret : bXj8Q~_v1Y.hArjCqwQBUhCE-MwAvqB_Q1AcAa-V
Domain Name   : secure-corp.org
```

⚠️ The Tenant ID is NOT provided.

You must discover it yourself.

---

# Environment

The Azure environment contains:

* App Registrations
* Service Principals
* Microsoft Graph API Permissions
* RBAC Configurations
* OAuth2 Authentication Workflows

---

# Hidden Flag

```text
The flag is the permission assigned to the application.
```

---

# Expected Skills

Participants are expected to understand:

* OAuth2 Client Credentials Flow
* Microsoft Graph API
* Azure App Registrations
* Service Principals
* JWT Tokens
* Cloud Identity Enumeration

---

# Attack Flow

```mermaid
flowchart TD

A[Leaked Credentials] --> B[Tenant Enumeration]

B --> C[OAuth2 Authentication]

C --> D[Access Token Retrieval]

D --> E[JWT Analysis]

E --> F[Microsoft Graph Enumeration]

F --> G[Permission Discovery]
```

---

# Rules

* Perform enumeration only within the provided scope.
* Do not attack external infrastructure.
* The challenge can be solved entirely through Microsoft Graph and Azure identity enumeration.

---

# Goal

Identify the hidden Microsoft Graph permission assigned to the application.
