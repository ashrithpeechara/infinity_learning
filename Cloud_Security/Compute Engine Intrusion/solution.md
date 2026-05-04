# Solution: GCP Metadata Exploitation via SSRF

---

# Architecture Overview

```mermaid
%%{init: {
  "theme": "base",
  "themeVariables": {
    "primaryColor": "#ffffff",
    "primaryTextColor": "#000000",
    "primaryBorderColor": "#4a90e2",
    "lineColor": "#555555",
    "background": "#ffffff"
  }
}}%%

flowchart LR
    A["🧑‍💻 Attacker"] --> B["🌐 Vulnerable Web Application"]
    B --> C["☁️ GCP Metadata Service"]
    C --> D["🔐 Service Account Data"]
```

---

## 1. Reconnaissance

The target web application was accessed via:

http://34.70.102.215/index.html

The homepage did not contain any visible input fields. Further enumeration led to the discovery of:

- /job_search.html

This page contained a form with multiple input fields.

```mermaid
%%{init: {
  "theme": "base",
  "themeVariables": {
    "primaryColor": "#ffffff",
    "primaryTextColor": "#000000",
    "primaryBorderColor": "#4a90e2",
    "lineColor": "#555555",
    "background": "#ffffff"
  }
}}%%

flowchart TD
    A["🌐 Target Website"] --> B["index.html"]

    B --> C["No Visible Input Fields"]

    C --> D["Directory / Endpoint Enumeration"]

    D --> E["Discovered:<br/>/job_search.html"]

    E --> F["Form with Multiple Input Parameters"]
```

![Step 1 Screenshot](./screenshots/1.png)

---

## 2. Vulnerability Identification

The following parameters were identified:

- url
- organization
- ip

Initial testing showed that the application accepts user-supplied URLs and processes them on the backend.

### Why this indicates SSRF:
If a server fetches user-provided URLs, it can be forced to access internal resources (like metadata services), which are normally inaccessible externally.

```mermaid
%%{init: {
  "theme": "base",
  "themeVariables": {
    "primaryColor": "#ffffff",
    "primaryTextColor": "#000000",
    "primaryBorderColor": "#f39c12",
    "lineColor": "#555555",
    "background": "#ffffff"
  }
}}%%

flowchart LR
    A["🧑 User Input"] --> B["url Parameter"]

    B --> C["🌐 Backend Fetches URL"]

    C --> D["🔒 Internal Resource Access"]

    D --> E["⚠️ SSRF Vulnerability"]
```

---

## 3. Exploitation (SSRF)

The vulnerability was exploited by injecting the following payload into the `url` parameter:

http://metadata/computeMetadata/v1/

### Why this URL?
- `metadata` resolves internally to `169.254.169.254`
- This is a special IP used by cloud providers (GCP) to expose instance metadata
- It is only accessible **from inside the VM**, not from the internet

### Why SSRF works here:
The application backend makes the request on our behalf, allowing us to access internal resources.

```mermaid
%%{init: {
  "theme": "base",
  "themeVariables": {
    "background": "#ffffff"
  }
}}%%

sequenceDiagram
    participant A as 🧑‍💻 Attacker
    participant B as 🌐 Web Application
    participant C as ☁️ GCP Metadata

    A->>B: Submit Metadata URL
    B->>C: Internal Request
    C-->>B: Metadata Response
    B-->>A: Return Sensitive Data
```

---

## 4. Why NOT Use Commands like curl in Input

Several attempts were made using payloads such as:

- `; curl ...`
- `$(curl ...)`
- `` `curl ...` ``

### Why these failed:
- The application does **not execute shell commands**
- It likely uses a function such as:
  - `file_get_contents()`
  - or similar HTTP-fetching logic

### Key Insight:
- Input is treated as a **URL**, not as a command
- Therefore, command injection (RCE) is not possible here
- Only SSRF-based exploitation works

```mermaid
%%{init: {
  "theme": "base",
  "themeVariables": {
    "primaryColor": "#ffffff",
    "primaryTextColor": "#000000",
    "primaryBorderColor": "#9b59b6",
    "lineColor": "#555555",
    "background": "#ffffff"
  }
}}%%

flowchart TD
    A["📥 User Payload"] --> B{"⚙️ Backend Behavior"}

    B -->|Executes Commands| C["💥 RCE Possible"]

    B -->|Fetches URLs Only| D["🌐 SSRF Exploitation"]
```

---

## 5. Metadata Enumeration

Once SSRF was confirmed, metadata enumeration was performed:

http://metadata/computeMetadata/v1/

This returned:

- instance/
- oslogin/
- project/

```mermaid
%%{init: {
  "theme": "base",
  "themeVariables": {
    "primaryColor": "#ffffff",
    "primaryTextColor": "#000000",
    "primaryBorderColor": "#27ae60",
    "lineColor": "#555555",
    "background": "#ffffff"
  }
}}%%

flowchart TD
    A["☁️ Metadata Root"] --> B["instance/"]
    A --> C["oslogin/"]
    A --> D["project/"]
```

