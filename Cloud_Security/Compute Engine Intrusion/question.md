# GCP SSRF to Metadata Exploitation Lab

## Objective
Identify and exploit a vulnerability in a web application hosted on a Google Cloud Platform (GCP) Compute Engine instance to extract sensitive metadata.

## Scenario
Secure Corp hosts a web application on a GCP Compute Engine instance. A vulnerability exists that allows attackers to access internal metadata via Server-Side Request Forgery (SSRF) and potentially Remote Code Execution (RCE).

## Goal
- Identify the vulnerable input field
- Exploit the vulnerability to access GCP metadata
- Recover the Service Account Email ID from the instance metadata

## Target
http://{ip.address}/index.html
where you will be given a json file the ip address will be given in that 