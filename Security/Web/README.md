# Security
---
**Identity** - Who you are?
**Role** - Who you are in the current context
**Claims** - Arbitary information about a person

For Eg: Driving License is required for verification if police stopped. If the same information is shown to College Security, security will not accept it because for College Entrance the student has to show college id card

### Types of Authentication
- **Token Based Authentication** - Users and claims are managed by OAuth service providers. For Eg: Facebook, Google, Github
Used mainly for Mobile/Javascript
- **Cookie Based Authentication** - Users, roles and claims are managed by application.
Used mainly for Server to Server Authentication

In .Net core there are four classes which supports Authentication and Authorization
 - IdentityUser
 - Identity Role
 - Identity User Role
 - Idenity User Claim

