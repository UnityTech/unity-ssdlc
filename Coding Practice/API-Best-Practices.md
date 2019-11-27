# API Best Practices Guidelines [Coding Practice] 
<font size="-1">_Author: Frank Arana - Dec. 2018_</font>

## Overview

This document covers general security guidelines for API endpoints within Unity. These guidelines will cover general points like:
- [Access control best practices](#access-controls)
- [Input validation](#input-validation)
- [Request verification](#request-integrity)
- [Replay attacks](#replay-attacks)
- [General security practices relevant to, but not specific to APIs](#general)

### Recommendations
#### Access Controls
###### Description

API endpoints should follow the principle of least privilege. Services with protected information should serve to the smallest group possible.
###### Why We Care

APIs with misconfigured access controls can lead to unintentional information leaks, or unauthorized and malicious state-changing actions on sensitive data.
###### Example of Issue (Optional)

A POST that allows the user to modify information on an account without checking if the user owns the account being modified.

A GET request that returns sensitive informative information without authentication.

###### How to Fix?

Determine which API actions should be considered sensitive or public.

For Internal (to Unity) APIs: Limit network access as much as possible. Ideally, these endpoints should be restricted to a closed network, and require authentication stronger than a singular API key. For example, multi-factor authentication.

For Public APIs with sensitive information: Require authentication before performing any action being requested. API keys should be both revocable and renewable.

For Public APIs providing public information: Ensure no state changing actions are being performed through a public API without authentication. Consider rate limiting to prevent a single host making too many requests in a small amount of time.
###### Risk Rating

Incorrect access controls can range from High to Low Severity.
###### References (Optional)

- https://www.owasp.org/index.php/REST_Security_Cheat_Sheet

---

#### Input Validation
###### Description

Incoming data can be malformed or crafted to cause unintended behavior when it is parsed.
###### Why We Care

Depending on how input is parsed, it is possible for unvalidated input to contain command injections, or other harmful actions.
###### Example of Issue (Optional)
*TBD*

###### How to Fix?

Type checking - Ensure input is of the expected data type, reject anything else.

Length and size checking - Input should be within an expected length or size. Reject anything larger or smaller than expected.

Whitelist accepted content-types

Restrict http methods

Parsing - Third party parsers should be kept up to date, changes to internal parsers should be carefully reviewed.
###### Risk Rating

Input Validation issues could range from Low to High depending on how the error can be leveraged.

---
### Request Integrity
###### Description

It is possible that a request could be modified in-transit between the original requestor and the API endpoint.
###### Why We Care

Modified requests may cause state changing actions to the original requestor’s data, or cause incorrect, modified, or unexpected data to be served.
###### Example of Issue (Optional)

---
### Replay Attacks
An attacker sends a previous, genuine request to cause an action to happen again at a later time.

Modified requests in-transit: An attacker modifies data in the genuine request as it is sent.
###### How to Fix?

Generate signatures or HMACs (Hashed Message Authentication Codes) for each request containing sensitive data or actions. Check the signature of the request to determine if it is genuine. Sign requests that include timestamps, and deny all requests that are relatively too old.
###### Risk Rating

Depending on what actions the request can take, severity could range from High to Low.
###### References (Optional)

- https://docs.aws.amazon.com/general/latest/gr/signing_aws_api_requests.html#why-requests-are-signed

---
### General
###### Description
Common security practices relevant, but not exclusive to APIs
###### Why We Care

Even by following best security practices for APIs, it is possible to miss other malicious activity.
###### Example of Issue (Optional)

No logging or monitoring missing security events

Returning stack traces or other descriptive information of the service backend
###### How to Fix?

Log and monitor API activity. Detecting anomalies can assist in finding malicious activity that isn’t apparent anywhere else.

Disable CORS (Cross-Origin Resource Sharing) if not needed, or scope it down as small as possible to prevent forged requests or data leakage.

Return vague error responses. Put as little information as possible when returning an error to the user. Do not return any information about the server environment or debug information like stack traces.
###### Risk Rating

Ranging from Low to High depending on context.
