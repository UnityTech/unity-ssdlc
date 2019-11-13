# HTTP Header Security [Coding Practice]
<font size="-1">_Author: Christian Håland - Dec. 2018_</font>
## Overview
This document describes the HTTP security headers that we strongly recommend are implemented on all of Unity’s Internet-facing services. https://observatory.mozilla.org can be used to check your site's current headers. The headers are listed in order of importance, but please note that a proper secure Content Security Policy can be difficult to implement unless the application is designed from the beginning with this in mind. 

Therefore, try to get some quick wins with headers such as: `HTTP Strict Transport Security`, `X-Frame-Options`, `X-XSS-Protection` and `X-Content-Type-Options` first.

For applications that only offer web APIs; check the recommendations under [API](#api) at the bottom of the page.

- [HTTP Strict Transport Security](#http-strict-transport-security)
- [Content-Security-Policy](#content-security-polict)
- [X-Frame-Options](#x-frame-options)
- [X-XSS-Protection](#x-xss-protection)
- [X-Content-Type-Options](#x-content-type-options)
- [Referrer-Policy](#referrer-policy)
- [API](#api)

#### HTTP Strict Transport Security

The strict transport security header tells browsers to load the website over encrypted connections only from now on. This helps avoid what is called SSL stripping attacks where an attacker who are in a Man-In-The-Middle position downgrades a user to an insecure connection.

**Recommended initial setting:**

    Strict-Transport-Security: max-age=86400 

This tells the browser to remember using HTTPS for one day. _Note that you cannot switch back to HTTP for the duration of "max-age". It is wise to start with a lower value for testing and increase it once you are confident the application works as intended._

Once confident that it's working you should aim for a max-age of minimum 6 months (max-age=15768000) but preferably 2 years (max-age=63072000). 

**Recommended final setting:**
    
    Strict-Transport-Security: max-age=63072000
---
#### Content-Security-Policy (CSP)

This header aims to reduce or even eliminate the impact of XSS (Cross-Site Scripting) attacks. It can be used to tell the browser where the website is supposed to load resources from.

A nice tool for generating your policy and get an explanation for all the directives can be found here. Note that this site can also serve as a reporting endpoint to test the policy without enforcing it:

https://report-uri.io/home/generate

Quoted from Mozilla:

>The primary benefit of CSP comes from disabling the use of unsafe inline JavaScript. Inline JavaScript -- either reflected or stored -- means that improperly escaped user-inputs can generate code that is interpreted by the web browser as JavaScript. By using CSP to disable inline JavaScript, you can effectively eliminate almost all XSS attacks against your site. 

Take a look at this site for more info: https://csp.withgoogle.com/. 

Google has found that most CSP utilising whitelisting does not prevent XSS because they are bypassable due to abusable endpoints on the whitelisted domains (such as google.com). Therefore the recommended approach is no longer whitelisting but use of hashes or nonces. This is a bit more tricky to implement as it requires that every `<script>` tag has an associated randomly generated nonce that is set in the CSP header in the server response. The same nonce must be set on every script tag through e.g. passing it to the template system. E.g:

    <script src="https://www.example.com/script.js" nonce="{{ csp-nonce }}"></script>

**Example of a recommended setting:**

    Content-Security-Policy:
    object-src 'none'; 
    script-src 'nonce-{random}' 'unsafe-inline' 'unsafe-eval' 'strict-dynamic' https: http:;
    base-uri 'none';
    frame-ancestors 'none';

 

**Report-only mode:**

    Content-Security-Policy-Report-Only:
    object-src 'none'; 
    script-src 'nonce-{random}' 'unsafe-inline' 'unsafe-eval' 'strict-dynamic' https: http:;
    base-uri 'none';
    frame-ancestors 'none';
    report-uri /csp-violation-report-endpoint/  

 

Note that you would need a reporting endpoint to receive the CSP violation reports at `<your-domain-name>/csp-violation-report-endpoint` for this last example to work properly. 

---
#### X-Frame-Options

This header tells the browser if framing the website is allowed by other domains. By setting this header to a list of allowed domains or simply deny framing we can prevent Clickjacking attacks. Read more about clickjacking attacks here: https://www.owasp.org/index.php/Clickjacking

**Recommended setting:**

    X-Frame-Options: DENY

    Content-Security-Policy: frame-ancestors 'none';

X-Frame-Options is being replaced by the Content-Security Policy header's "frame-ancestors" attribute. This attribute is more flexible and should be added as well. 
 
 ---
#### X-XSS-Protection

X-XSS-Protection sets the configuration for the cross-site scripting filter built into most browsers. Read more about it here: https://scotthelme.co.uk/hardening-your-http-response-headers/#x-xss-protection

**Recommended setting:**

    X-XSS-Protection: 1; mode=block
---
#### X-Content-Type-Options

This header tells the browser not to guess the MIME (Multipurpose Internet Mail Extension) type of the content served, but instead trust the “Content-Type” header. Without the X-Content-Type-Options header set, some older browsers can incorrectly detect files as scripts and stylesheets, potentially leading to XSS attacks.

**Recommended setting:**

    X-Content-Type-Options: nosniff

---
#### Referrer-Policy

The header basically controls what is being sent in the referer header. The security benefit from using this header is to avoid situations where session identifiers or other sensitive information which is transmitted in the URL is passed to other domains via the referer header. There are different options here which range from sending no referer header whatsoever to sending only the domain (origin). The best setting must be determined based on the application's use of the referer header. The recommended setting "origin" only sends the domain name and not the path part of the URL. It is also chosen because it is supported across largest body of browsers. 

**Recommended setting:**

    Referrer-Policy: origin

We may also want to do "Referrer-Policy: strict-origin" to only send the referrer header to sites that are HTTPS to protect the user's privacy, but currently it's not supported on Edge and Safari. 

---
#### API

For APIs (Application Programming Interfaces) we want to ensure traffic is encrypted and restrict the potential impact of XSS vulnerabilities e.g. in error pages then they can't be used to run Javascript due to the Content-Security Policy not allowing scripts or iframes. Note that for APIs with documentation on the same domain such as Swagger, this won't work and would need to be adapted.

**Recommended settings:**

    Content-Security-Policy: default-src 'none'; frame-ancestors 'none'
    Strict-Transport-Security: max-age=31536000

    X-Content-Type-Options: nosniff

---

###### Resources:

For more best practices and recommendations by Mozilla on Web Application Security see this link:

- https://developers.google.com/web/fundamentals/security/csp/
- https://wiki.mozilla.org/Security/Guidelines/Web_Security
- https://csp.withgoogle.com/
- https://www.w3.org/TR/referrer-policy/
- http://caniuse.com/#feat=referrer-policy
