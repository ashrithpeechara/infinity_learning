# AWS Cognito Verification Email Enumeration

## Challenge Name
AWS Cognito Verification Email Enumeration

## Difficulty
Easy

---

# Scenario

Secure Corp deployed a Job Portal application using AWS Cognito for authentication and user management.

The organization configured self-registration for users through a Cognito Hosted UI integrated with an S3 static website. During the onboarding process, AWS Cognito sends verification emails to newly registered users.

As a cloud security analyst, your objective is to analyze the registration workflow and identify the email address used by Secure Corp for sending verification codes.

---

# Target Application

```text
http://aws-security-cognito-01-s3-static-website-hosting.s3-website.ap-south-1.amazonaws.com
```

---

# Objectives

1. Access the target application
2. Identify the AWS Cognito Client ID
3. Register a new user using AWS CLI
4. Trigger the verification email workflow
5. Analyze the email headers
6. Identify the sender email address

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
    "background": "#ffffff",
    "secondaryColor": "#ffffff",
    "tertiaryColor": "#ffffff"
  }
}}%%

flowchart LR

A[User Browser]
--> B[S3 Static Website]

B --> C[AWS Cognito Hosted UI]

C --> D[Cognito User Pool]

D --> E[Amazon SES Email Service]

E --> F[Verification Email]

F --> G[Email Header Analysis]

style A fill:#161b22,color:#ffffff
style B fill:#161b22,color:#ffffff
style C fill:#161b22,color:#ffffff
style D fill:#161b22,color:#ffffff
style E fill:#161b22,color:#ffffff
style F fill:#161b22,color:#ffffff
style G fill:#161b22,color:#ffffff
```

---

# Skills Learned

- AWS Cognito Enumeration
- Cognito Hosted UI Analysis
- AWS CLI Usage
- Verification Workflow Analysis
- Email Header Investigation
- Cloud Authentication Reconnaissance

---

# Tools Required

- AWS CLI
- Web Browser
- Email Account

---

# Expected Outcome

Identify the email address used by Secure Corp to send AWS Cognito verification emails.