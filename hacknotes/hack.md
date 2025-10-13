notes for hacking


The top 20 most common mistakes web developers make that are essential for us as penetration testers are:
No.
Mistake
1. Permitting Invalid Data to Enter the Database
2. Focusing on the System as a Whole
3. Establishing Personally Developed Security Methods
4. Treating Security to be Your Last Step
5. Developing Plain Text Password Storage
6. Creating Weak Passwords
7. Storing Unencrypted Data in the Database
8. Depending Excessively on the Client Side
9. Being Too Optimistic
10. Permitting Variables via the URL Path Name
11. Trusting third-party code
12. Hard-coding backdoor accounts
13. Unverified SQL injections
14. Remote file inclusions
15. Insecure data handling
16. Failing to encrypt data properly
17. Not using a secure cryptographic system
18. Ignoring layer 8
19. Review user actions
20. Web Application Firewall misconfigurations

These mistakes lead to the OWASP Top 10 vulnerabilities for web applications, which we will discuss in other modules:
No. Vulnerability
1. Broken Access Control
2. Cryptographic Failures
3. Injection
4. Insecure Design
5. Security Misconfiguration
6. Vulnerable and Outdated Components
7. Identification and Authentication Failures
8. Software and Data Integrity Failures
9. Security Logging and Monitoring Failures
10. Server-Side Request Forgery (SSRF)


1. Broken Access Control
Restrictions are not appropriately implemented to prevent users from accessing other users accounts, viewing sensitive data, accessing unauthorized functionality, modifying data, etc.
2. Cryptographic Failures
Failures related to cryptography which often leads to sensitive data exposure or system compromise.
3. Injection
User-supplied data is not validated, filtered, or sanitized by the application. Some examples of injections are SQL injection, command injection, LDAP injection, etc.
4. Insecure Design
These issues happen when the application is not designed with security in mind.
5. Security Misconfiguration
Missing appropriate security hardening across any part of the application stack, insecure default configurations, open cloud storage, verbose error messages which disclose too much information.
6. Vulnerable and Outdated Components
Using components (both client-side and server-side) that are vulnerable, unsupported, or out of date.
7. Identification and Authentication Failures
Authentication-related attacks that target user's identity, authentication, and session management.
8. Software and Data Integrity Failures
Software and data integrity failures relate to code and infrastructure that does not protect against integrity violations. An example of this is where an application relies upon plugins, libraries, or modules from untrusted sources, repositories, and content delivery networks (CDNs).
9. Security Logging and Monitoring Failures
This category is to help detect, escalate, and respond to active breaches. Without logging and monitoring, breaches cannot be detected..
10. Server-Side Request Forgery
SSRF flaws occur whenever a web application is fetching a remote resource without validating the user-supplied URL. It allows an attacker to coerce the application to send a crafted request to an unexpected destination, even when protected by a firewall, VPN, or another type of network access control list (ACL).
