# IAM Access Compass

## Challenge Overview

Secure Corp has requested a simulated red-team assessment of its Google Cloud Platform (GCP) IAM infrastructure. Recent internal audits revealed potential IAM misconfigurations, privilege escalation paths, and overly permissive service account relationships.

Your objective is to enumerate the environment, identify risky IAM bindings, and recover the hidden flag.

---

# Scenario

You are provided with credentials belonging to an internal service account inside Secure Corp’s GCP environment.

The cloud environment contains:

- Service Accounts
- Custom IAM Roles
- IAM Policy Bindings
- Storage Buckets
- Delegated Administrative Permissions

Your task is to:

1. Enumerate IAM roles assigned to the compromised service account
2. Discover custom role permissions
3. Identify privileged service accounts
4. Trace IAM trust relationships
5. Recover the final flag

---

# Objective

The final flag format is:

```text
CWL{BASE64_ENCODED_STRING}
```

Where:

```text
BASE64("Flag1+Flag2")
```

---

# Flag Components

| Flag | Description |
|---|---|
| Flag1 | Role name assigned to `testing-service-account` |
| Flag2 | Service account email having admin permissions over `devops-service-account` |

---

# Initial Access

Participants receive:

```text
testing-srvacc-key.json
```

Authenticate using:

```bash
gcloud auth activate-service-account --key-file testing-srvacc-key.json
```

---

# Attack Surface

- Google Cloud IAM
- Custom Roles
- Service Account Delegation
- IAM Policy Bindings
- Storage Bucket Permissions

---

# Environment Architecture

```mermaid
%%{init: {
  "theme": "base",
  "themeVariables": {
    "primaryColor": "#ffffff",
    "primaryTextColor": "#ff0000",
    "primaryBorderColor": "#ffffff",
    "lineColor": "#ffffff",
    "background": "#ffffff"
  }
}}%%

flowchart LR

A[Attacker] --> B[testing-service-account]

B --> C[Custom IAM Role]
B --> D[Project IAM Policies]

D --> E[devops-service-account]

F[prod-service-account]
 -->|Admin Permissions| E

E --> G[Privileged Resources]

G --> H[Storage Buckets]
G --> I[Secrets]
```

---

# Recon Goals

## Enumerate Projects

```bash
gcloud projects list
```

---

## Enumerate Service Accounts

```bash
gcloud iam service-accounts list
```

---

## Enumerate IAM Bindings

```bash
gcloud projects get-iam-policy PROJECT_ID
```

---

## Enumerate Service Account Policies

```bash
gcloud iam service-accounts get-iam-policy SERVICE_ACCOUNT
```

---

## Enumerate Custom Roles

```bash
gcloud iam roles list --project PROJECT_ID
```

---

# Expected Skills

- GCP IAM Enumeration
- Service Account Analysis
- IAM Privilege Escalation Discovery
- Cloud Reconnaissance
- Base64 Encoding
- Policy Mapping

---

# Tools Allowed

- gcloud CLI
- gsutil
- IAM Enumeration Scripts
- RhinoSecurityLabs GCP IAM PrivEsc Scanner

---

# Bonus

Participants may automate privilege escalation discovery using:

```bash
python3 enumerate_member_permissions.py
python3 check_for_privesc.py
```

---

# End Goal

Recover the final flag and expose the hidden IAM trust chain before the simulated attacker pivots deeper into the cloud infrastructure.