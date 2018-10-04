---
layout: post
title: A Brief Intro to JWTs
date: 2018-10-04
comments: true
categories:
- Authentication
tags:
- auth
---

Json Web Tokens (JWTs) have become a popular way of handling authentication in Single Page Applications (SPAs). This post is the first in a series covering how to do authentication between an Angular app and a .Net Core Web API Server using JWTs. In this post we go through a brief introduction of JWTs.

<!-- more -->

## What are JWTs?

Lets start by having a quick overview of what JWTs are. I won't go into too much detail as there are other resources on the web which do a better job. However in this post we cover enough to understand the basics.

A Json Web Token (JWT) is simply a Json object containing security information about who a user is and what they can do. This is represented as a set of claims such as a user's name, roles, email etc. 

You can put anything you like in these claims, however there are some registered claims defined by the "JWT Spec" such as the time at which the claim was created, the expiry time etc. These claims are not required however they are recommended and you should be aware of them so you don't accidently redefine them as something else.

See [https://tools.ietf.org/html/rfc7519](https://tools.ietf.org/html/rfc7519) for more details. 

## How are they used?

JWTs are commonly used when authenticating between a SPA client and a Server Side Web API. This is usually done in the following way:

- The client sends some credentials to the server.
- The server checks the credentials. If the credentials are correct it generates a JWT containing claims about the user and sends it back in the response.
- The client sends the JWT in its HTTP "auth" header when making subsequent requests.
- The server Authenticates and Authorizes the client's requests by checking the claims in the JWT.

## How are they created?

A few things stand out as being obvious from the above flow. 

- When inspecting a JWT the server needs to be able to verify who generated it.
- The server also needs to be able to verify that the JWT hasn't been altered since it was generated, i.e. no claims have been added, removed or modified.
- The JWT should be encoded making it easier to transfer via HTTP.

These concerns are handled by base64 encoding the claims and cryptographically signing them. In particular a JWT is made up of three parts:

#### Parts of a JWT

- *The Header* - A Json object describing the hashing algorithm (used to cryptographically sign the token) and the type of token. e.g. 
```json
{
    "alg": "HS256",
    "typ": "JWT"
}
```

- *The payload* - A Json object containing claims.  e.g.
```json
{
    "name": "John Smith",
    "email": "john@example.com",
    "roles": ["admin"],
    "iat": 1516239022
}
```

- *The Signature* - Cryptographically sign the header and the payload using a secret. This allows us to check who generated the JWT and to ensure that the JWT hasn't been altered.

#### Generating the JWT

The JWT is generated with the following operation
```
base64encode(header) + '.' + 
base64encode(payload) + '.' + 
cryptographicallySign(
  base64encode(header), 
  base64encode(payload), 
  secret)
```

The result is a string made up of three parts seperated by dots which can be easily transmitted over HTTP.

Now that we know a little about JWTs the next post in this series looks at how we generate them using .Net Core.
