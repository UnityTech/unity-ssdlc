# Secrets Management _(i.e., Storing Passwords Safely)_ [Coding Practice] 
<font size="-1">_Authors: Shawn Verzilli, Tim Herold - Apr. 2019_</font>

## Description

Developing and deploying secure services and applications at Unity requires integration with a number of internal and external systems, such as authentication and cloud storage. Further, much of the software development at Unity requires integration with a growing number of services, requiring developers to handle several secrets, for example: database credentials, api tokens, oauth secrets, etc.

The secrets need to be high-entropy, unique per service, and unique per environment. In many cases this will lead to many secrets for a team to manage, and as such these secrets have frequently been stored checked in as part of configuration files - an insecure practice. Once saved into a configuration file, they are frequently, and accidentally, checked into your favorite version control system on a publicly accessible cloud-hosted repositories, and hence into the hands of attackers. These secrets cannot be stored in such a manner, and need to be kept in an access-controlled environment.
### Why We Care

Secrets are secret for a reason, and often provide access to a lot of data, computing resources or privileged accounts. It is therefore crucial that we handle secrets well, ensure we limit their scope and use least privilege, and make the probability of accidental leaks as small as possible. (See Unity's Common Control Framework security controls CCF-4.04 and CCF-4.05)
### Examples of Problematic/Incorrect Secrets Use

Credential leaks to Github are a common, and troublesome issue within the industry. This can give malicious actors access to internal systems, data, apis and more. There are also other examples, such as accidentally hosting script files that contain credentials on a public web server.

It should go without saying, but when your credentials fall into someone-else's hands they can assume any of your privileges. A common example we see is someone granting themselves “Admin” so they do not have to concern themselves with identifying precisely the minimum privileges needed. They may only use these credentials for a small handful of API calls but when these credentials are exposed, a malicious actor can do anything “Admin” can do. These credentials are often exploited for launching cryptominers, ransomware attacks, or setting up backdoors for later use.

This is not limited to the example of “Admin” by any means. For example, compromising a user grants access to all groups the user may be a member of. These groups have the potential to span teams, projects, accounts, and more. This is commonly seen when a developer uses their own LDAP/Google credentials to test out a prototype of what their working on, only to later check-in that code to save their progress. It’s a bad habit to get into, despite the convenience it offers.
**To summarize, TL;DR - Commonly found secrets in code (unencrypted):**
- Raw HTML pages
- Deployed Javascript
- Public Githubs
- Open AWS Buckets
- Deployed binaries (secrets are not safe here!)
- Deployed scripts
- Forum posts

The key take-away from all the above - you should NEVER type a secret anywhere but a secrets manager. Also, obfuscation (rot13, base64 encoding, etc.), is not a valid method of protecting secrets.
### What are secrets and how to handle them?
A Secret is anything that can be used to authenticate or authorize.

- Username & Password
- API Token
- TLS Certificate

A Secret is also anything confidential, sensitive, or Personally Identifiable Information (PII)

- Credit Card
- Social Security Number
- Passport Number

#### Secrets handling best practices

To handle Secrets properly, we must consider the following:
- How do applications (non-human) get secrets?
- How do humans get secrets?
- How are secrets updated?
- How are secrets revoked?
- When were the secrets used and by who?
- What do we do in the event of a compromise?

Common examples of secret usage are:
- CI/CD Scripts
- Cloud resource access from mobile/web app
- Service account auth in backend
- Private key encryption
- Transport encryption (ex., HTTPS/TLS)
![Secrets on Fire](../images/secrets%20fire.png)


As a rule of thumb, you should use temporary credentials at every opportunity. Not all systems are built in a way that supports this and so using a secrets management tool, like Hashicorp Vault or AWS (Amazon Web Services) Secrets Manager, is strongly recommended. Environment variables are passable but not recommended. You should never store secrets in code.
##### How to protect secrets

After taking the steps above to prepare your application, leverage a secrets management infrastructure to safely store and access your secrets. Cloud options in Google Cloud KMS (Key Management Service) or Amazon AWS KMS.

The application security team has tools monitoring for secrets in source code repositories, but this mechanism is only supposed to be the last line of defense and to avoid credential leaks it is up to the developer to be diligent when handling these sensitive pieces of data.
#### Secret Stores
###### HashiCorp Vault:

Vault supports a couple of options for secrets storage - Static and Dynamic. If possible, using Dynamic secrets would be the most ideal solution, but requires more work up front, with the trade-off of time saved by secrets that are automatically managed, such as creation and can be more simply revoked.
###### Static Secrets

These are your traditional secrets. Think Username, Password, and/or API Key. These can be stored via the Vault command-line tools, for example:

    vault write secret/stg/ads/ads-performance-valuation/AWS_ACCESS_KEY_ID secret=AKIASUP3R1337DEADBEEF

_For more Vault documentaion, see the linked guides below, in the Appendix_

###### Dynamic Secrets:

Vault has a variety of Secrets Engines to hook into services like AWS, GCP (Google Cloud Platform), SSH (Secure SHell), PKI (Public Key Infrastructure), (and more) to provide temporary credentials which expire on a time-based lease procured by Vault. In the example of the AWS Secrets Engine, Vault acts as an abstraction to an AWS STS (Security Token Service) call and serves up the resultant Access/Secret Key pair.

For details on dynamic secrets, see Hashicorp’s guidance: [Dynamic Secrets Vault Support](https://learn.hashicorp.com/vault/getting-started/dynamic-secrets)

If you need support in using or troubleshooting Vault, contact the Security Team
##### GCP

The best practice for application authentication in Google Cloud is to use a Service Account. Rather than store the service account secret in the application source code, use an environment variable pointing to credentials outside of the application’s source code, such as Vault. Restrict who can act as service accounts. Users who are granted the Service Account Actor role for a service account can access all of the resources for which the service account has access.

The Security team is still working to develop more complete best practices on managing GCP secrets; in the meantime, leverage Google’s own document: Understanding Service Accounts. As always, if further instruction is required, ask the Security Team.

##### AWS Keys:

AWS keys generally come in the following format:
- `AWS_ACCESS_KEY_ID:AKIA***********VT43D`
- `AWS_SECRET_ACCESS_KEY:SYU********************************TIDHF`


These keys are generated by AWS IAM (Identity and Access Management) or AWS STS.

Keys generated by IAM: these keys are assigned to an IAM User (max. 2 keys per user), which may be used by either an individual (human) or a service account (non-human). However, while these keys can be used for service accounts, it is best practice to leverage STS to generate temporary credentials wherever possible for non-human usage.

Keys generated by STS: these keys are temporary and expire by default in 12 hours (min. 15mins, max. 36hrs). It is recommended to generate these keys with the minimum duration necessary. These keys are best practice as compromised keys quickly become useless to a malicious actor. STS also solves the problems presented by Creds in Source Code. Often times developers will use their personal Access Key/Secret Key pairs which can cause unexpected outages when someone leaves the company and their accounts are terminated; STS avoids this as well.
```python
    import boto3

    # Create a session with attached role
    session = boto3.Session()    

    # Create an sts client with the session
    sts = session.client("sts")    

    # Get temporary credentials for the current session
    temporary_creds = sts.get_session_token()['Credentials']
```



The access granted to the keys is governed by an attached IAM Policy (min.1, max. 10). If leaked, the keys grant immediate global access to all resources allowed by the attached IAM Policy.

For secure, maintainable key handling the following criteria must be adhered to:

- Never use keys generated for an individual user; always use Temporary STS keys, attached IAM Roles, or a service account (non-human IAM User). This is to make sure access to resources isn’t compromised when e.g. an individual changes roles within or leaves the company
- Ensure the policy is created on a least-privilege basis. For disparate use cases, it is better to create multiple service accounts with fewer privileges than share a single account with too many privileges.
- Use STS Tokens when possible. If role assignment is available for any service you are using in AWS, you should be using IAM Roles and STS to acquire temporary credentials.

##### Risk of Leaked Credentials

Leaking credentials can be critical as they often grant a high level of access to Unity systems and can lead to compromise of data and systems. Credential leaks are historically a serious source of breaches throughout the industry, and require very little expertise from attackers to exploit. 

###### Risk Rating:
High
 
---

##### Frequently Asked Questions - FAQ:
###### The secrets are for my dev/staging/test/fantasy environment - do those still matter?

Yes.

In the vast majority of cases, the dev/staging environments are nearly identical to production environments (usually with just differences in network design), but sometimes the same systems. Further, some teams share environments, increasing the risk of lateral movement by attackers.

Beyond the direct risks to infrastructure, the secrets themselves are now also stored on the servers where the code is hosted, as well as all the developer workstations of anyone whom as cloned or forked the code. The assumption should be the secrets are in the open, as there's limited to no way to track of they've been spread.

And of course, as is common with personal passwords, we see a high incident of re-use; so chances are, your staging secrets are the same as your production secrets.
###### What about my test code?

Similar to the answer above, but this can be slightly more nuanced. If the test code is leveraging 'fake' credentials (as in, don't actually work anywhere) to test parsing code or error handling, then this is probably fine. If these credentials are used for testing against a local, or ephemeral mock instance, then this is probably fine as well. The concern is around 'permanent' test facilities that are also hosted along side the rest of our infrastructure - wherein the same problems exists as do those in dev/staging environments.

###### References

- https://github.com/yeyintminthuhtut/Awesome-Red-Teaming#-lateral-movement

- [ Risk Rating (aka., Security Bug Bar) - (Security Process)](../Security%20Process/Risk-Rating.md)

## APPENDIX
#### Vault Guidance
Secrets Engine - https://www.vaultproject.io/docs/secrets/index.html
#### AWS Resources
- Best Practices for Managing AWS Access Keys - https://docs.aws.amazon.com/general/latest/gr/aws-access-keys-best-practices.html
- Working with AWS Secrets Manager - https://docs.aws.amazon.com/secretsmanager/latest/userguide/integrating.html
#### GCP Resources

- Understanding Service Accounts - https://cloud.google.com/iam/docs/understanding-service-accounts

- Using Environment Variables - https://cloud.google.com/functions/docs/env-var 
