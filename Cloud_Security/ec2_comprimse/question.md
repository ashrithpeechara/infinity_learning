Alert! Secure Corp has deployed a web application on an EC2 instance, but recent security audits have flagged potential SSRF vulnerabilities. To strengthen security, the company has engaged a penetration tester (you!) to identify and exploit any weaknesses in the infrastructure.

Your mission? Find and exploit the SSRF vulnerability to access the EC2 instance metadata, retrieve IAM credentials, and uncover the hidden flag!

Initial Access:
       The participant is provided with the URL of a web application running on an EC2 instance. The web application has an SSRF vulnerability that can be exploited to access the EC2 instance metadata.

AWS Resources:
       The EC2 instance running the web application has IAM Credentials in the metadata.

Hidden Flag:
       The flag is the Instance ID

Your Task:
Analyze the web application for vulnerabilities.
Exploit SSRF to access EC2 instance metadata.
Retrieve IAM security credentials from metadata.
Authenticate with AWS CLI and enumerate the environment.
Capture the flag – the Instance ID of the EC2 instance.