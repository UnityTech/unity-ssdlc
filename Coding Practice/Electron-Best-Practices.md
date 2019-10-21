# Electron Best Practices [Language Best Practices]
<font size="-1">_Author: Andrew Luke - Dec. 2018_</font>

### Overview 

This is a set of security guidelines for developing Electron applications. Each element in this document will be Electron-specific. General web application and API guidelines will be covered here. This guideline is intended to be a living security reference for development teams and quality assurance as part of the Application Security Team’s Secure Software Development Life Cycle (SSDLC). As such, it will be continually added to and updated as the Application Security team is made aware of new security considerations or gaps in the current guidelines.

Electron was created as a way to develop desktop applications with locally residing content, including HTML and Javascript. Its security model is based around this. As a result, security issues arise when you begin loading remote, untrusted content into your Electron application.

Default Electron functionality exposes potentially dangerous functionality to renderers, which makes sense when they renderers are loading local files. For example, Chromium’s sandboxing has been removed. If untrusted content contains vulnerabilities, such as Cross-Site Scripting, is loaded into a renderer, it can result in code execution on the system. Most of the recommendations in this guideline will be focused on properly configuring and using Electron with untrusted content.   
#### Security Considerations 

By default:
- Chromium’s sandboxing is disabled.
- Preload scripts run on the same JavaScript runtime as the renderer.
- NodeJS APIs are accessible within renderers.

#### Resources

###### Electron Security Page
- https://electronjs.org/docs/tutorial/security

###### Security Checklist
- https://www.blackhat.com/docs/us-17/thursday/us-17-Carettoni-Electronegativity-A-Study-Of-Electron-Security.pdf
- https://www.blackhat.com/docs/us-17/thursday/us-17-Carettoni-Electronegativity-A-Study-Of-Electron-Security-wp.pdf

#### Security Checklist

1. Disable nodeIntegration for untrusted origins
2. Use sandbox for untrusted origins
3. Review the use of command line arguments
4. Review the use of preload scripts
5. Do not use disablewebsecurity
6. Do not allow insecure HTTP connections
7. Do not use Chromium’s experimental features
8. Limit navigation flows to untrusted origins
9. Use setPermissionRequestHandler for untrusted origins
10. Do not use insertCSS, executeJavaScript or eval with user-supplied content
11. Do not allow popups in webview
12. Review the use of custom protocol handlers
13. Review the use of openExternal

