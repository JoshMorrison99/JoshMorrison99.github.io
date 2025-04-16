---
title: PortSwigger - JWT Authentication Bypass via Unverified Signature
date: 2025-04-15 18:20:00 -5000
categories: [CTF]
tags: [ctf, web, jwt]     # TAG names should always be lowercase
---

<br/>

This lab uses a JWT-based mechanism for handling sessions. Due to implementation flaws, the server doesn't verify the signature of any JWTs that it receives.

<br/>

To solve the lab, modify your session token to gain access to the admin panel at `/admin`, then delete the user `carlos`.

<br/>

You can log in to your own account using the following credentials: `wiener:peter`

<br/>

![](/assets/portswigger/jwt-authentication-bypass-via-unverified-signature/0.png)
*Website Landing Page*

<br/>

Login with the given credentials: `wiener:peter`

![](/assets/portswigger/jwt-authentication-bypass-via-unverified-signature/1.png)

<br/>

After logging in, we are given a JWT.

![](/assets/portswigger/jwt-authentication-bypass-via-unverified-signature/2.png)

<br/>

I like to use [jwt.io](https://jwt.io) to view JWTs

![](/assets/portswigger/jwt-authentication-bypass-via-unverified-signature/3.png)

<br/>

By trying to access `/admin`, we get a 401 Unauthorized and a message saying: `dmin interface only available if logged in as an administrator`.

![](/assets/portswigger/jwt-authentication-bypass-via-unverified-signature/4.png)

<br/>

Using the following [Burp extension](https://github.com/portswigger/jwt-editor), we can easily edit JWTs. After installing, we will now have a tab to edit the JWT.

![](/assets/portswigger/jwt-authentication-bypass-via-unverified-signature/5.png)

<br/>

By changing the `sub` field’s value from `wiener` to `administrator` will allow us to check if the JWT is being verified or not on the backend.

![](/assets/portswigger/jwt-authentication-bypass-via-unverified-signature/6.png)

<br/>

Requesting `/admin` again with the edited JWT, we now get a 200 success.

![](/assets/portswigger/jwt-authentication-bypass-via-unverified-signature/7.png)

<br/>

To delete the `carlos` user you can see that we need to send a request to `/admin/delete?username=carlos`

![](/assets/portswigger/jwt-authentication-bypass-via-unverified-signature/8.png)

<br/>

![](/assets/portswigger/jwt-authentication-bypass-via-unverified-signature/9.png)

<br/>

![](/assets/portswigger/jwt-authentication-bypass-via-unverified-signature/10.png)
*Lab Solved*
