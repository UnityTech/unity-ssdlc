# .Net/C# Best Practices [Language Best Practices]
<font size="-1">_Author: Brandon Caldwell - Jan. 2019_</font>

- [Untrusted Code and Resources](#untrusted-code-and-resources)
- [Handling User Input](#handling-user-input)

 # Overview
Microsoft's .Net (DotNet) framework, like most modern frameworks, is generally secure from memory corruptions and code control (ex., stack corruption) style attacks, as long as the framework is kept up-to-date with the latest version from Microsoft. However, over the years, there have been some features of the framework that are now considered unsafe, or do not provide the protections developers believe they do. Below is general best practices on what to watch out for when developing in .Net languages.

- [Untrusted Code and Resource](#untrusted-code-and-resources)
- [Handling User Input](#handling-user-input)

### Recommendations
#### Untrusted Code and Resources

The following best practices are in regards to loading code or resources from an untrusted source, i.e., anything from the internet or not signed by Unity (or trusted 3rd party). When working with these types of resources, refer to the below:

- Do not use partial trusted code, Code Access Security (CAS), or AllowPartiallyTrustedCaller.
  - Problem: CAS was billed as a way to load untrusted libraries/code at runtime, by assigning 'trust levels' to various portions of code. Microsoft now recommends against using CAS as a means to protect against malicious code. Partial trust code and AllowPartiallyTrustedCaller has similar issues, wherein is does not provide any effective protections.

  - Solution: Only load code and resources that have been signed by a trusted party.
- Do not use .NET Remoting or binary formatters (for object serialization/deserialization)
  - Problem: Hints that serialization vulnerabilities in the .Net framework have existed since 2012 (MS12-035 via James Forshaw), later revelations found major vulnerabilities within serialization, beyond what was published in that article.
  - Solution: Use DataContractSerialiazer as part of serializing objects.
- Do not use Distributed Component Object Model (DCOM).
  - Problem: Again, thanks to James Forshaw, it was determined that DCOM was generally unsafe, and has issues with safe (or unsafe?) object serialization.
  - Solution: Don't use it at all. Consider named pipes for local services (available cross-platform). Avoid HTTP/REST-like endpoints if possible, as they can be difficult to protect against browser attacks - see [CVE-2017-12939](https://unity3d.com/security#CVE-2017-12939).

In short, to quote the existing [Microsoft documentation - Secure Coding Guidelines](https://docs.microsoft.com/en-us/dotnet/standard/security/secure-coding-guidelines):

>Code Access Security and Security-Transparent Code are not supported as a security boundary with partially trusted code. We advise against loading and executing code of unknown origins without putting alternative security measures in place.

---
#### Handling User Input

The following best practices is largely relevant to ASP.Net and .Net Embedded browser support:

- Any user data in a server response runs in the context of the server's site on the client. If your Web server takes user data and inserts it into the returned Web page, it might, for example, include a `<script>` tag and run as if from the server.
- Remember that the client can request any URL.
- Consider tricky or invalid paths:
  - ..\ , extremely long paths.
  - Use of wild card characters (*).
  - Token expansion (%token%).
  - Strange forms of paths with special meaning.
  - Alternate file system stream names such as filename::$DATA.
  - Short versions of file names such as longfi~1 for longfilename.
- Remember that Eval(userdata) can do anything.
- Be wary of late binding to a name that includes some user data.
- If you are dealing with Web data, consider the various forms of escapes that are permissible, including:
  - Hexadecimal escapes (%nn).
  - Unicode escapes (%nnn).
  - Overlong UTF-8 escapes (%nn%nn).
  - Double escapes (%nn becomes %mmnn, where %mm is the escape for '%').
- Be wary of user names that might have more than one canonical format. For example, you can often use either the `MYDOMAIN\username` form or the `username@mydomain.example.com` form.
  - (From https://github.com/dotnet/docs/blob/master/docs/standard/security/security-and-user-input.md)

 ---
###### References:

- https://github.com/dotnet/docs/blob/master/docs/standard/security/secure-coding-guidelines.md
- https://docs.microsoft.com/en-us/dotnet/framework/misc/code-access-security
- https://docs.microsoft.com/en-us/dotnet/framework/misc/security-transparent-code
- https://support.microsoft.com/en-us/help/2698981/asp-net-partial-trust-does-not-guarantee-application-isolation
- Attacking .Net Serialization: https://speakerdeck.com/pwntester/attacking-net-serialization?slide=8
- .Net Remoting Security Bulletin: - https://docs.microsoft.com/en-us/security-updates/SecurityBulletins/2012/ms12-035
