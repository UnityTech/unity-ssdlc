# Risk Rating (aka., Security Bug Bar) [Security Process]
<font size="-1">_Author: Carlo Valentin - Dec. 2018_</font>

# Overview
When vulnerabilities are identified by Application Security, the risk of the vulnerability needs to be effectively communicated to teams so that they can be fixed in a timely manner. As such, a standard set of definitions is needed for how Application Security classifies these vulnerabilities. These definitions ensure that development teams can quickly and easily understand the potential risk a vulnerability poses to their application or service. This gives a unified understanding of how to prioritize fixing known vulnerabilities.

## Description
When communicating vulnerabilities within security and to Unity’s product teams, Application Security has defined definitions and classes for the risk of each vulnerability. In determining the risk rating for a particular vulnerability, an aggregate score is used by taking the Impact and Likelihood ratings of a vulnerability and giving it a final rating. This provides the flexibility of taking different aspects of a vulnerability into account, while defining the standard by what Application Security judges vulnerabilities against.

##### Classification via Rating Matrix
In order to determine the risk rating for a vulnerability, take the identified Impact and Likelihood and use the matrix below to determine the final rating.

<table>
<tr style="background-color:#aaddff">
<td></td><th>Low Impact</th><th>Medium Impact</th><th>High Impact</th></tr>
<tr>
<th style="background-color:#aaddff">High Likelihood</th>
<td>Medium</td><td>High</td><td style="background-color:red;color:white;font-weight:bold">Critical</td>
</tr>
<tr>
<th style="background-color:#aaddff">Medium Likelihood</th>
<td>Low</td><td>Medium</td><td>High</td>
</tr>
<tr>
<th style="background-color:#aaddff">Low Likelihood</th>
<td>Low</td><td>Low</td><td>Medium</td>
</tr>
</table>





### Risk Rating
The risk rating determines the level of risk a vulnerability poses to Unity, and affects the prioritization of when to fix a bug.

The rating definitions are as follows:

- Critical - A vulnerability with immediate, likely risk of exposing highly sensitive business data or total compromise.

- High - A vulnerability with immediate risk of compromise, or a large scale breach of business data.

- Medium - A vulnerability that is difficult to exploit, but still causes a large breach of business data; a vulnerability that is easy to exploit, on a small subset of business data.

- Low - A vulnerability that has minor impact or can only be exploited in a chain of other vulnerabilities.

For more information on determining the Risk Rating/Severity here at Unity, please see our [Unity Security Bug Bar](./Bug-Bar.md).

### Impact
Impact classifies the reach and effect that occurs when the vulnerability is successfully exploited. This rating should take into account the loss of confidentiality, integrity, and availability of the targeted system. Reputational damage also plays a factor when rating impact.

Impact ratings are classified as follows:

- High - Attackers can read or modify all system data, execute arbitrary code, or escalate privileges to an administrator level. Successful execution of the vulnerability may affect a large range of users with a single exploitation.

- Medium - Attackers can read or modify some system data, deny availability to the system, or expose internal technical details of the network. Exploitation may affect a subset of users with a single exploitation.

- Low - Attackers can read or modify a small amount of system data or negatively affect other users’ experiences with the application. Exploitation may be limited in only affecting a single non-privileged target.

### Likelihood
Likelihood classifies the probability an vulnerability will be exploited. This rating should take into account the availability of information needed to exploit the vulnerability, social engineering requirements, authorization requirements, and if the issue has been publicly reported to Unity by a third-party.

Likelihood ratings are classified as follows:

- High - Attackers can exploit the vulnerability remotely, with little or no authentication or authorization. All data needed to perform the attack is publicly accessible, or has been reported to us by an external third-party.

- Medium - Attackers require non-public information to exploit, or need to perform social engineering to acquire information in order to exploit. The attack may require privileged access or is performing a blind attack without feedback.

- Low - Attackers require difficult to guess internal information or require a significant amount of social engineering to exploit the vulnerability.
