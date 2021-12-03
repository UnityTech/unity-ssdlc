# Dependency Security [Coding Practice]

## Overview

### Description

This document provides guidelines for how to securely store and retrieve dependencies when building and deploying your software. There are some security practices and considerations to be aware of when retrieving dependencies, both as the developer retrieving a dependency, and as a developer hosting dependencies for others to consume.

NOTE: Dependencies come in many flavors (modules, packages, libraries, etc.), so we will refer to a dependency generically as "package" in this document.#store-private-and-symmetric-keys-in-a-secure-location)

- [Python - Pip](#python---pip)
- [NPM/Yarn](#npmyarn)
- [Golang - Go Modules](#golang---go-modules)
- [Maven](#maven)
- [Nuget](#nuget)
- [Brew](#brew)
- [Artifactory](#artifactory)
- [Jenkins](#jenkins)
- [Gems](#gems)

### General Recommendations

- Keep dependency management software up to date.
- Internally developed and used dependencies should be stored on an internal package repository.
  - The internal package repository should NOT proxy public registries.
- Ensure that your build and deployment scripts are querying the correct package repositories for your internal packages.
- Public repositories should not be queried for internally stored packages.
- Utilize package checksum or hash integrity checking to ensure the correct dependency is being retrieved
- Pin specific dependency versions to be used.
- Follow the dependency management specific recommendations below.

### Vulnerable Technologies

| Technololgy       | Vulnerable? |
| :---------------- | :---------- |
| Python/Pip        | Yes         |
| NPM/Yarn          | Yes         |
| Ruby/Gems         | Yes         |
| Golang/Go Modules | No          |
| Maven             | Yes         |
| Nuget             | Yes         |
| Homebrew          | Yes         |
| Artifactory       | Yes         |
| Jenkins           | Yes         |

## Python - Pip

### Description

The Python package manager *pip* has two command line argument that are used for specifying the URL of the Python Package Index: *--index-url* and *--extra-index-url*. The latter can be dangerous if not fully understood. Like the *--index-url* argument, -*-extra-index-url* argument takes an index URL, but it will also do a lookup and comparison against the package in PyPI and select the package with the highest version number. According to [this research](https://medium.com/@alex.birsan/dependency-confusion-4a5d60fec610), the sequence of events are:

1. Check whether the library exists on the specified (internal) package index
2. Check whether the library exists on the public package index (PyPI)
3. Install whichever version is found. If the package exists on both, it defaults to installing from the source with the higher version number.

If an attacker uploads malicious code to PyPI with the same package name and a higher version number, that malicious code will be loaded and execute in the build.

If the -*-extra-index-url* command line argument is used without control of the package in PyPI, an attacker can gain code execution on the build system, compromising the security of that system and any systems associated with that application.

### How to Fix?

Avoid usage of *--extra-index-url* when using *pip* in your deployment scripts or pipelines. Instead use the --index-url switch, which does not contain this kind of unexpected behavior. Use “pip-compile” to generate locked file with hashes and enable hash checking mode when installing.

### References

- https://medium.com/@alex.birsan/dependency-confusion-4a5d60fec610
- https://azure.microsoft.com/en-us/resources/3-ways-to-mitigate-risk-using-private-package-feeds/

## NPM/Yarn

### Description

Node Package Manager (NPM) by default uses the [https://npmjs.com](https://npmjs.com/) package registry. If a developer does not explicitly define scopes and versions for their dependencies, they may download a package from an unintended registry, such as the public NPM registry. If a malicious attacker uploads malicious code to NPM publicly for a private package with a higher version, the malicious package will be automatically downloaded when the user pulls down updated dependencies.

### How to Fix?

The following mitigations should be in place:

1. Define strict scopes for 

   dependencies

   ​	a. Leverage the \@scope notation in order to define explicit scopes for packages in the project's local .npmrc file

   ​	b. Example:

```
@mycompany:registry = https:``//registry.mycompany.local/
```

1. Only use trusted or private registries, and do not proxy external registries.

2. Use explicit version numbers in the package.json

   ​	a. Example:

```
{
	"name": "@mycompany/foo",
	"version": "1.2.3",
	"description": "just a scoped package name example",
	"dependencies": {
		"@mycompany/bar": "2.0"
	}
}
```

### References

- https://github.blog/2021-02-12-avoiding-npm-substitution-attacks/

## Golang - Go Modules

### Description

Go 1.11 introduced Go Modules as the official dependency management system. Prior to Go Modules, there were a number of 3rd party dependency management options, such as Glide, Govendor, and dep. Since Go 1.11, these have all be deprecated, archived, or are no longer maintained. Due to this, it is best to use Go Modules for dependency management. Go Modules is not vulnerable to dependency confusion.

### How to Fix?

Only use Go Modules for dependency management. Other Go dependency management solutions are no longer maintained

### References

- https://blog.golang.org/using-go-modules
- https://golang.org/ref/mod

## Maven

### Description

Multiple repository URLs can be specified in user or project profiles. These are queried in order, but the order is not obvious from any one configuration file. Dependencies are checked for modifications since the original upload but cannot be automatically verified against prior installs.

### How to Fix?

Configure a single mirror that is `<mirrorOf>*</mirrorOf>` to direct all requests through a single repository. This will ensure your private repository takes priority. Enable upstreams on the private repository. Alternatively, the default public repository can be overridden to disable the setting.

Maven currently has no integrated features for further checks. Your only option is to switch to Gradle for checksum and signature checks.

### References

- Sourced from: https://azure.microsoft.com/en-us/resources/3-ways-to-mitigate-risk-using-private-package-feeds/
- https://maven.apache.org/guides/mini/guide-mirror-settings.html#using-a-single-repository

## Nuget

### Description

Multiple package sources specified by user or project. The latest version found from any of sources checked will be installed. If an attacker uploads malicious code to a public source with the same package name and a higher version number, that malicious code will be loaded and execute in the build.

### How to Fix?

Clear all packageSource settings in project configuration or nuget.config, add only your private gallery, and enable upstreams. Ensure your nuget.config packageSources section starts with a `<clear />` entry to remove any inherited configuration, and use a single `<add />` entry for your private feed.

Prefer the “nuget restore --locked-mode” command and include a generated packages.lock.json with your project. A packages.lock.json file will be automatically created on “nuget restore.” When the file exists and is included with your project, it will be used by “nuget restore --locked-mode” to validate that the packages have not changed using version pinning and integrity checking. 

An ID prefix can be registered by publishers to restrict uploads to the public gallery. Packages under a registered prefix can only be uploaded by approved accounts, which also protects against public substitution attacks. This reservation can be done whether you intend to publish your packages to [NuGet.org](http://nuget.org/) or not. Using a registered ID prefix for private packages helps ensure that an attacker cannot claim any of your names.

### References

- Sourced from: https://azure.microsoft.com/en-us/resources/3-ways-to-mitigate-risk-using-private-package-feeds/
- https://docs.microsoft.com/en-us/nuget/nuget-org/id-prefix-reservation
- https://docs.microsoft.com/en-us/nuget/consume-packages/package-references-in-project-files#locking-dependencies

## Brew

### Description

Homebrew is a package manager used for MacOS to install command line tools, applications built directly from source. Homebrew typically installs/compiles code based on the repository owner and name, when using the `brew install` command. This can lead to an issue if the proper casks have not been tapped, that the user may download the incorrect package.

### How to Fix?

We recommend adding a prefix to all internal tool repositories. This will mitigate the issues when trying to pull new brew packages. Additionally, we recommend using the full formula name, in order to avoid ambiguity.

## Artifactory

### Description

Artifactory has a "Virtual Repositories" feature that essentially works like pip's --extra-index-url flag. When using virtual repositories, Artifactory will search both private and public packages, and then select the one with the highest version number. Documentation explicitly states that Artifactory will grab the newest version, even if a local repo is found first. (Documentation on [Virtual Repositories](https://www.jfrog.com/confluence/display/JFROG/Virtual+Repositories)) Artifactory is currently treating the fix for this as a "feature request" so there is no timeline on granular access controls as a mitigation.

![Example of the repo selection view](https://www.jfrog.com/confluence/download/attachments/89296181/available_virtual_repos.png?version=1&modificationDate=1562244868000&api=v2)

### How to Fix?

Remove all third party repository sources from the Virtual Repository lists, or disable Virtual Repositories entirely.

### References

- https://www.jfrog.com/confluence/display/JFROG/Virtual+Repositories

## Jenkins

### Description

The Jenkins project uses its own Artifactory binary repository, to distribute core, library, and plugin releases. Only artifacts uploaded there can be considered released. Plugins and libraries are not uploaded or mirrored to Maven Central. The Jenkins Artifactory is hosted at [https://repo.jenkins-ci.org](https://repo.jenkins-ci.org/). Anyone has permission to view and download all (public) artifacts. Jenkins will only delete published packages that are considered a “risk”, so it's extra important to carefully check the packages being pulled.

### How to Fix?

Use an explicit package version when pulling from the Jenkins Artifactory. Consider using a controlled source to pull packages from.

### References

- https://www.jenkins.io/doc/developer/publishing/artifact-repository/

## Gems

### Description

Dependency confusion can occur when multiple global sources are listed in Bundler. For example, suppose you need to include both private and public Gems in your project. You would need to include the sources for both public and private Gems. Sources can be declared in numerous ways and Bundler will chose the source based on Source Priority Rules below:

1. The source explicitly attached to the gem (using `:source`, `:path`, or `:git`)
2. For implicit gems (dependencies of explicit gems), any source, git, or path repository declared on the parent. This results in bundler prioritizing the ActiveSupport gem from the Rails git repository over ones from `rubygems.org`
3. The sources specified via global `source` lines, searching each source in your `Gemfile` from last added to first added.

Dependency Confusion can occur if no other source is explicitly declared **and** multiple global sources are specified. There are two primary scenarios where this can occur:

1. Your intended source is listed **before** the unintended global source
2. Your intended source **fails** and Bundler tries the next global source

### How to Fix?

When specifying required gems with Bundler,

- **Do not** specify multiple global sources. Listing multiple global sources can lead to gems being installed from unintended sources. Bundler will throw a warning if a Gem has multiple sources and will tell you *which* source was used as well as alternative sources
- **Do not** use the `:github` shorthand to define public Github sources. This will default to the insecure `git://` protocol which is vulnerable to man-in-the-middle attacks.
- **Do** wrap gems from alternate/private sources blocks. This specifies the proper source for gems from alternate sources. Alternatively, you can specify individual gems inline if only one gem is required from a particular private source.
- **Do** explicitly define sources to routes using `https://`

**Bundler Config For Multiple Sources**

```
# Global Sourced Gems
source 'https://rubygems.org'
 
# Gems here
 
# Source wrapped Gems
source 'https://gems.example.com' do
  # Multiple Gems from the alternative source here
end
 
#Individual Gem from a private source
gem 'testgem', '1.0', :source => 'https://gems.example.com'
 
#Individual Gem from git repository
gem 'testgem', :git => 'https://github.com/company-internal/testgem.git', :branch => 'master'
```

## References

https://mensfeld.pl/2021/02/rubygems-dependency-confusion-attack-side-of-things/

https://bundler.io/gemfile.html

https://bundler.io/man/gemfile.5.html

https://bundler.io/man/gemfile.5.html#SOURCE-PRIORITY

## Contributors

* Carlo Valentin, Cody Ebert, Frankie Arana, Jonathan Downing, Andrew Luke - December 2021