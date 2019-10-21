# Implementation Review Process [Security Process]
<font size="-1">_Author: Carlo Valentin - Dec. 2018_</font>

## Overview
An implementation review is a code-assisted penetration test, performed to discover any security bugs in the implementation of a new product or feature. This review is a key component of a secure development lifecycle, and this is where we will find most of the issues for a product.

## Requesting an Implementation Review
An implementation review should ideally be requested once a release has been implemented. This would be a feature-complete release, typically when an application has been deployed to a staging environment. When requesting an implementation review, please contact Application Security (AppSec) to schedule a meeting and review scope. Contact AppSec by putting a message in Slack at #support-security.

## Requirements
When requesting a security review, there are a few requirements that need to be met in order to perform an effective security review. The following items should be gathered and in place before the initial meeting takes place.

#### Initial Scope

The development team should provide an initial scope what needs to be reviewed. A final scope will be agreed upon by both the development team and AppSec when the review is being scheduled.

 

#### Designated Point of Contact

The development team should have a designated point of contact to field questions and assist AppSec in removing any blockers to testing or reviewing the application.

 

#### Code Repository Access

AppSec will need access to the repository that contains the code for the application that we will be testing. If the application has multiple repositories, all relevant repositories should be provided to the AppSec engineer performing the review.

 

#### Testable Environment

A testable environment is necessary to get a full implementation review of the new application or feature that is being developed. With a working environment, AppSec can perform dynamic, targeted testing based on the review of the code. The testing environment should be as close to production as possible.

 

The best case scenario is if the development team can provide the repository and all necessary dependencies in order to build the application locally. Alternatively, an application that is deployed and reachable in a staging environment is acceptable. If the service is already live and deployed to production, then production can be used. However, there will be limitations on testing a production service, depending on how critical to the business the application or service is.

 

If there are multiple user roles within the application, credentials should be provided so that the application can be thoroughly tested from multiple aspects and for authorization issues.




#### Documentation

Any documentation related to the application that is needed to run or use the application should be provided. This includes things like design flow diagrams, roadmaps, and any specifications that were generated during development (e.g. API specifications, data specifications).

 

#### Design Review Artifacts

Any artifacts, flowcharts, and/or threat models that were performed by AppSec previously should be provided. If there have been any significant changes to the design that may not match up with the threat model, those details should be provided. If a design review did not occur, a preliminary design review and threat model will occur during the initial meeting.

## Methodology
AppSec will review applications by performing a code review and performing manual testing to validate potential issues found in the code. Additionally, a number of automated tools will be run on the code, and any identified issues will be manually verified.

### Timeframe
Typically an implementation review will occur over the span of one to two weeks. However ,level of effort for a review will depend on the size and risk of an application. When scheduling an implementation review an agreed upon amount of time will be decided between AppSec and the development team. If the engineer performing the design review runs into any issues, the development team will be notified immediately, and further steps will be decided.

## Things We Care About
An application will be compared against any requirements set forth by the Audit and Compliance team. Any additional security requirements that were determined in the design review phase will also be validated during the test. The code will be reviewed against the [language-specific guidelines provided by AppSec](../Language%20Best%20Practices). A set of common vulnerabilities that AppSec maintains can also be used to check an application against. AppSec will also ensure the application follows [provided design principles](../Coding%20Practice/Secure-Design-Principles.md).

## Reporting
At the end of an implementation review, AppSec will schedule a meeting to discuss the findings and answer any questions the development team has. Any critical issues or high issues that need immediate attention will be reported to the designated point of contact immediately. All findings will be held to the escalation process as outlined in our [[Security Process] Risk Rating (aka., Security Bug Bar)](./Risk-Rating.md).