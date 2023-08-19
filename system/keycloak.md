# Features
* Single Sign On(SSO).
* Identity brokering and supports standard protocols. Supported OpenId connect, SAML 2.0, kerberos.
* social login like google, facebook and github etc.
* Connect to existing LDAP, Active directory. We can also implement our own provider using relational databases.
* Admin console and Admin management console.

# Install in docker
We can install and run keycloak using docker using this command.
```bash
docker run \
    --name keycloak-1 \
    -p 8080:8080 \
    -e KEYCLOAK_ADMIN=admin \
    -e KEYCLOAK_ADMIN_PASSWORD=admin \
    quay.io/keycloak/keycloak:22.0.1 start-dev
```
We can access the keycloak in browser. The address is: http://localhost:8080/. Next, click to the 'Administration console' section. The url is: http://localhost:8080/admin

In the docker command I'm using the username is admin and password is admin. Using those credentials, login to the console.

# Concepts
* Realm - Equivalent to tenant. Each realm has isolated settings, users, groups etc.
* Client - The application which needs authentication and authorization and keycloak will do those jobs. A client can be a web app or mobile app or another servers.
* Client scopes - This is a set of protocols and roles. Clients can reuse those sets. So, no need to create separate sets for multiple clients.
* Users - The users who will use those apps/clients.
* Groups - User groups.

# Creating realm
In admin console, go to the top right corner and click the dropdown. Give a name and click the create button.

![keycloak](keycloak-images/1.png)

# Create a client
Go to Clients -> Create Client. Follow the steps

![keycloak](keycloak-images/2.png)

![keycloak](keycloak-images/3.png)

![keycloak](keycloak-images/4.png)

# Create client specific roles
Go to the specific client and click the Roles tab.

![keycloak](keycloak-images/5.png)

Provide necessary information.

I've created 2 roles: app_admin and app_user.

# Create a default user
Go to Users->create user

![keycloak](keycloak-images/6.png)

Hit the Create button.

*Important: Set a password from the Credentials tab*

![keycloak](keycloak-images/10.png)

# Assign role to the default user
Once created the user, go to the Role mapping tab. Notice that 'app_admin' and 'app_user' roles aren't available.

![keycloak](keycloak-images/7.png)

Click the "Assign role" button. Still those aren't available. Select "Filter by clients", then we can see those roles.

![keycloak](keycloak-images/8.png)

# Token Generator endpoint
Go to Realm settings -> General

Click Endpoints -> OpenID Endpoint Configuration

![keycloak](keycloak-images/9.png)

We can see those info in a json format.
```json
  "issuer": "http://localhost:8080/realms/todoapp",
  "authorization_endpoint": "http://localhost:8080/realms/todoapp/protocol/openid-connect/auth",
  "token_endpoint": "http://localhost:8080/realms/todoapp/protocol/openid-connect/token",
  "introspection_endpoint": "http://localhost:8080/realms/todoapp/protocol/openid-connect/token/introspect",
  "userinfo_endpoint": "http://localhost:8080/realms/todoapp/protocol/openid-connect/userinfo",
  "end_session_endpoint": "http://localhost:8080/realms/todoapp/protocol/openid-connect/logout",
```

Take the token_endpoint and make a post request like this

```bash
curl -X POST http://localhost:8080/realms/todoapp/protocol/openid-connect/token -d "grant_type=password&client_id=todo-web&username=admin&password=admin"
```

We will get a response like this

