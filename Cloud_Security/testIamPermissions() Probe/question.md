# GCP IAM Permission Enumeration Lab

## Scenario

Alert! Secure Corp suspects that certain IAM configurations might be exposing sensitive permissions. As part of the Red Team, your mission—should you choose to accept it—is to brute-force IAM permissions and identify any misconfigurations.

Can you uncover what permissions the authenticated caller holds?

---

# Objective

Your task is to enumerate IAM permissions using the `testIamPermissions()` method and identify the hidden flag assigned to the authenticated user or service account.

---

# Initial Access

The participant is provided with the credentials of one of the employee users/service accounts.

---

# GCP Resources

- `testIamPermissions()`
- Google Cloud IAM
- Google Cloud Resource Manager API

---

# Hidden Flag

The flag is hidden inside the permission attached to the authenticated user/service account.

---

# Goal

- Authenticate using the provided service account credentials
- Enumerate IAM permissions using `testIamPermissions()`
- Identify accessible permissions
- Discover the hidden flag embedded within the assigned permission

---

# Tools & Technologies

- Kali Linux
- Google Cloud SDK (`gcloud`)
- curl
- Google Cloud IAM API

---

# Key Learning Areas

- IAM Enumeration
- Cloud Reconnaissance
- OAuth Access Tokens
- Service Account Abuse
- API-Based Permission Discovery