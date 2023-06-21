# How SSO(Single Sign On) works
SSO is a broder term. There are many protocols, types and workflow. I'm listing here the most simpliest term. This is called the CAS(Central Authentication Service) architecture.

## Parties
There are 3 parties involved in a SSO workflow
* **Principal**: The user who is trying to access a service or web application.
* **Service Provider**: The web application the user tries to access.
* **Identity Provider**: Authentication and token provider.

## Workflow
1. A user tries to access a secure resource of an web application or website which is our service provider.
2. The service provider then sends a token that contains some information about the user like their email address to the Identity provider.
3. The identity provider first checks if the user has already been authenticated or not. If authenticated go to step 5.
4. If the user isn't logged in then the identity provider sends a browser promt for logging in. The user may need to provide username and password or in a multi-factor auth scenario, OTP.
5. Once the creadentials are validated, the identity provider sends a token to the service provider confirming a successful authentication.
6. The service provier then sends this token to the user and this token will be saved to the user's browser.
7. In next request, the user will send this token to the service provider.
8. The service provider will use this token for next processing. No need to authenticate the user again. So, the user don't need to provide their credentials again until the token is expired.

## Token types
Now a days typically there are 2 types of tokens.
* SAML - Security Assertion Markup Language: Uses XML for generating tokens.
* OIDC - Open ID Connect: Uses JWT(Json web token) for generating tokens.
SAML is best for some enterpise use cases. OIDC supports varity of devices. It is best to use OIDC for newer applications.

## Some commercial SSO provider
* OneLogin
* Okta/auth0
* Azure AD
