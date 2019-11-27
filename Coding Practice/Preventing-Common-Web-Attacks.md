# Preventing Common Web Attacks [Coding Practice]
<font size="-1">_Brandon Caldwell - Dec. 2018_</font>

## Overview

This guideline covers how to prevent some common vulnerability classes that can be eradicated, such as:
- [Clickjacking](#preventing-clickjacking)
- [HTTPS Downgrade (HTTP Security Header)](#all-the-cool-kids-use-http-security-headers)
- [Cross-Site Scripting (XSS)](#preventing-xss)
- [SQL Injection](#preventing-sql-injection)
- [Cross-Site Request Forgery (CSRF)](#preventing-cross-site-request-forgery)
- [Credential leaks](#credential-leaks)

By following the guidelines in this document your application will be more robust against these vulnerability classes and provide a solid foundation for developers to develop secure features for the application.

## Recommendations
### Preventing Clickjacking
###### Description

Clickjacking, also known as a "UI redress attack", occurs when an attacker uses a transparent or opaque iframe to trick a user into clicking on a button or link on another page when they were intending to click on the the top level page.
###### Why We Care

With this attack an attacker can trick the user into performing sensitive actions on a page that only the user has access to.
###### Example of Issue

Say that we have a “Pay out” button on an internal advertising payout tool and the application does not implement any protection against Clickjacking. An attacker that knows the internal URL for the tool can now include a hidden iframe on his site `www.cute-and-funny-puppies.net`:

```html
<iframe src=”https://ads-awesome-payout-tool.unity3d.com” style=”opacity:100”></iframe>
```

When an admin that gets bored of approving payouts visits the attacker’s site to view some funny dog pictures, he simultaneously gets tricked into approving a payout for an attacker by clicking an element on the site that really clicks the payout button in the hidden iframe.
###### How to Fix?

Set the following HTTP response headers in your application:

    X-Frame-Options: DENY

    Content-Security-Policy: frame-ancestors 'self';
###### Security Level

This attack is usually a pretty low risk because most application don’t have that many single click sensitive actions and also because the attack has a social engineering component that requires the user to visit a site that the attacker controls. However, in the some cases the risk can be high if there is a button such as “Make user admin” in the application.
References

https://www.owasp.org/index.php/Clickjacking_Defense_Cheat_Sheet

 ---
### All the cool kids use HTTP Security Headers
###### Description

Browser vendors are trying to help solve security issues that are common in web applications. However, they need to take backward compatibility into account because the Internet is full of old websites. Therefore, some of the new security features in browsers are opt-in, using HTTP Security headers which can be set by an application to enable these features. You should do this for all your applications to take full advantage of modern browser security.
###### Why We Care

Setting HTTP Security headers mostly protects your users by turning on security features in their browser when visiting Unity applications. We want to protect our customers so that they can safely browse our applications, even in “hostile environments” such as airports or cafés and this is a very cheap defense-in-depth security measure.  
###### Example of Issue

For example: Almost all web applications redirect from HTTP->HTTPS so that users don’t have to type https:// at the start of the url when they visit your site. However, if the user already had an active session on the site, the browser might send some sensitive information on that very first HTTP request before it gets redirected. This information can be sniffed by an attacker unless we take some precautions and tell it to always browse Unity applications over an encrypted connection. This is one example of an issue that HTTP Security headers can help with.
###### How to Fix?

We have a whole page dedicated to setting HTTP Security Headers which can be found here:
[HTTP Header Security](./Coding%20Practive/HTTP-Header-Security.md)


Read through the recommendations, add your headers in the early phases of the project, and make sure to choose a modern front-end technology that is friendly to Content Security Policy.
###### Security Level

Adding security headers is basic security hygiene and you can see that large companies such as Facebook and Google are using these and organizations such as Mozilla require all their applications to implement them. 
###### References

Mozilla has a great web security guideline reference here which includes a lot about security headers:

https://infosec.mozilla.org/guidelines/web_security

Unity's Security Team has a tool for checking how your application grades based on its security headers here:
[AppCollector](https://github.com/UnityTech/appcollector/) - _(Currently Internal Only - Public Release TBD)_

---
### Preventing XSS
###### Description

XSS or Cross-Site Scripting is perhaps the most common web application vulnerability out there. The vulnerability allows an attacker to modify the front-end behaviour of your application by injecting Javascript or HTML into the application, altering the intended behaviour for malicious purposes.
###### Why We Care

XSS vulnerabilities can be serious as they can be used to steal sensitive information, such as credentials, session tokens or credit card data. It can be used to bypass firewalls and get access to internal networks e.g. the Unity office network because it allows an attacker’s code to run in the the victim’s browser which may very well be running on a computer in the office or that’s connected via VPN. It is also commonly used to attack browsers directly by injecting browser exploit code or, as have been seen lately, to steal computing power to mine cryptocurrency by injecting currency mining code into an innocent website.
###### Example of Issue

At Unity we have several applications that have suffered from XSS vulnerabilities and still do. We have over 150 security issues in Jira related to XSS. Typically the issue arise from an input field or URL parameter that is included in the page such as:

https://www.application.unity.com/newuser?name=<script>alert(1)</script>

If this parameter ends up being treated as HTML by the application an alert box would pop up to prove we can inject Javascript.

Another example is data coming from the user that is stored in a database and later included in a webpage. E.g. an address, or a name, that the user enters.
###### How to Fix?

XSS prevention is a huge topic but the main takeaways are as follows:

- Choose a front-end framework that support output encoding by default
- Sanitize all input from users and other applications and reject unexpected input
- Implement a Content Security Policy that disallows inline Javascript

The first and best way to fix this is to use a front-end framework that output encodes / escapes data by default making it hard for any developer to make the mistake of not handling malicious input.

The following front-end frameworks are good choices that will make it harder to introduce XSS vulnerabilities as they were designed with this in mind:

- https://angular.io/guide/security#xss
- https://reactjs.org/docs/introducing-jsx.html#jsx-prevents-injection-attacks

There are more frameworks that have good default XSS prevention, but these two that are in use already at Unity and and they are supported and used by big companies such as Google and Facebook

Secondly, make sure that your application sanitizes any data coming in. This needs to be done on the server for it to be effective. Data can come from many sources including but not limited to direct input by a user or data coming from another application. For example, if you expect a phone number to be entered, make sure that your program validates that only expected data such as numbers, dashes and maybe a + for country code is accepted by the application.

There are libraries that can help with this such as Joi for NodeJS, Ruby on Rails have Active Record Validations and so on.

Finally, we can implement a safeguard mechanism against XSS issues that all modern browsers support.Content Security Policy is effective in limiting the impact of XSS vulnerabilities should  they occur in your application even after you output encode data and sanitize input.

By carefully designing a CSP we can tell the browser which scripts are allowed to run and which aren’t. We can limit where the application is allowed to load Javascript resources from, and we can disallow any inline Javascript. Malicious Javascript injected by exploiting an XSS vulnerability usually ends up being inline in the HTML document and not in an external .js file, so if we instruct the browser to not allow inline Javascript we can prevent a lot of the common XSS vulnerabilities.

An example policy that only allows script files from the domain the application lives on would look like this:

    Content-Security-Policy: "script-src 'self'"
###### Security Level

XSS vulnerabilities is very common and constitute a medium to high risk depending on context and application. It’s a vulnerability class that requires a lot of resources from the application security team to deal with, unless the application is well designed and we would very much like to extinguish this vulnerability class in Unity applications.
###### References
- https://www.owasp.org/index.php/XSS_(Cross_Site_Scripting)_Prevention_Cheat_Sheet
- https://csp.withgoogle.com/

---
### Preventing SQL Injection
###### Description

SQL injection is an old vulnerability class which allows an attacker to alter an application’s SQL queries to perform other actions than those intended by the developer. The problem occurs due to a lack of separation between data and code, allowing the an attacker to input data to alter the logic of a query. SQL injection vulnerabilities are still found even in modern web applications leading to loss of data, authentication bypass and server compromise.

Even applications that do not use traditional relational databases have been found to be vulnerable to SQL injection. If user input is used to build the database query, an attacker can potentially alter the query being made and bypass authentication, get unintended access to data or remote code execution.
###### Why We Care

Protecting data from attackers is paramount for any data-driven company, and protecting our web applications from SQL injection is a very important part of preventing data from being stolen. Many of the breaches that have been seen over the years have been caused by SQL injection in web applications leading either to full server compromise or access to all data in the database.
###### Example of Issue

We’ve found examples of SQL injection in both the old Asset Store and in Multiplay’s Gameforge. In both cases the vulnerability allowed us to access and extract all the content of the application’s database.
###### How to Fix?
###### _Relational Databases (MySQL, PostgreSQL, ...)_

All SQL queries should be parameterized queries, the exact syntax varies based on technologystack used, but usually looks more or less like this:

```sql
SELECT * FROM users WHERE username = ‘?’, username
```

Sanitize all input, but especially input that you intend to use in a SQL query.
Limit the privileges of your database user. You should never run application SQL queries as the database admin or a root / admin user.

Use an Object Relational Mapper (ORM) that helps you build queries securely and also eases the pain of writing raw SQL queries.
 
###### _noSQL Databases (MongoDB, CouchDB, ...)_

Almost the same principles apply for protecting applications against noSQL injection: 

Sanitize user input that is used in queries. Check the type of the input (string, array, dict etc), and the expected format against a regex or a whitelist.
Use a library to help you out such as Mongoose (similar to an ORM).
Limit the privileges of your database user to the minimum.

 
###### Security Level

The severity of a SQL injection vulnerability ranges from high to critical depending on the application and context. For example, a SQL injection in your auth provider would be a critical vulnerability because it could lead to compromise of all user data and potentially bypass of access controls.
###### References

- https://www.owasp.org/index.php/SQL_Injection_Prevention_Cheat_Sheet

- https://blog.websecurify.com/2014/08/hacking-nodejs-and-mongodb.html (Example noSQL attacks)

--- 
### Preventing Cross-Site Request Forgery
###### Description

Cross-Site Request Forgery attacks allow an attacker to perform actions in an application in the context of an authenticated user. By abusing the way cookies work, if the application’s authentication is cookie-based (e.g. a session cookie is stored and sent with every request), an attacker can setup a website that makes requests to the application for instance via an iframe, an image or by automatically posting some form data and the browser will send along the cookies of the user.

This attack is done across origins from website www.malicious.com to www.application.unity.com so the attacker won’t be able to read the response of the cross-origin request due to the Same-Origin Policy. However, this isn’t necessary for the attack to succeed if the application only requires a session cookie and don’t validate that the request initiated from the expected domain.

The attack can happen in the background on a malicious website while the user browses funny cat pictures and suspects nothing. Exploitable actions can be anything the application allows, such as changing a password, changing shipping address, deleting a user, or anything else that might be used to an attacker’s benefit.
###### Why We Care

Cross-Site Request Forgery is a well-known attack vector today, but many prominent sites such as Gmail suffered from this 10 years ago. The attack is usually targeted towards a single user account, but if that user account is an administrator of the application the business consequences can be very serious.
###### Example of Issue

Our internal legacy system for doing deploys for the web team used to be vulnerable to Cross-Site Request Forgery. It made it possible for an attacker outside the Unity network to trigger deploys and even gain code execution on the server due to a combination of two flaws. The following HTML page shows how simple this CSRF attack was:

If a logged in user visited this website it would create a file on the Online Deploy server by abusing the privileges of the user and a command injection flaw in the “t” parameter.
###### How to Fix?

There are several strategies to fixing this problem, and it depends a bit on the application’s architecture. Here are the main strategies that are most commonly used and accepted as a standard way of solving this problem:
###### _Authentication via HTTP header_

Some auth APIs use the `“Authentication: Bearer <token goes here>”`  header for authentication. Requests that require an authentication header to succeed are not vulnerable to CSRF attacks.
###### _CSRF Tokens_

This is the most common and accepted protection mechanism. It works by requiring every form submission and request that perform an action to include a random CSRF token in the submitted data. Since this token is set and stored by the server, and tied to a user’s session, an attacker won’t be able to access it or guess it. This prevents CSRF attacks from succeeding as the server should reject any request that don’t have a valid token.

The first thing to make sure is that your application uses the HTTP GET and POST verbs appropriately, see http://guides.rubyonrails.org/security.html#csrf-countermeasures

Next you should look at your web application framework to see if they have support for CSRF tokens. Most frameworks, like Ruby on Rails as mentioned above, will either have built-in support for easily adding CSRF tokens or libraries that help implement it. Here are two popular options for NodeJS and Golang:
-  https://github.com/expressjs/csurf
- https://github.com/utrack/gin-csrf

###### _Checking Origin / Referer headers_

This is another method that can be used to prevent CSRF attacks. This method relies on checking the “Origin” header of a request to make sure it is coming from the origin on which the application is hosted and not a malicious domain. Only the browser is allowed to set the “Origin” header so it isn’t spoofable by a malicious application. It isn’t as bulletproof as using CSRF tokens because browsers don’t have to, but may, set the origin header, as specified in RFC-6454.

It is common to combine this check with checking the “referer” header of the request. The “referer” header is also exclusively set by the browser, and contains the URL or domain that the request originated from. The application can check that this header is set to the correct domain to make sure that the request originated from a user session, and not from a malicious domain.
###### _Samesite Cookies_

This is the third mechanism for protecting against CSRF. It is a new browser feature that allows marking cookies with a flag controlling whether requests initiated from a third-party site will include the cookies.

There are two modes: Strict and lax.

Strict mode prevents the browser from sending the cookies cross-origin for any request. This can have negative consequences in situations where a logged-in user clicks a link leading back to an authenticated page in the application because the cookies won’t be sent along and the user needs to reauthenticate.

Lax mode prevents this situation and allows GET requests to send along the cookies. Requests using any other HTTP verbs won’t send along the cookies. If the application uses HTTP verbs appropriately this would normally be enough to prevent CSRF attacks because the application shouldn’t use a GET request to perform any action in the application.

- Set Lax mode: `Set-Cookie: CookieName=CookieValue; SameSite=Lax;`
- Set Strict mode: `Set-Cookie: CookieName=CookieValue; SameSite=Strict;`

Since this is a very new browser security feature, not all browsers support it yet and not all frameworks have APIs for setting cookies with this flag. Consider implementing it though, as it is simple, unobtrusive and effective way of preventing CSRF attacks and browser support is likely to increase in the future.
Security Level

The example above shows how serious CSRF vulnerabilities can be. The impact of a CSRF vulnerability can range from low to critical depending on the application. CSRF attacks are usually targeted and need to be tailored to a specific application, but there have also been CSRF vulnerabilities in commonly used libraries and frameworks that can be used by an attacker to target a multitude of sites at the same time.
###### References
- https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF)_Prevention_Cheat_Sheet
- https://www.netsparker.com/blog/web-security/same-site-cookie-attribute-prevent-cross-site-request-forgery/

 ---
### Credential Leaks
###### Description

Application development requires integration with a growing number of services. This requires developers to handle a number of secrets: Database credentials, api tokens, oauth secrets etc. These should be high-entropy, unique per service and per environment leading to large number of secrets to manage. These often end up being stored in configuration files which are then accidentally checking into your favorite version control system on a publicly accessible cloud-hosted repositories, and hence into the hands of attackers.
###### Why We Care

Secrets are secret for a reason and often provide access to a lot of data, computing resources or privileged accounts. It is therefore crucial that we handle secrets well, and make the probability of accidental leaks as small as possible.
###### Example of Issue

Credential leaks to Github are a common, and troublesome issue within the industry. This can give malicious actors access to internal systems, data, apis and more. There are also other examples, such as accidentally hosting script files that contain credentials on a public web server.
###### How to Fix?

When developing an application it is best practice to load secrets from environment variables.

This has a few benefits, first, it stores the credentials in the running process’ address space, this means that other user accounts on the machine that aren’t privileged, won’t be able to access the credentials.

Secondly, it separates the secrets from your application normal configuration files which are often checked into version control systems. When developing the application, use a .env file for storing the secrets and load them into your environment before running the application locally.

Finally, it prepares your application for using our secret management system, Vault, which allows you to store secrets in a central, secure place. If you are running your application in Infrastructure Engineerings’ infrastructure you could make use of Vault quite easily. If you want to read more about Vault and how to get started go here.

The application security team have tools monitoring for secrets in source code repositories, but this mechanism is only supposed to be the last line of defense and to avoid credential leaks it is up to the developer to be diligent when handling these sensitive pieces of data.
Security Level

Leaking credentials can be critical as they often grant a high level of access to Unity systems and can lead to compromise of data and systems.
References

https://github.com/UnityTech/Secret-Finder - _(Currently Internal Only - Public Release TBD)_ 
