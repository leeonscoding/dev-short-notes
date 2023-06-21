# OAuth 2.0 terminology
OAuth(Open Authorization) is an open protocol for secure authorization.

## Key terms
* **Resource owner**: The user
* **Resource Server**: Where the user's secure resource are stored. Resource examples are files, photos etc. The resource server is also called as the API. The user must give access to some portion of their account of the resource server. The resource server must be able to accept and validate access tokens.
* **Authorization Server**: This can be the same as the resource server. This authorization server displays a prompt to the user for the approval or denies for the application request. This server is also responsible for granting access tokens after the user authorizes the application.
* **Client**: The application where the user first comes and it need the protected resource.
    * **Confidential client**: Can maintain the confidentiality of the client_secret. This clients are the web servers where the user can't access the source code.
    * **Public client**: Can't maintain the confidentiality of the client_secret. For example: mobile app, javascript app. The user can view the source code.

## Other terms
* **Access token**: Used for a authenticated request to the resource server/api. Using this a user can authorize a third party application to access their protected resources from outside of the application. The token has a corresponding duration of acccess, scope and other informations. The access token is a string.
* **Refresh token**: Used to get a new access token when the existing access token is expired.
* **Authorization code**: An intermidiate token used in the server-side app flow. An authorization code is returned to the client after a successful authorzation and then the client exchanges it for an access token.