```json
{
   "access_token":"eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJpYkNxUGFhM3BTUkk0LUZDSlVyVFFpa1RGMld0NTFBRkR1NnBrSjlMeGpFIn0.eyJleHAiOjE2OTIyOTg1OTYsImlhdCI6MTY5MjI5ODI5NiwianRpIjoiYzg1YjRhYjEtYzBiMC00YTQxLTgwMmEtNzdiZTBkMzYzYzY1IiwiaXNzIjoiaHR0cDovL2xvY2FsaG9zdDo4MDgwL3JlYWxtcy90b2RvYXBwIiwic3ViIjoiYmZkZjJmNjctM2ZhNy00MGI5LWE2M2MtMzM1Y2Q5NWI3Yjk2IiwidHlwIjoiQmVhcmVyIiwiYXpwIjoidG9kby13ZWIiLCJzZXNzaW9uX3N0YXRlIjoiYWVlOTcwYWUtYjc4Yy00YmZiLWI3NTMtZDMwM2U2MTg5MmIwIiwiYWNyIjoiMSIsImFsbG93ZWQtb3JpZ2lucyI6WyIqIl0sInJlc291cmNlX2FjY2VzcyI6eyJ0b2RvLXdlYiI6eyJyb2xlcyI6WyJhcHBfYWRtaW4iXX19LCJzY29wZSI6ImVtYWlsIHByb2ZpbGUiLCJzaWQiOiJhZWU5NzBhZS1iNzhjLTRiZmItYjc1My1kMzAzZTYxODkyYjAiLCJlbWFpbF92ZXJpZmllZCI6ZmFsc2UsInByZWZlcnJlZF91c2VybmFtZSI6ImFkbWluIiwiZ2l2ZW5fbmFtZSI6IiIsImZhbWlseV9uYW1lIjoiIiwiZW1haWwiOiJhZG1pbkBtYWlsLmNvbSJ9.YMx3kdlZfn7uVNTXc_MI5FlI1NkuzERWllJZuGWtZFlmBm1FrvDGP6geF7uzH-MpmKK7fVoOmNj5U_fUXBmrJJ0KonU6isDPyoAGIBip4ZwyMh9BlvzNEVohbSqFYYDivDEV8Rw1QhXP62wh8g1_sEAEtifKByPQ1iEcXCRt98DGFabu19IWhWzHtVCe8xEKDwF-b_89--JPqgx_z1ZFmKQau7umA5o3_fyYeZUptCxBKufJfUHKd8H0MGk0izhYKJilUtral7wyV71gXP1MemRI5oJ-IlDqy7mMjdjI_OEZeUNGtiKDyLLzNhFmtoBVdNOOPvA21Hcy_R9xWf_nUA",
   "expires_in":300,
   "refresh_expires_in":1800,
   "refresh_token":"eyJhbGciOiJIUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJlOTgwNTIwZS1iNmQ1LTRhMWYtOTIwZS03MzdiOTQ4MGM0OWUifQ.eyJleHAiOjE2OTIzMDAwOTYsImlhdCI6MTY5MjI5ODI5NiwianRpIjoiNmI0NmRkZjQtNDc0Yy00NWU2LTgxYWItYjk3NTI5Nzg5MzlmIiwiaXNzIjoiaHR0cDovL2xvY2FsaG9zdDo4MDgwL3JlYWxtcy90b2RvYXBwIiwiYXVkIjoiaHR0cDovL2xvY2FsaG9zdDo4MDgwL3JlYWxtcy90b2RvYXBwIiwic3ViIjoiYmZkZjJmNjctM2ZhNy00MGI5LWE2M2MtMzM1Y2Q5NWI3Yjk2IiwidHlwIjoiUmVmcmVzaCIsImF6cCI6InRvZG8td2ViIiwic2Vzc2lvbl9zdGF0ZSI6ImFlZTk3MGFlLWI3OGMtNGJmYi1iNzUzLWQzMDNlNjE4OTJiMCIsInNjb3BlIjoiZW1haWwgcHJvZmlsZSIsInNpZCI6ImFlZTk3MGFlLWI3OGMtNGJmYi1iNzUzLWQzMDNlNjE4OTJiMCJ9.Da950qygWlDUAmHgI8Z1_YAMubw9BvapZ8Qxo4ka5IQ",
   "token_type":"Bearer",
   "not-before-policy":0,
   "session_state":"aee970ae-b78c-4bfb-b753-d303e61892b0",
   "scope":"email profile"
}
```

Now copy the access_token value and go to the jwt.io debugger. It will decode the token. It may look like this

The header part
```json
{
  "alg": "RS256",
  "typ": "JWT",
  "kid": "ibCqPaa3pSRI4-FCJUrTQikTF2Wt51AFDu6pkJ9LxjE"
}
```

The payload part
```json
{
  "exp": 1692298596,
  "iat": 1692298296,
  "jti": "c85b4ab1-c0b0-4a41-802a-77be0d363c65",
  "iss": "http://localhost:8080/realms/todoapp",
  "sub": "bfdf2f67-3fa7-40b9-a63c-335cd95b7b96",
  "typ": "Bearer",
  "azp": "todo-web",
  "session_state": "aee970ae-b78c-4bfb-b753-d303e61892b0",
  "acr": "1",
  "allowed-origins": [
    "*"
  ],
  "resource_access": {
    "todo-web": {
      "roles": [
        "app_admin"
      ]
    }
  },
  "scope": "email profile",
  "sid": "aee970ae-b78c-4bfb-b753-d303e61892b0",
  "email_verified": false,
  "preferred_username": "admin",
  "given_name": "",
  "family_name": "",
  "email": "admin@mail.com"
}
```

And so on.