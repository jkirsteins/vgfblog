---
title: "Auth0+spa+backend"
date: 2019-12-06T05:52:41+01:00
draft: true
tags: [c#,javascript,dotnet core,aspnet core,spa,auth0,jwt,authentication]
---

For authentication, I chose to use Auth0.

It's pretty great. No sign-up/sign-in development, no testing, nothing. Should be a few hours worth of investment tops, right?

It's an SPA (important point, as I presume the SDK for a regular web app would not have this issue).

> Note to self: check the flow with regular cookies

Mostly fine, but there are two issues. 

# How does Auth0 + SPA + backend work?

Skippable background.

# JWT are not revokable

You can never invalidate a session server-side with JWTs as long as it does not expire.

# Safari logout

Every refresh logs you out due to the "Prevent cross-site tracking"

> Note to self: add a diagram?

This does not happen with Chrome and other browsers because they can make a request to Auth0 (which has cookies) and fetch the JWT again.

## Can we keep the JWT locally?

Could store the JWT in 

- technically could (and e.g. Firebase appears to do this)
- less secure. Local storage is not a good place for storing secrets
- couple that with JWTs lasting "until expiration" usually, and you have no way to mitigate JWT loss normally

# Solution to both problems

Use the JWT to bootstrap a regular cookie based authentication.

> Note to self: add a diagram?

With this baseline in place:
- you have everything in place to track used JWTs to prevent reuse
- you can invalidate a session whenever you need to