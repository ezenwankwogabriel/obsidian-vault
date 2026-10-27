Http which browsers use to connect with a server, is essentially stateless. Each request is independent. Server doesn't auto remember who made the previous request
So a need to remember a logged in user comes up

Cookie: Something your browser stores and sends back to the server
Session: A record the server uses to remember you
JWT: A signed token that carries information the server can verify

### Cookie
This is an ID card you carry, stored in the browser. 
After loggin in, a server might send a response as: `Set-Cookie: session_id=abc123`
Browser stores the cookie. On later request, it sends it back: `Cookie: session_id=abc123`
The server uses this to recognize it was you.
It typically contains a session ID, a JWT or other small pieces of information. It doesn't authenticate you.

### Session
This is the record the server keeps
When you present your card (cookie), the server looks up its sessions record and validates it authentic.
Typically: 
Log in > server creates session > sends to browser
Browser sends cookie on subsequent request > server looks up cookie and retrieves session information

This could be redis, db, memory storage

### JWT
A signed pass you can verify
Instead of looking up a card everytime, imagine a signed digital pass containing information about you. Can be verified to see if it was issued by the server or if the contents been modified.
Typically contains information as:
```
{
  "userId": "user123",
  "role": "admin",
  "exp": 1792000000
}
```
The server signs the token so it can verify its authenticity. On log in:
Server verifies email & password, creates and sign JWT, client receives JWT, and sends it with subsequent requests, sever verifies the signature and checks if it is valid/expired before accepting it


## How they work together
![[Pasted image 20261015101353.png]]

You'd see that session & jwt can either be used in the server, saved in cookie in browsers.



The catch with JWTs
Suppose a user logs out or their account is compromised.
A session can be invalidated centrally, so subsequent requests using that session can be rejected.
A self-contained JWT, however, may remain valid until it expires unless you introduce revocation checks or another mechanism. Short-lived access tokens and securely managed refresh tokens are common ways to manage this trade-off.
Remember this for system design interviews
- Session: The server remembers the login.
- JWT: The token carries verifiable identity claims.
- Distributed systems: JWTs can let multiple services verify identity independently.
- Statelessness: JWTs remove the need for a per-user server-side login lookup in the common case; they don't eliminate application state.
The most important takeaway: JWT is a tool for stateless authentication, not a guarantee that your distributed system is stateless.