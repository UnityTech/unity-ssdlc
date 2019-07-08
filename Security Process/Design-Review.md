# Design Review and Threat Modelling Process [Security Process]
<font size="-1">_Author: Carlo Valentin - Dec. 2018_</font>

### Overview

Design reviews and threat modelling are key components of ensuring a secure development lifecycle. A design review allows the security team to review an application’s specification and design against security best practices and design patterns. Coupled with a threat model, the development team will be able to determine the need or design of security controls needed for an application. By involving AppSec early on in the process, security will be a consideration throughout the development lifecycle - reducing the amount of security issues that will occur during the implementation and testing phases.
#### Requesting a Design Review

A design review can occur at any point in the development life cycle depending on the situation. Ideally, a design review is performed before any code has been written for an application or service, occurring during the design phase of software development. This is the best time to make adjustments to the application design. Once an application has a design specification and the development team has gathered the appropriate requirements for an application is when the design review should occur. Design reviews can also be performed prior to a security review if AppSec is not familiar with the application. When requesting an design review, please contact Application Security (AppSec) to schedule a meeting and review scope.
#### Requirements

When requesting a design review, the development team will need to provide documentation and diagrams related to the new application. The following is a non-comprehensive list of documentation that will help a design review go smoothly. While they are not hard requirements for a design review, AppSec cannot guarantee the quality of the review without materials provided by the development team.

 

###### Design Specifications

Providing a design specification of the application allows AppSec to review the planned design of an application. AppSec will provide consultation on various security requirements related to the design authentication, access controls, and other security controls.

 

###### Architecture Diagrams

A general architecture diagram detailing the major components of an application and how they interact is very useful for a threat model and design review. This typically is a diagram describing how components of an application integrate and function together.

 

###### Data Classifications

The risk of an application varies wildly, and almost always depend on the types of data the application handles and stores. Providing what classifications of data the application uses will give context to the security requirements an application requires.

 

###### Data Flow Diagram/Transitions

While very similar to an architecture diagram, a data flow diagram provides which data flows into and out of each part of the application. This makes it clear which components interact with each class of data, providing AppSec with a holistic view of the application, allowing for an effective threat model.

 

###### Stakeholders and Integrations

Applications don’t exist in a vacuum, and as such there will be additional Unity teams and third party services an application integrates or leverages. Providing a list of these teams and services will allow us to assess the risk an application poses.

 
#### Methodology

Since a design review ideally takes place early during the software development lifecycle, AppSec’s methodology for a design review is high level, and adapted to fit any application. Within a design review, we focus on two main components to ensure the review brings value to the development process.
###### Secure Design Consultation

Many times during the design of an application, the development team realizes the need for a security related control - such as role based access or authentication. While there are many resources online, very few times are they adapted to fit our needs here at Unity. As such, during the design review we provide recommendations and consultation regarding the design of security controls.

 
###### Threat Modelling

> “What could go wrong?” - Unnamed Developer, c. 2011

 

Threat models are the primary artifact from a design review. 

Most of the time, a threat model includes the following:

- A description / design / model of what you’re worried about
- A list of assumptions that can be checked or challenged in the future as the threat landscape changes
- A list of potential threats to the system
- A list of actions to be taken for each threat
- A way of validating the model and threats, and verification of success of actions taken

 

All of the above are discussed during the design review meeting, and act as a working discussion during the meeting. From that meeting we should be able to provide additional design recommendations and a list of controls used to mitigate the threats to an application. This should result in something very similar to a design flow diagram, but with notes regarding potential threats and the mitigating controls.
