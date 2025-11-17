# Web Application Security Architecture & WAF Implementation

## Project Overview
This project demonstrates the implementation of a Defense-in-Depth security strategy for a public-facing legacy web application. The objective was to reduce the attack surface, enforce encryption standards, and mitigate OWASP Top 10 vulnerabilities without refactoring the underlying application code.

**Role:** Security Architect / Analyst
**Technologies:** AWS CloudFront, AWS WAFv2, AWS Certificate Manager (ACM), Legacy Shared Hosting.

## Architecture Design
The solution utilizes a cloud-native edge security layer to protect a legacy origin server.

* **Edge Security (CDN):** AWS CloudFront handles TLS termination, ensuring strict SSL/TLS encryption (TLS 1.2+) closer to the user.
* **Application Firewall (WAF):** AWS WAFv2 is deployed at the CloudFront edge to inspect incoming traffic against managed rule sets.
* **Identity & Access Management (PKI):** Public certificates provisioned via AWS Certificate Manager (ACM) to establish a chain of trust.

## Security Controls Implemented

### 1. Threat Mitigation (WAF Rules)
To align with **NIST CSF** protection standards, the following AWS Managed Rule Groups were implemented:
* **AWSManagedRulesCommonRuleSet:** Protects against high-risk vulnerabilities like Cross-Site Scripting (XSS) and size constraints.
* **AWSManagedRulesSQLiRuleSet:** Deep packet inspection to block SQL Injection attempts.
* **AWSManagedRulesWordPressRuleSet:** Specific heuristics to block exploits targeting WordPress plugins and core files.

### 2. Data Protection (Encryption in Transit)
* Enforced **HTTPS-Only** policy at the Viewer (Client) level.
* Configured **HTTP Strict Transport Security (HSTS)** behavior via CloudFront redirects.
* Rotated legacy SSL certificates to modern ACM-provisioned certificates.

## Risk Analysis & Troubleshooting Case Study
During implementation, I encountered and resolved complex connectivity issues between the modern cloud edge and the legacy origin.

**Incident:** 403 Forbidden & 502 Bad Gateway Errors on traffic handoff.
**Root Cause Analysis:**
* **502 Error:** Identified a TLS handshake incompatibility. The legacy host did not support the ciphers required by CloudFront's strict HTTPS origin policy.
* **403 Error:** Determined that the legacy host's firewall (ModSecurity) was flagging CloudFront's high-volume requests as a bot attack.
**Remediation:**
* Implemented a split-protocol architecture: Enforcing strict HTTPS for clients (User <-> CloudFront) while using an authenticated HTTP tunnel for the origin fetch (CloudFront <-> Origin) to bypass legacy SSL limitations.
* Identified the need for IP Allow-listing at the origin firewall level to permit AWS IP ranges.

## Future Roadmap (GRC & Automation)
* **IaC Migration:** Import the manual infrastructure into Terraform state for drift detection and compliance auditing.
* **Logging:** Enable WAF logging to S3/Kinesis for integration with a SIEM (Security Information and Event Management) tool for threat hunting.
