You are a hired penetration tester for Secure-Corp, a company that heavily relies on Azure Virtual Machines (VMs) to host its critical applications and services. A recent security audit has flagged a potential Server-Side Request Forgery (SSRF) vulnerability in one of their web applications, which could allow attackers to extract sensitive metadata from the underlying VM.

Your mission, should you choose to accept it, is to exploit this SSRF vulnerability, extract critical metadata, retrieve an access token, and uncover the hidden flag—which in this case is the Azure Subscription ID.

Initial Access:
       The participant is provided with the URL of a web application running on an VM instance. The web application has an SSRF vulnerability that can be exploited to access the VM instance metadata.

Azure Resources:
       The VM instance running the web application can help extract the access_token through the metadata.

Hidden Flag:
       The flag is the subscription ID