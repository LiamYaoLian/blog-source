---
title: JWT vs Session Cookie
date: 2022-11-04 10:41:58
tags:
---
# JWT
1. server authenticates (correct username and password)
2. server encrypts userId and expiresIn by ACCESS_TOKEN_SECRET to generate access token
```JavaScript
const accessToken =
jwt.sign( {userId}, ACCESS_TOKEN_SECRET, {expiresIn: "15m"} )
```
3. server sends access token to browser
4. browser saves access token
5. browser sends access token in subsequent requests
6. server gets access token from request header
7. server decrypt access token to get user id
```JavaScript
jwt.verify( accessToken, ACCESS_TOKEN_SECRET, (err, user) => {
if (err) console.log(err) // eg. invalid token, or expired token
req.user = user
})
```
8. server uses user id to look up persisted data, such as data in shoppingCartDB

* Pro
- No DB call required to get user id, so it's faster
- No need for shared session DB, can horizontally scale

* Con
- after logout, cannot invalidate token immediately

# Session Cookie
1. server authenticates (correct username and password)
2. server generates session id (signed with “secret key”)
```JavaScript
app.use( session ({ secret: “secret key”}) )
```
3. server saves session id in session DB
4. server sends cookie with session id to browser
5. browser saves cookie in cookie storage
6. browser sends cookie in subsequent requests
7. server gets session id from the request cookie.
8. server verifies session Id integrity using "secret key" and looks
up in session DB to get user id
```JavaScript
app.get("/", (req, res) => {
   console.log(req.session.id)      // get sessionId
   console.log(req.session.cookie)  // get cookie
})
```
9. server uses user id to look up persisted data, such as data in shoppingCartDB
