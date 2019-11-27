# Authorization and Authentication Guidelines [Coding Practice]
<font size="-1">_Author: Carlo Valentin - Dec. 2018_</font>

## Overview

This document provides guidelines for how to implement authentication and authorization in Unity applications.These guidelines will cover general points like:
- [Use a Trusted Identity Provider](#use-a-trusted-identity-provider)
- [Require Multi-Factor Authentication for Sensitive Applications](#require-multi-factor-authentication-for-sensitive-applications)
- [Limit OAuth2 Scope by the Principle of Least Privilege](#limit-oauth2-scope-by-the-principle-of-least-privilege)
- [Limited Token Lifetimes](#limited-token-lifetimes)
- [Validating OAuth2 Redirect URIs](#validating-oauth2-redirect-uris)
- [Validating the OAuth2 State Parameter](#validating-the-oauth2-state-parameter)

**Authentication** is the process of verifying the user’s identity typically through an initial login and subsequently by verifying a unique secret identifier on each HTTP request made by the user’s browser.

**Authorization** is the process of ensuring a user can only access resources they are supposed to, usually based on some role or Risk Rating that is defined according to business logic.
## Recommendations
### Require Multi-Factor Authentication for Sensitive Applications
###### Description

During login, the user should be prompted for at least one second factor of authentication for sensitive services. Sensitive services typically include, but are not limited to: administrative panels, services that handle PII or payment information, and flagship service.
###### Why We Care

By adding multiple layers of verification we can ensure that a user’s account is not compromised, especially for sensitive applications and services. This will ensure in the case where a password (the most common initial factor of authentication) is not enough to compromise a user’s account.
###### How to Fix?

Contact the appropriate identity provider to be used at Unity, and require 2FA if needed.
###### Risk Rating

Medium

---
### Limit OAuth2 Scope by the Principle of Least Privilege
###### Description

When an OAuth2 client, a scope is defined for the the client. This scope determines which backend APIs are accessible to the client after performing authentication via the auth provider. When setting up a new OAuth2 client, the team requesting the client should identify what APIs/data they need access to, and limit the scope to those specific APIs.
###### Why We Care

Limiting the scope of our APIs is a best practice that can limit the impact of a successful attack. Also, often it is a Unity ID user that is authenticating using the given scope, and with the given access token will give the user direct access to the APIs. Since this is publicly accessible, care should be taken that the user cannot access sensitive APIs, in particular, APIs that handle personally identifiable information (PII).
###### How to Fix?

During the requirements phase, determine what information is needed for the OAuth2 client, and limit the APIs to only what is necessary. If possible, work with the auth provider to build a custom scope that will limit access to wider APIs. 
###### Risk Rating

Medium
###### References

- https://tools.ietf.org/html/rfc6819#section-3.1.1

--- 
### Limited Token Lifetimes
###### Description

When authenticating, a variety of tokens are used to prove the identity of a user. This is typically through a session cookie, or an access token attached to a user’s account. These tokens should have the shortest lifetime that balances the needs of the users with security. Tokens should not be valid for an infinite length of time, and should expire within a reasonable timeframe.
###### Why We Care

This is primarily a defense-in-depth measure, focusing on limiting the impact of an attack if these tokens are compromised. This also decreases the attack surface of a resulting application, reducing the number of valid tokens at any given time.
###### How to Fix?

Check all tokens related to authentication, and review their lifetimes. If you are unsure, contact security for recommendations on their lifetimes. When possible, use the shortest lifetime within a reasonable use.
###### Risk Rating

Medium

---
### Validating OAuth2 Redirect URIs
###### Description

When performing the OAuth2 login flow, a redirect URI is used to securely send the access token to the client web application, after it has authenticated with the auth provider. This is provided by the client during the OAuth2 flow. This URI should be validated to ensure it is a known URI that is used by the OAuth2 client.
###### Why We Care

This prevents a potential information disclosure attack when there is an open redirect on a valid redirect URI. This results in a user’s OAuth token being potentially leaked, compromising their account.
###### How to Fix?

Validate the entire URI against a whitelist of known valid URIs. Avoid using regular expressions or substring matching.

###### Examples:

_Good Example:_
    
    def validate_redirect_uri(redirect_uri):
        if redirect_uri == "https://newservice.unity3d.com/validate_oauth" :
            return True;
        return False;

_Bad Example_
    
    def validate_redirect_uri(redirect_uri):
        if "unity3d.com" in redirect_uri:
            return True;
        return False;


###### Risk Rating

High

---
### Validating the OAuth2 State Parameter
###### Description

When starting the initial OAuth2 flow, the client has the option of providing a state parameter. This state parameter is then sent back with the authorization to be validated by the client later in the flow. This state parameter should be generated upon the initial OAuth2 request, and stored to validate once the authorization from the auth provider has occurred.
###### Why We Care

This is to ensure that the OAuth2 flow is initiated by the user, and is not vulnerable to a potential CSRF (Cross-Site Request Forgery). By ensuring that the user has initiated the OAuth2 flow, they will be performing actions within their account with known permissions.
###### How to Fix?

Generate a cryptographically-secure random state parameter when the OAuth2 flow is initialized. The client should validate this when it is returned through the redirect URI callback. The parameter should only be valid for the same OAuth2 flow that initiated it.
###### Risk Rating

Low
###### References

- https://tools.ietf.org/html/rfc6819#section-3.6