![Step 3 Screenshot](./screenshots/2.png)

---

### Why further enumeration is required:
Metadata services are structured like directories. Sensitive data is stored in nested paths.

---

## 6. Service Account Enumeration

Next, the following endpoint was accessed:

http://metadata/computeMetadata/v1/instance/service-accounts/

This revealed:

- default/

```mermaid
%%{init: {
  "theme": "base",
  "themeVariables": {
    "primaryColor": "#ffffff",
    "primaryTextColor": "#000000",
    "primaryBorderColor": "#e67e22",
    "lineColor": "#555555",
    "background": "#ffffff"
  }
}}%%

flowchart TD
    A["instance/"] --> B["service-accounts/"]
    B --> C["default/"]
```

![Step 4 Screenshot](./screenshots/3.png)

Further enumeration:

http://metadata/computeMetadata/v1/instance/service-accounts/default/

Returned:

- email
- token
- scopes
- identity

```mermaid
%%{init: {
  "theme": "base",
  "themeVariables": {
    "primaryColor": "#ffffff",
    "primaryTextColor": "#000000",
    "primaryBorderColor": "#3498db",
    "lineColor": "#555555",
    "background": "#ffffff"
  }
}}%%

flowchart TD
    A["default/"] --> B["📧 email"]
    A --> C["🔑 token"]
    A --> D["📜 scopes"]
    A --> E["🆔 identity"]
```

---

## 7. Sensitive Data Extraction

The following endpoint was used:

http://metadata/computeMetadata/v1/instance/service-accounts/default/email

### Why this endpoint?
- It directly returns the **service account identity**
- This is the goal of the challenge
- It confirms compromise of cloud identity

```mermaid
%%{init: {
  "theme": "base",
  "themeVariables": {
    "background": "#ffffff"
  }
}}%%

sequenceDiagram
    participant A as 🧑‍💻 Attacker
    participant B as 🌐 Vulnerable App
    participant C as ☁️ Metadata Service

    A->>B: Request /default/email
    B->>C: Internal Metadata Query
    C-->>B: Service Account Email
    B-->>A: Exposed Cloud Identity
```

---

## 8. Why This Works Without curl or Headers

Normally, GCP requires:

Metadata-Flavor: Google

However:

- The lab environment allows metadata access without strict header enforcement
- This makes SSRF sufficient without needing full RCE

```mermaid
%%{init: {
  "theme": "base",
  "themeVariables": {
    "primaryColor": "#ffffff",
    "primaryTextColor": "#000000",
    "primaryBorderColor": "#16a085",
    "lineColor": "#555555",
    "background": "#ffffff"
  }
}}%%

flowchart LR
    A["☁️ Standard GCP"] --> B["Requires Metadata-Flavor Header"]

    C["🧪 Lab Environment"] --> D["Header Enforcement Disabled"]

    D --> E["✅ SSRF Access Successful"]
```

---

## 9. Flag

The recovered Service Account Email ID is:

**129668539536-compute@developer.gserviceaccount.com**

---

## 10. Impact

This vulnerability allows:

- Access to internal cloud metadata
- Exposure of service account identity
- Potential extraction of access tokens
- Risk of privilege escalation in GCP environment

```mermaid
%%{init: {
  "theme": "base",
  "themeVariables": {
    "background": "#ffffff"
  }
}}%%

mindmap
  root((⚠️ SSRF Impact))
    Metadata Access
    Service Account Exposure
    Token Extraction
    Privilege Escalation
    Cloud Compromise
```

---

## 11. Mitigation

To prevent such vulnerabilities:

- Validate and sanitize all user inputs
- Restrict outbound requests from the server
- Block access to metadata endpoints (169.254.169.254)
- Use allowlists for external requests
- Enforce metadata access protection mechanisms

```mermaid
%%{init: {
  "theme": "base",
  "themeVariables": {
    "primaryColor": "#ffffff",
    "primaryTextColor": "#000000",
    "primaryBorderColor": "#2ecc71",
    "lineColor": "#555555",
    "background": "#ffffff"
  }
}}%%

flowchart TD
    A["🛡️ Mitigation Strategies"]

    A --> B["✅ Input Validation"]

    A --> C["🌐 Restrict Outbound Requests"]

    A --> D["🚫 Block Metadata Endpoint"]

    A --> E["📋 URL Allowlisting"]

    A --> F["🔒 Metadata Protection"]
```

---

## 12. Conclusion

This lab demonstrates how a simple SSRF vulnerability can be leveraged to access internal cloud metadata and extract sensitive information such as service account identity.

The key takeaway is understanding:
- When to use SSRF vs RCE
- How cloud metadata services are structured
- Why internal endpoints must never be exposed indirectly

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
    A["🧑 User-Controlled URL"]
    --> B["🌐 Backend Request"]
    --> C["☁️ Internal Metadata Access"]
    --> D["🔓 Sensitive Data Disclosure"]
    --> E["⚠️ Potential Cloud Compromise"]
```