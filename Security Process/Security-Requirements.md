# Application Security Requirements at Unity [Security Process]
<font size="-1">_Author: Carlo Valentin - Dec. 2018_</font>

### What are Security Requirements?

Applications have a range of requirements that are taken into consideration during the development process. For example, there may be considerations made to who needs to use an application, that will decide whether it is public facing or not. Similar to this, there are a number of security-related requirements, which should be considered during the development process.

### How do I identify security requirements?

Security requirements are any needs or requirements of an application that act or perform some sort of security control. These requirements include concepts like authentication or role-based access controls (RBAC). Many of these requirements can be determined by following [secure design principles](../Coding%20Practice/Secure-Design-Principles.md).

 

An example of security requirements that an application can encounter:

- Authentication
  - Requirement for any application that has multiple user levels of access. Ensures non-repudiation and is the foundation for the principle of least-privilege.
- Authorization/Role Based Access Control
  - Requirement for any application that has multiple user levels of access. Ensures implementation of the principle of least-privilege.
- Input Validation/Output Encoding
  - Requirement for any application that receives untrusted data, and exposes said data back to the user. Ensures the authenticity and integrity of the data consumed by the application.
- Secrets Management
  - Requirements for where to store secrets, such as API tokens, that are used by the application.
- Encryption
  - Requirements for how and when to encrypt data at rest or in transit.
- Logging
  - Requirement for many applications to ensure non-repudiation of application actions.

### Once I have these requirements, what do I do?

A failure or flaw in the implementation of a security-related requirement can lead to additional various vulnerabilities or in some cases require a re-architecting of the application. As such, care needs to be taken when implementing the identified requirements. If you and your team are unsure about the design or implementation of security requirements, please schedule a [design review](./Design-Review.md) or [implementation review](./Implementation-Review.md) with the Unity AppSec team.
