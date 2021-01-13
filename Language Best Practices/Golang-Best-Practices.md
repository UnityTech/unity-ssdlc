# Golang Security [Language Best Practices]
<font size="-1">_Author: Andrew Luke - Dec. 2018_</font>

## Overview


This is a set of security guidelines for the Golang programming language. Each element in this document will be Golang-specific. General web application and API guidelines will be covered here. This guideline is intended to be a living security reference for development teams and quality assurance as part of the Application Security Team’s Secure Software Development Lifecycle (SSDLC). As such, it will be continually added to and updated as the Application Security team is made aware of new security considerations or gaps in the current guidelines.

Golang is a strongly typed language that has taken steps to prevent security issues found in other languages. For example, there is no pointer arithmetic. Garbage collection is automatic. Another example, the standard crypto package for AES does not provide Cipher Block Chaining (CBC) as an option.


##### Recommended Packages


- Bcrypt for password hashing: https://godoc.org/golang.org/x/crypto/bcrypt
- AES for symmetric encryption:
  -    https://golang.org/pkg/crypto/aes/
  -    https://golang.org/src/crypto/cipher/example_test.go
- CSRF Protection: https://github.com/gorilla/csrf
- Validator for input validation: https://github.com/go-playground/validator
- Output Sanitization:
  -    https://golang.org/pkg/html/template/
  -    https://github.com/kennygrant/sanitize
  -    https://github.com/microcosm-cc/bluemonday
- Pseudo-random number generation: https://golang.org/pkg/crypto/rand/
- Safe SQL Queries:
  -    https://golang.org/pkg/database/sql/#DB.Prepare
  -    https://github.com/jinzhu/gorm

##### Resources
- Checkmarx Web Application Secure Coding Practices: https://checkmarx.gitbooks.io/go-scp/

### Recommendations
- [Output Encoding with Native Templates](#output-encoding-with-native-templates)
- [Beware Output Encoding Bypasses](#beware-output-encoding-bypasses)
- [Go Security Protections Don’t Extend To C Code](#go-security-protections-dont-extend-to-c-code)
- [Use crypto/rand Instead Of math/rand](#use-cryptorand-instead-of-mathrand)

---
#### Output Encoding with Native Templates
###### Description

The net/http Package does not provide any type of output encoding. The html/template package implements data-driven templates for generating HTML output safe against code injection. It provides the same interface as package text/template and should be used instead of text/template whenever the output is HTML. This package wraps package text/template so you can share its template API to parse and execute HTML templates safely. This package understands HTML, CSS, JavaScript, and URIs. It adds sanitizing functions to each simple action pipeline.
###### Why We Care

Using the net/http package exposes web application users to cross-site scripting attacks due to its lack of output encoding.
###### How to Fix?

Use the html/template package to automatically escape HTML, CSS, JavaScript, and URIs. If the net/http package is used, use manual escaping and manually set the Content-Type, because it is set automatically depending on what is served.
###### Risk Rating
High

###### References

- https://golang.org/pkg/html/template/
 
---
#### Beware Output Encoding Bypasses
###### Description

The html/template package provides a way to circumvent the escaping pipeline, using template. `[HTML|JS|URL]()`method set a string to its type. 
###### Why We Care

Placing user into into this declaration function will bypass default output encoding, introduce cross-site scripting and HTML injection vulnerabilities into your web application.
_Example of Issue_:

The following will declare the `<b>World</b>` string as HTML, printing it as-is into the HTML:

```golang
tmpl.Execute(out, template.HTML(`<b>World</b>`))
```
 
###### How to Fix?

The EscapeString() function in the native html package accepts a string and returns the same string with the special characters escaped. (i.e. `<` becomes `&lt;`). The html/template package has a stripTags() function, but it's unexported.

The html/template package provides several functions for escaping strings:

```golang
    HTMLEscape()
    HTMLEscapeString()
    HTMLEscaper()
    JSEscape()
    JSEscapeString()
    JSEscaper()
    URLQueryEscaper
```
Avoid using template Typed Strings on user input.
###### Risk Rating

High
###### References

- https://golang.org/pkg/html/template/
- https://github.com/kennygrant/sanitize
- https://github.com/microcosm-cc/bluemonday

---
#### Go Security Protections Don’t Extend To C Code
###### Description

The cmd/cgo package provides a way to call C code within a Go application. Go was created with many security protections in place, like memory safety and strong typing. These protections do not extend to the C code called with this package.
###### Why We Care

By using cmd/cgo, you introduce all of the C code security considerations and concerns into your Go application.
###### How to Fix?

Be aware that C code security concerns are still present when ran by a Go application. Understand the inherent security issues with various C code functionality used, and develop the C code ran by your Go application in a secure manner.
###### Risk Rating

High
###### References

- https://golang.org/cmd/cgo/ 

---
#### Use crypto/rand Instead Of math/rand
###### Description

The math/rand package is a deterministic, pseudo-random number generator. It takes a seed and generates pseudo-random numbers with it. Because it is deterministic, these values will be the same each time the application is ran, allowing an attacker to predict the values.
###### Why We Care

Anyone with knowledge of the math/rand seed can determine the pseudo-random values used by the application.
###### How to Fix?

The crypto/rand package should be used instead of the math/rand package. This package uses operating system randomness to generate pseudo-random numbers. Due to this, the generated values are not deterministic and a seed is not needed.

_Example:_

This example reads 10 cryptographically secure pseudorandom numbers from rand.Reader and writes them to a byte slice:
```golang
    package main

    import (
    "bytes"
    "crypto/rand"
    "fmt"
    )

    func main() {
        c := 10
        b := make([]byte, c)
        _, err := rand.Read(b)
        if err != nil {
        fmt.Println("error:", err)
        return
    }

    // The slice should now contain random bytes instead of only zeroes.
    fmt.Println(bytes.Equal(b, make([]byte, c)))
    }
```
 
###### Risk Rating

High
###### References

- https://golang.org/pkg/crypto/rand/
- https://golang.org/pkg/crypto/rand/#example_Read