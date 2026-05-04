# SimulatePrincipalPolicy Mirage

## Scenario

Secure Corp has deployed IAM resources within its AWS environment. Recent security audits have raised concerns about users with the `iam:SimulatePrincipalPolicy` permission, which could expose unintended privileges.

To strengthen cloud security, the company has engaged a penetration tester to simulate an attack and identify vulnerabilities in IAM configurations.

---

# Objective

Your objective is to identify permissions assigned to the authenticated IAM user by abusing the AWS IAM Policy Simulator.

The challenge demonstrates how attackers can enumerate effective permissions without directly viewing attached IAM policies.

---

# Initial Access

The participant is provided with:

- AWS Access Key
- AWS Secret Key

The credentials belong to an IAM user that has:

- `iam:SimulatePrincipalPolicy`

---

# AWS Resources

- IAM User
- IAM Policy Simulator API

---

# Goal

Use the AWS IAM Policy Simulator to:

1. Identify permissions assigned to the current IAM user
2. Determine which actions are allowed
3. Discover attached policy information

---

# Architecture Diagram

```mermaid
%%{init: {
  "theme": "base",
  "themeVariables": {
    "primaryColor": "#ffffff",
    "primaryTextColor": "#ff0000",
    "primaryBorderColor": "#ffffff",
    "lineColor": "#ffffff",
    "background": "#000000",
    "secondaryColor": "#000000",
    "tertiaryColor": "#000000"
  }
}}%%

flowchart LR
    A[Attacker Machine] --> B[AWS CLI]
    B --> C[Authenticated IAM User]
    C --> D[AWS IAM]
    D --> E[simulate-principal-policy API]
    E --> F[Permission Enumeration]
    F --> G[Policy Discovery]
```

---

# Skills Tested

- AWS CLI
- IAM Enumeration
- IAM Policy Simulation Abuse
- Cloud Security Assessment
- AWS Reconnaissance

---

# Learning Outcomes

After completing this lab, participants will understand:

- How `simulate-principal-policy` works
- Risks of exposing IAM simulation permissions
- IAM permission enumeration techniques
- Cloud reconnaissance methodologies
- IAM misconfiguration risks