---
#### Recommendations
- [Disable NodeJS Integration for Remote Content](#disable-nodejs-integration-for-remote-content)
- [Enable Context Isolation for Remote Content](#enable-context-isolation-for-remote-content)
- [Handle Session Permission Requests From Remote Content](#handle-session-permission-requests-from-remote-content)
- [Do Not Disable WebSecurity](#do-not-disable-websecurity)
- [Define a Content Security Policy](#define-a-content-security-policy)
- [Override and Disable eval() If Not Needed](#override-and-disable-eval-if-not-needed)
- [Do Not Set allowRunningInsecureContent to true](#do-not-set-allowrunninginsecurecontent-to-true)
- [Do Not Use allowpopups](#do-not-use-allowpopups)
- [Verify WebView Options Before Creation](#verify-webview-options-before-creation)
- [Disable or Limit Navigation](#disable-or-limit-navigation)
- [Disable or Limit Creation of New Windows](#disable-or-limit-creation-of-new-windows)


##### Disable NodeJS Integration for Remote Content
###### Description

It is paramount that you disable NodeJS integration in any renderer (BrowserWindow, BrowserView, or <webview>) that loads remote content. The goal is to limit the powers you grant to remote content, thus making it dramatically more difficult for an attacker to harm your users should they gain the ability to execute JavaScript on your website.

After this, you can grant additional permissions for specific hosts. For example, if you are opening a BrowserWindow pointed at `my-website.com/", you can give that website exactly the abilities it needs, but no more.
###### Why We Care

A cross-site-scripting (XSS) attack is more dangerous if an attacker can jump out of the renderer process and execute code on the user's computer. Cross-site-scripting attacks are fairly common - and while an issue, their power is usually limited to messing with the website that they are executed on. Disabling Node.js integration helps prevent an XSS from being escalated into a so-called "Remote Code Execution" (RCE) attack.
###### How to Fix?

    // Bad
    const mainWindow = new BrowserWindow()
    mainWindow.loadURL('https://my-website.com')

    // Good
    const mainWindow = new BrowserWindow({
        webPreferences: {
        nodeIntegration: false,
        preload: './preload.js'
        }
    })
    mainWindow.loadURL('https://my-website.com')


    <!-- Bad -->
    <webview nodeIntegration src="page.html"></webview>  

    <!-- Good -->
    <webview src="page.html"></webview>

 

When disabling Node.js integration, you can still expose APIs to your website that do consume Node.js modules or features. Preload scripts continue to have access to require and other Node.js features, allowing developers to expose a custom API to remotely loaded content.

In the following example preload script, the later loaded website will have access to a window.readConfig() method, but no Node.js features.
 

    const { readFileSync } = require('fs')  
    window.readConfig = function () {
        const data = readFileSync('./config.json')
        return data
    }

 
###### Risk Rating

High
 
 ---
##### Enable Context Isolation for Remote Content
###### Description

Context isolation is an Electron feature that allows developers to run code in preload scripts and in Electron APIs in a dedicated JavaScript context. In practice, that means that global objects like Array.prototype.push or JSON.parse cannot be modified by scripts running in the renderer process.

Electron uses the same technology as Chromium's Content Scripts to enable this behavior.
###### Why We Care

Context isolation allows each the scripts on running in the renderer to make changes to its JavaScript environment without worrying about conflicting with the scripts in the Electron API or the preload script.

While still an experimental Electron feature, context isolation adds an additional layer of security. It creates a new JavaScript world for Electron APIs and preload scripts.

At the same time, preload scripts still have access to the document and window objects. In other words, you're getting a decent return on a likely very small investment.
###### How to Fix?

    // Main process
    const mainWindow = new BrowserWindow({
        webPreferences: {
        contextIsolation: true,
        preload: 'preload.js'
        }
    })

 
###### Risk Rating

Medium
 
 ---
##### Handle Session Permission Requests From Remote Content
###### Description

You may have seen permission requests while using Chrome: They pop up whenever the website attempts to use a feature that the user has to manually approve ( like notifications).

The API is based on the Chromium permissions API and implements the same types of permissions.
###### Why We Care

By default, Electron will automatically approve all permission requests unless the developer has manually configured a custom handler. While a solid default, security-conscious developers might want to assume the very opposite.
###### How to Fix?

    const { session } = require('electron')

    session
    .fromPartition('some-partition')
    .setPermissionRequestHandler((webContents, permission, callback) => {
        const url = webContents.getURL()

        if (permission === 'notifications') {
        // Approves the permissions request
        callback(true)
        }

        if (!url.startsWith('https://my-website.com')) {
        // Denies the permissions request
        return callback(false)
        }
    })

 
###### Risk Rating

Medium
 
 ---
##### Do Not Disable WebSecurity
###### Description

You may have already guessed that disabling the webSecurity property on a renderer process (BrowserWindow, BrowserView, or <webview>) disables crucial security features.

Do not disable webSecurity in production applications
###### Why We Care

Disabling webSecurity will disable the same-origin policy and set allowRunningInsecureContent property to true. In other words, it allows the execution of insecure code from different domains.
###### How to Fix?

Electron defaults to webSecurity being enabled. We recommend leaving it enabled.
###### Risk Rating

High
 
 ---
##### Define a Content Security Policy
###### Description

A Content Security Policy (CSP) is an additional layer of protection against cross-site-scripting attacks and data injection attacks. We recommend that they be enabled by any website you load inside Electron.
###### Why We Care

CSP allows the server serving content to restrict and control the resources Electron can load for that given web page. https://your-page.com should be allowed to load scripts from the origins you defined while scripts from https://evil.attacker.com should not be allowed to run. Defining a CSP is an easy way to improve your application's security.

The following CSP will allow Electron to execute scripts from the current website and from apis.mydomain.com.


 

Content-Security-Policy: script-src 'self' https://apis.mydomain.com

 
###### How to Fix?

There are two options here. The first is to add a Content-Security-Policy meta tag to the HTML page being loaded:

 

<meta http-equiv="Content-Security-Policy" content="default-src 'none'">

 

The second option is to add the Content-Security-Policy header to any server HTTP response prior to processing:

 

const {session} = require('electron')  
  session.defaultSession.webRequest.onHeadersReceived((details, callback) => {
    callback({responseHeaders: `default-src 'none'`})
  })

 
###### Risk Rating

Medium
 
 ---
##### Override and Disable eval() If Not Needed
###### Description

eval() is a core JavaScript method that allows the execution of JavaScript from a string. Disabling it disables your app's ability to evaluate JavaScript that is not known in advance.
Why We Care

The eval() method has precisely one mission: To evaluate a series of characters as JavaScript and execute it. It is a required method whenever you need to evaluate code that is not known ahead of time. While legitimate use cases exist, like any other code generators, eval() is difficult to harden.

Generally speaking, it is easier to completely disable eval() than to make it bulletproof. Thus, if you do not need it, it is a good idea to disable it.
###### How to Fix?

 

// ESLint will warn about any use of eval(), even this one
// eslint-disable-next-line
window.eval = global.eval = function () {
  throw new Error(`Sorry, this app does not support window.eval().`)
}

 
###### Risk Rating

Low
 
 ---
##### Do Not Set allowRunningInsecureContent to true
###### Description

By default, Electron will not allow websites loaded over HTTPS to load and execute scripts, CSS, or plugins from insecure sources (HTTP). Setting the property allowRunningInsecureContent to true disables that protection.

Loading the initial HTML of a website over HTTPS and attempting to load subsequent resources via HTTP is also known as "mixed content".
###### Why We Care

Loading content over HTTPS assures the authenticity and integrity of the loaded resources while encrypting the traffic itself.
###### How to Fix?

 

// Bad
  const mainWindow = new BrowserWindow({
    webPreferences: {
      allowRunningInsecureContent: true
    }
  })

 
###### Risk Rating

Medium
 
 ---
##### Do Not Use allowpopups
###### Description

If you are using <webview>, you might need the pages and scripts loaded in your <webview> tag to open new windows. The allowpopups attribute enables them to create new BrowserWindows using the window.open() method. <webview> tags are otherwise not allowed to create new windows.
Why We Care

If you do not need popups, you are better off not allowing the creation of new BrowserWindows by default. This follows the principle of minimally required access: Don't let a website create new popups unless you know it needs that feature.
###### How to Fix?

 

<!-- Bad -->
  <webview allowpopups src="page.html"></webview>

<!-- Good -->
  <webview src="page.html"></webview>

 
###### Risk Rating

Medium

---
##### Verify WebView Options Before Creation
###### Description

A WebView created in a renderer process that does not have Node.js integration enabled will not be able to enable integration itself. However, a WebView will always create an independent renderer process with its own webPreferences.

It is a good idea to control the creation of new <webview> tags from the main process and to verify that their webPreferences do not disable security features.
###### Why We Care

Since <webview> live in the DOM, they can be created by a script running on your website even if Node.js integration is otherwise disabled.

Electron enables developers to disable various security features that control a renderer process. In most cases, developers do not need to disable any of those features - and you should therefore not allow different configurations for newly created <webview> tags.
###### How to Fix?

Before a <webview> tag is attached, Electron will fire the will-attach-webview event on the hosting webContents. Use the event to prevent the creation of webViews with possibly insecure options.

 

app.on('web-contents-created', (event, contents) => {
    contents.on('will-attach-webview', (event, webPreferences, params) => {
      // Strip away preload scripts if unused or verify their location is legitimate
      delete webPreferences.preload
      delete webPreferences.preloadURL
  
      // Disable Node.js integration
      webPreferences.nodeIntegration = false
  
      // Verify URL being loaded
      if (!params.src.startsWith('https://yourapp.com/')) {
        event.preventDefault()
      }
    })
  })

 

Again, this list merely minimizes the risk, it does not remove it. If your goal is to display a website, a browser will be a more secure option.
###### Risk Rating

Medium

---

##### Disable or Limit Navigation
###### Description

If your app has no need to navigate or only needs to navigate to known pages, it is a good idea to limit navigation outright to that known scope, disallowing any other kinds of navigation.
###### Why We Care

Navigation is a common attack vector. If an attacker can convince your app to navigate away from its current page, they can possibly force your app to open web sites on the Internet. Even if your webContents are configured to be more secure (like having nodeIntegration disabled or contextIsolation enabled), getting your app to open a random web site will make the work of exploiting your app a lot easier.

A common attack pattern is that the attacker convinces your app's users to interact with the app in such a way that it navigates to one of the attacker's pages. This is usually done via links, plugins, or other user-generated content.
###### How to Fix?

If your app has no need for navigation, you can call event.preventDefault() in a will-navigate handler. If you know which pages your app might navigate to, check the URL in the event handler and only let navigation occur if it matches the URLs you're expecting.

We recommend that you use Node's parser for URLs. Simple string comparisons can sometimes be fooled - a startsWith('https://google.com') test would let https://google.com.attacker.com through.

 

const URL = require('url').URL  

  app.on('web-contents-created', (event, contents) => {
    contents.on('will-navigate', (event, navigationUrl) => {
      const parsedUrl = new URL(navigationUrl)

      if (parsedUrl.origin !== 'https://my-own-server.com') {
        event.preventDefault()
      }
    })
  })

 
###### Risk Rating

Medium

---
##### Disable or Limit Creation of New Windows
###### Description

If you have a known set of windows, it's a good idea to limit the creation of additional windows in your app.
###### Why We Care

Much like navigation, the creation of new webContents is a common attack vector. Attackers attempt to convince your app to create new windows, frames, or other renderer processes with more privileges than they had before; or with pages opened that they couldn't open before.

If you have no need to create windows in addition to the ones you know you'll need to create, disabling the creation buys you a little bit of extra security at no cost. This is commonly the case for apps that open one BrowserWindow and do not need to open an arbitrary number of additional windows at runtime.
###### Risk Rating
High