# Unity Security Bug Bar
<font size="-1">_Michael De Libero - Nov 2018_</font>
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

