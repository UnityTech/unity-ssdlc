# Unity Security Bug Bar
<font size="-1">_Michael De Libero & Carlo Valentin - Nov 2018_</font>
## Overview
This outlines a standard rating scale for software security issues both internally and externally found. The goal for this is to set a remediation timeline expectation and to give insight into how we rate issues. The bars specified are the minimum bars, meaning some issues depending on the circumstances could be rated higher. 

### The Bug Bar
 
<table>
<tr>
<th>Severity</th>

<th>Example Bugs</th>

<th>Reasoning</th>
<tr>
<td>Critical</td>
<td>
- Internet accessible RCE

- Internet accessible SQL injection

- Stealing customer’s code from our web services (collaborate, etc..)

- RCE in the Unity Editor with no user interaction

XSS in an Asset Store page

Exploit in a service with an open port, such as Collab or Connect
</td>
<td>
Issues of this severity have either been found to be actively exploited or publicly found. This severity should be used sparingly and for issues that need people to drop everything and fix right now.
</td>
<tr>
<td>High</td>
<td>
- Persistent & Reflected XSS

- Leaked secrets or credentials

- Code execution with little user interaction

Clicking a link in a browser opens Unity Editor and runs a script

- Remote DoS (Persistent)

- Single payload DoS
</td>
<td>
These security issues can cause a big impact on our customers and/or our business. The bug should have limited mitigating security controls and should have an exploit that can be demonstrated.

---

_References:_
- [Secrets (& Credential) Management](../Coding%20Practice/Secrets-Management.md)
</td>
</tr>
<td>
Medium
</td>
<td>
- Server-Side Request Forgery

- Server-Side Request Forgery (SSRF) that allows information disclosure and discovery of our internal network.

- Any other low impact CSRF (POST-based that changes user’s “About Me” section, etc.)

- Stealing customer IP from their computer’s through our software
</td>
<td>
These issues could have a large impact to our customers and/or business. However, they often require some level of user-interaction and/or are difficult to exploit. At some level, there should be some mitigating security controls for these types of issues. While they should get fixed it should be part of an application team’s normal release cycle.
</td>
</tr>
<tr>
<td>
Low
</td>
<td>
- Non-exploitable memory corruption

- Multi-payload DoS

- Leaked HTTP headers

- Error pages leaking file system information

- Missing security headers

- “Copy-paste” attacks

- Clickjacking

- Open redirect that doesn’t expose any sensitive information
</td>
<td>
These issues are usually defense-in-depth issues. They don’t lead to actively exploited bugs but yield information or other footholds to attackers. These lowest risk issues could be used collectively to cause large problems. However, these issues should be added to the ice-box or long-term bug debt the development team has. Then when the development team has the time or is working in a similar area they should fix these issues.
</td>
</tr>
</table>
 

### FAQ:

##### Why have a bar?
The bar allows us to triage security bugs to ensure those that are most severe will get fixed before those that are less severe. This translates directly into customer protection - if we spend all our time focusing on fixing a local DoS (effectively a reliability bug), we won’t be able to address a publicly reported RCE. This, of course, doesn’t mean we never fix the local DoS, but during an incident, we need to focus resources appropriately.

##### How does the bar direct patching and releasing security updates for the Editor?
High and above will require a patch gets created and released for our customers. If at all possible we should work on getting these fixes in with patches that are already going out. Lows will be filed and the team owning the feature can address it as a normal bug.

##### What are some examples of the table above applied to actual attack types?
The below table has many examples of common attacks and how they would land on the bug bar described above:
|          |                                                             |                                              |                |                    | 
|----------|-------------------------------------------------------------|----------------------------------------------|----------------|--------------------| 
| Severity | Finding                                                     | Category                                     | Finding Impact | Finding Likelihood | 
| Critical | Internet Accessible Remote Code Execution                   | Remote Code Execution (RCE)                  | High           | High               | 
|          | Stored Cross Site Scripting on unity.com                    | Cross-Site Scripting (XSS)                   | High           | High               | 
|          | Administrative Privilege Escalation                         | Broken Access Control/Authorization (BAC)    | High           | High               | 
|          | Authentication Bypass  w/ Access to User Data               | Broken Authentication and Session Management | High           | High               | 
|          |                                                             |                                              |                |                    | 
| High     | Remote Code Execution on an isolated instance               | Remote Code Execution (RCE)                  | Medium         | High               | 
|          | Partial Privilege Escalation                                | Broken Access Control/Authorization (BAC)    | Medium         | High               | 
|          | Production Secrets in Code Repository                       | Sensitive Data Exposure                      | High           | Medium             | 
|          | Server-side Request Forgery w/ accessible internal services | Server-Side Injection                        | High           | Medium             | 
|          | Cross site request forgery for sensitive data               | Cross-Site Request Forgery (CSRF)            | Medium         | High               | 
|          | Access Token Leak resulting in full account compromise      | Sensitive Data Exposure                      | High           | Medium             | 
|          | Reflected Cross-Site Scripting                              | Cross-Site Scripting (XSS)                   | Medium         | High               | 
|          | Persistent remote denial of service                         | Denial-of-Service (DoS)                      | High           | Medium             | 
|          | No TLS/HTTPS/Transport Encryption on sensitive service      | Insecure Data Transport                      | High           | Medium             | 
|          |                                                             |                                              |                |                    | 
| Medium   | Cross site request forgery for non-sensitive data           | Cross-Site Request Forgery (CSRF)            | Medium         | Medium             | 
|          | Server-side Request Forgery (Info Disclosure)               | Server-Side Injection                        | Medium         | Medium             | 
|          | Session Fixation                                            | Broken Authentication and Session Management | Medium         | Medium             | 
|          | Access Token Leak w/ Limited Scope                          | Sensitive Data Exposure                      | Low            | High               | 
|          | Multi-factor authentication bypass                          | Broken Authentication and Session Management | High           | Low                | 
|          | Multi-packet Denial of Service                              | Denial-of-Service (DoS)                      | Medium         | Medium             | 
|          | No TLS/HTTPS/Transport Encryption on non-sensitive service  | Insecure Data Transport                      | Medium         | Medium             | 
|          | Insecure Cryptographic Options                              | Broken Cryptography                          | High           | Low                | 
|          | Publicly Accessible Google Groups                           | Insufficient Security Configurability        | Medium         | Medium             | 
|          | Cookie Flags                                                | Server Security Misconfiguration             | Low            | High               | 
|          |                                                             |                                              |                |                    | 
| Low      | Clickjacking                                                | Server Security Misconfiguration             | Low            | Medium             | 
|          | Open Redirect                                               | Unvalidated Redirects and Forwards           | Low            | Medium             | 
|          | Multi-packet DoS                                            | Denial-of-Service (DoS)                      | Low            | Medium             | 
|          | Leaked HTTP Headers                                         | Server Security Misconfiguration             | Low            | Medium             | 
|          | Information Disclosure                                      | Sensitive Data Exposure                      | Low            | Medium             | 
|          | Open CORS Policy                                            | Server Security Misconfiguration             | Low            | Medium             | 
|          | Tabnabbing                                                  | Unvalidated Redirects and Forwards           | Low            | Low                | 


