# Ruby Security Guidelines [Language Best Practices]

<font size="-1">_Author: Andrew Luke Dec. 2018_</font>

- [Avoid Combing User Input with Dangerous Calls](#)
- [Use Proper REGEX Start and End of String Symbols](#)
- [Ensure Dynamic Content is Encoded](#)
- [Exploitation of Mass Assignment (Versions 2.3.x and 3.x)](#)
- [Enforce HTTPS and Strict Transport Security](#)
- [Prevent Logging of Sensitive Information](#)
- [Avoid Disabling deep_munge](#)
- [Store the Rails Master Key in an Environmental Variable](#)
- [Don’t Allow User Input Determine Which View to Render](#)
- [Session Fixation](#)



### Overview

This is a set of security guidelines for the Ruby programming language, with emphasis on the Ruby on Rails web application framework. Each element in this document will be Ruby-specific. General web application and API guidelines will be covered here. This guideline is intended to be a living security reference for development teams and quality assurance as part of the Application Security Team’s Secure Software Development Lifecycle (SSDLC). As such, it will be continually added to and updated as the Application Security team is made aware of new security considerations or gaps in the current guidelines. In general, this guide is geared towards the most recent version of Ruby and Ruby on Rails. Any exceptions will be identified as to the versions they apply to.
### Security Considerations
#### Mailing Lists

To be made aware of security issues as they occur, subscribe to the Ruby and Ruby on Rails mailing lists:

- https://groups.google.com/forum/#!forum/ruby-security-ann
- https://groups.google.com/forum/#!forum/rubyonrails-security

#### Cross-Site Request Forgery (CSRF)

By default, Rails includes an unobtrusive scripting adapter (https://github.com/rails/rails/tree/master/actionview/app/assets/javascripts), which adds a header called X-CSRF-Token with the security token on every non-GET Ajax call. Without this header, non-GET Ajax requests won't be accepted by Rails. When using another library to make Ajax calls, it is necessary to add the security token as a default header for Ajax calls in your library. To get the token, have a look at <meta name='csrf-token' content='THE-TOKEN'> tag printed by <%= csrf_meta_tags %> in your application view.
#### Weak Gem Signing

Ruby gems have a signing mechanism in place, but it is not required - or widely used - and lacks a chain of trust for the signing keys. As a result, care should be taken when using third party libraries
### Recommended Libraries

- Terrapin for system commands: https://github.com/thoughtbot/terrapin
- Strong_parameters to prevent mass assignment issues in 3.x: https://github.com/rails/strong_parameters
- Strong_parameters_rails2 to prevent mass assignment issues in 2.3.x: https://github.com/grosser/strong_parameters/tree/rails2

### Resources

- http://guides.rubyonrails.org/security.html
- http://guides.rubyonrails.org/v3.2.8/security.html
- https://www.owasp.org/index.php/Ruby_on_Rails_Cheatsheet

## Recommendations
#### Avoid Combing User Input with Dangerous Calls
###### Description

Ruby on Rails supports a number of dangerous calls that introduce a lot of potential for vulnerabilities.

    - Eval("ruby code here")
    - Instance_eval()
    - Class_eval()
    - Kernel.exec("os command here")
    - System("os command here")
    - Popen
    - open("| os command here")
    - DRb.start_server()
    - `ls -al`  (backticks contain system commands)
    - Find_by_sql()
    - %x()
    - send()
    - __send__()
    - public_send()
    - try()
    - render()

###### Why We Care

Combining these calls with user input could result in server compromise.
###### How to fix?

It is best practice to avoid using unfiltered user input to form a system or Ruby command with one of the dangerous calls above. If user input is needed to inform the command, there are a couple of things you can do to minimize exposure. Comparing the user input to a whitelist of expected values before passing it to the command execution functionality is an effective way to validate the input.

For executing system commands using user input, using a tool like Terrapin is preferable over the dangerous calls above. With Terrapin, user input can be safely shell-escaped. Terrapin will only shell-escape what is passed in as interpolations to the run() method. It WILL NOT escape what is passed in to the second argument of new. Terrapin assumes that you will not be manually passing user-generated data to that argument and will be using it as a template for your command line's structure.
###### Risk Rating

High
###### References

- https://github.com/thoughtbot/terrapin

---
#### Use Proper REGEX Start and End of String Symbols
###### Description

Ruby uses a slightly different approach than many other languages to match the end and the beginning of a string. In Ruby `^` and `$` match the line beginning and line end, not the string beginning and string end. The `\A` and `\z` should be used instead of `^` and `$.`
###### Why We Care

When using REGEX to validate user input, misuse of these symbols can introduce the potential for bypasses that get malicious input past the REGEX validation.
###### Example of Issue

The following REGEX filter is meant to accept only valid URLs
```regex
    /^https?:\/\/[^\n]+$/i
```

Due to the ^ symbol use instead the \A symbol, the following submission would pass the filter because each line is processed separately and the second line matches:

  ```ruby  
 javascript:exploit_code();/*
    http://hi.com

    */
```
 
###### How to fix?

To fix this, the example issue should use the following REGEX:

    /\Ahttps?:\/\/[^\n]+\z/i

 
###### Risk Rating

High
###### References

- http://guides.rubyonrails.org/security.html#regular-expressions

---
#### Ensure Dynamic Content is Encoded
###### Description

By default, Rails 3.0 and up protects against cross-site scripting (XSS) through the output encoding of data placed into views. However, there are several ways in Ruby on Rails for developers to bypass the default XSS protections. These are the dangerous template tags which bypass default output encoding:
```ruby
    <%= raw @product.name %>

    <%= @product.name.html_safe %>

    <%= content_tag @product.name %>

    <%= link_to “Personal Website”, @user.website %>

        Here, the @user.website value could begin with javascript: tag, executing any JavaScript payload that follows it.
```
###### Why We Care

XSS can be used to perform any action the authenticated user has access, including other services if authentication is handled through the same auth provider.
###### How to fix?

These methods of printing raw HTML to the page should only be used for trusting, non-user generated content that doesn’t include user input. If users are able to influence the data used with these bypasses, it becomes vulnerable to XSS.
###### Risk Rating

High
###### References

- https://www.owasp.org/index.php/Ruby_on_Rails_Cheatsheet#Cross-site_Scripting_.28XSS.29

---
#### Exploitation of Mass Assignment (Versions 2.3.x and 3.x)
###### Description

Although the major issue with Mass Assignment has been fixed by default in base Rails specifically when generating new projects, it still applies to older and upgraded projects so it is important to understand the issue and to ensure that only attributes that are intended to be modifiable are exposed.

The mass-assignment feature may become a problem in older versions, as it allows an attacker to set any model’s attributes by manipulating the hash passed to a model’s new() method. This vulnerability is extended even further with the introduction of nested mass assignment (and nested object forms) in Rails 2.3.

To prevent this, Strong Parameters should be used globally to enforce whitelisting of attributes in all models. Strong Parameters requires that you specify the required and permitted parameters each time parameters are retrieved. In modern Rails, Strong Parameters is enabled by default. In older versions, it is possible to use the Strong_Parameters gem with Rails 3.x, and the strong_parameters_rails2 gem for Rails 2.3.x applications. This means you'll have to make a conscious choice about which attributes to allow for mass updating and thus prevent accidentally exposing that which shouldn't be exposed.

An alternative for older versions of Rails is to blacklist specific model attributes using the attr_protected method or whitelist specific model attributes using the attr_accessible method when defining your models. If you are working in Rails < 3.2.3 you must enable attribute whitelisting with the following:

 
```ruby
config.active_record.whitelist_attributes = true
```


Whitelist example that allows access by role:

    
```ruby
    attr_accessible :name

    attr_accessible :name, :is_admin, :as => :admin
```
 
###### Why We Care

Interactions with datastores are typically one of the most sensitive areas in a web application, from a security perspective. Ensuring that users cannot exploit those interactions to escalate privileges or gain unauthorized access to data is important to overall application security.
###### Example of Issue

The following function mass assigns user input when creating a new user.

 
```ruby 
    def signup
        params[:user]
        @user = User.new(params[:user])
    end
```
 


If you create a new user using mass-assignment, and rely on the user record to determine administrative access, it may be possible to become an administrator:

 

http://www.example.com/user/signup?user[name]=ow3ned&user[admin]=1

 


In this case, the created use would have it’s admin field set to 1 due to mass assignment.
###### How to fix?

Do not disable strong parameters in modern versions of Rails. In older versions, use the Strong_Parameters gem (3.x) or the strong_parameters_rails2 gem (2.3.x).
###### Risk Rating

Medium
###### References

- https://github.com/rails/strong_parameters
- https://github.com/grosser/strong_parameters/tree/rails2
- http://guides.rubyonrails.org/v3.2.8/security.html#mass-assignment

---
#### Enforce HTTPS and Strict Transport Security
###### Description

AppSec recommends that all Unity web applications enforce transport encryption (HTTPS) and utilize Strict Transport Security to protect session cookies, when they are used as session management. Strict Transport Security tells the browser to only access the web application over an encrypted channel.
###### Why We Care

Strong encryption of traffic between the user and the web server plays a key part in protecting the user’s authenticated session and the data exchanged with the server.
###### How to fix?

In Rails 3.1 and later, this can be accomplished by always forcing SSL connections in your application config file:

 
```ruby
    config.force_ssl = true
```

###### Risk Rating

Medium

---  
#### Prevent Logging of Sensitive Information
###### Description

By default, Rails logs all requests being made to the web application. But log files can be a huge security issue, as they may contain login credentials, credit card numbers et cetera.
###### Why We Care

Logging of sensitive, user information puts that data at higher risk of compromise. It is also a GDPR issue.
###### How to fix?

When designing a Rail application, make sure to designate request parameters that should not be logged. These should be appended to the list of filtered parameters in the Rails configuration:

 
```ruby
    config.filter_parameters << :cc_number
```
 
###### Risk Rating

Medium
###### References

- http://guides.rubyonrails.org/security.html#logging

--- 
#### Avoid Disabling deep_munge
###### Description

Due to the way Active Record interprets parameters in combination with the way that Rack parses query parameters it was possible to issue unexpected database queries with IS NULL where clauses. As a response to that security issue, the deep_munge() method was introduced as a solution to keep Rails secure by default. AppSec recommends avoiding disabling deep_munge().
###### Why We Care

If deep_munge() is disabled, the vulnerabilities above will be reintroduced into the application, exposing it to risk of SQL injection attacks.
###### How to fix?

If it must be disabled, be fully aware of the vulnerabilities you are introducing, and how to properly avoid them. Otherwise, avoid disabling deep_munge().
###### Risk Rating

Medium
###### References

- http://guides.rubyonrails.org/security.html#unsafe-query-generation

--- 
#### Store the Rails Master Key in an Environmental Variable
###### Description

Rails will store a master key that's generated into a version control ignored config/master.key file. However, Rails will also look for that key in ENV["RAILS_MASTER_KEY"].
###### Why We Care

Part of protecting secrets is keeping them out of the code base. AppSec recommends utilizing Infrastructure Engineering’s Vault instance for secrets management. Alternatively, storing and retrieving secrets from environmental variables is a better alternative.
###### How to fix?

As with all secrets, AppSec recommends removing the key from the config/master.key file and placing it in a RAILS_MASTER_KEY environmental variable in production.
###### Risk Rating

Low
###### References

- http://guides.rubyonrails.org/security.html#custom-credentials

---
#### Don’t Allow User Input Determine Which View to Render
###### Description

Rails’ render() method allows for dynamic view rendering by supplying the template name to render.
###### Why We Care

If the Rails application contains views that are restricted to certain user roles, this could allow an attacker to circumvent those access controls and access restricted views as lower privilege users.
###### How to fix?

Avoiding using user input to choose which view to render when certain views are restricted.
###### Risk Rating

Low
 
 ---
#### Session Fixation
###### Description

Session fixation is the term for when a pre-authentication session token used as the post-authentication session token. In other words, the a new session token is not set after authenticating. This is not an issue if header-based authentication is in use.
###### Why We Care

An attacker with knowledge of the current session token set in the browser could hijack the user’s authenticated session once they log in.
###### How to fix?

Use the reset_session() function after a user successfully authenticates to set a new session cookie in their browser.
###### Risk Rating

Low
###### References

- http://guides.rubyonrails.org/security.html#session-fixation-countermeasures