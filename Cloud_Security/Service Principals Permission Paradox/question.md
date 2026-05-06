# Azure RBAC Privilege Escalation Challenge

## Description

Alert! Secure Corp’s Azure environment may contain a dangerous RBAC misconfiguration.

As a Cloud Security Specialist, your mission is to investigate Azure Active Directory (Entra ID) service principals and identify excessive IAM privileges that could lead to privilege escalation.

Your objective is to:
- Enumerate Azure RBAC assignments
- Identify dangerous permissions
- Escalate privileges
- Access sensitive Azure Storage resources
- Retrieve the hidden flag

Cloud privilege escalation is one of the most critical attack paths in modern environments. Misconfigured IAM permissions can transform a low-privileged identity into a full administrative compromise.

---

# Initial Access

The following Service Principal credentials are provided:

| Parameter | Value |
|---|---|
| Client ID | 5ee2cd9a-8ec5-4a06-a543-30ce0fc1585f |
| Client Secret | o8g8Q~jZzIZ-eoCgxSC0CDSsdwJ9pjsTRVEIJdsT |
| Domain | secure-corp.org |

---

# Objectives

1. Discover the Azure Tenant ID
2. Authenticate using the Service Principal
3. Enumerate RBAC permissions
4. Identify dangerous role assignments
5. Exploit the RBAC misconfiguration
6. Escalate privileges
7. Access the Azure Storage Account
8. Retrieve the hidden flag

---

# Environment

- Azure Active Directory (Entra ID)
- Azure RBAC
- Azure Storage Account
- Service Principals
- Azure CLI

---

# Key Learning Areas

- Azure Service Principal Enumeration
- Azure RBAC Analysis
- Privilege Escalation via Role Assignments
- Storage Account Enumeration
- Blob Storage Exfiltration
- Cloud IAM Misconfiguration Abuse

---

# Hint

Pay close attention to permissions related to:

```text
Microsoft.Authorization/roleAssignments/write
```

If a Service Principal can create role assignments, it may be able to grant itself elevated privileges within the allowed scope.

---

# Expected Outcome

Retrieve the flag hidden inside Azure Blob Storage.

---