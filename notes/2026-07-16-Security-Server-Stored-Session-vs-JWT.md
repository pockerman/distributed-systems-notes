# Lesson 38: Server-Stored Sessions vs JWTs for Authentication

## Overview

Server-stored sessions and JWT authentication are, respectively, a stateful and a stateless approach to keeping users logged in. Server-side sessions keep the user's session data on the server, which makes revocation and security control easier but requires server storage and coordination. JWTs (JSON Web Tokens) carry the data in a signed token held by the client, which allows stateless, scalable authentication at the cost of more complex token management and harder revocation.

## Session-based authentication (server-stored sessions)

Session-based authentication, also called cookie-based or stateful authentication, keeps track of the user's login state on the server:

1. When a user logs in with valid credentials, the server creates a session record (in memory, a cache, or a database) containing the user's ID and other information (login time, expiry, etc.).
2. The server sends a session ID, a long random string, back to the client, usually as a cookie.
3. On subsequent requests, the browser automatically includes the session cookie, and the server looks the ID up in the session store to retrieve the user's details.

In short, the server holds the authoritative session data, and the cookie is just a key to it.

### Advantages

This approach is simple and reliable. The server has a single source of truth for the authentication state, so revoking sessions in real time is easy: logging a user out, or invalidating a compromised session, is as simple as deleting the session record, after which any request with that session ID is rejected.
Session IDs are random, opaque strings, so they don't reveal any user data, and they are small (minimal bandwidth overhead).
Also, web frameworks support sessions out of the box, which makes them straightforward to implement.

### Disadvantages

Because the server must consult the session store on every request, sessions can add latency and create scaling issues for large applications.
In a distributed or microservices environment, sessions require either [sticky sessions](2026-05-11-Sticky-Sessions.md) (sending a user to the server that holds their session) or a shared session store, which adds complexity and overhead.
Each active session consumes server memory or storage, so resource usage grows with the number of users.
There are also security considerations: if an attacker steals a user's session cookie (e.g. via XSS, or by sniffing an unencrypted connection), they can hijack the session.
Moreover, cookie-based sessions are vulnerable to CSRF (cross-site request forgery) attacks, so developers must implement protections such as `SameSite` cookies or CSRF tokens.

## Token-based authentication with JWTs (stateless tokens)

Token-based authentication uses JWTs to keep the login state on the client side:

1. After a user logs in successfully, the server issues a signed token (the JWT) that encodes the user's identity and relevant claims, such as roles and an expiration time, in a JSON payload.
2. The client stores the token (for example, in memory, `localStorage`, or a cookie) and sends it with each request, often in an `Authorization: Bearer <token>` header.
3. The server does not store any session information; it verifies the token's signature and claims on each request.

JWTs are self-contained: all the information needed to authenticate the user is inside the token, so no database lookup is required while processing a request.

### Advantages

Because JWTs eliminate the per-request session lookup, they can reduce latency for APIs. The server only validates the token locally, which is a fast cryptographic signature check.
This makes JWTs highly scalable and well suited to distributed systems and microservices, since any server instance can validate a token without shared session state. The payload can also carry useful information (e.g. the user's role or permissions), so the server can make authorization decisions without extra queries.
JWTs work well for mobile and single-page applications, and for cross-domain or single sign-on (SSO) scenarios, where keeping state on a central server is inconvenient. The same token can be used across multiple services, as long as they trust the issuer's signature.
Finally, JWTs are signed (JWS), and optionally encrypted (JWE), to prevent tampering; sent over HTTPS, they are also kept confidential in transit.

### Disadvantages

The biggest drawback is that JWTs cannot easily be revoked or changed once issued.
A JWT remains valid until it expires; the server generally cannot pull it back or update the claims inside it. Revoking a user's access immediately (e.g. logging out of all devices, or downgrading a user's role) therefore requires waiting until the token expires, or maintaining a denylist, which undermines the stateless design.
For better security, JWT-based systems use short-lived access tokens (e.g. 15 minutes) together with refresh tokens, balancing user convenience with the ability to expire credentials.
JWTs are also larger than a session ID, so sending them with every request adds some network overhead.
If a JWT is stored in browser-accessible storage (e.g. `localStorage`), it is exposed to XSS attacks, since JavaScript can read it; if it is stored in a cookie, CSRF must be handled (`HttpOnly` and `SameSite` cookies mitigate both risks).
In short, JWTs put more responsibility on developers: token storage, key management, rotation, and validation must all be done correctly. Bugs or misconfigurations, like weak signing secrets, accepting the `none` algorithm, or not validating the signature, issuer, audience, and expiry, introduce vulnerabilities.

## Comparison

| Aspect | Server-stored sessions | Token-based authentication (JWT) |
|---|---|---|
| Where session data lives | On the server (in memory, a database, or a cache) | On the client, inside the token |
| How it's identified | A session ID in a cookie; the server looks it up | The JWT is sent with each request (usually in a header or a cookie) |
| Statefulness | Stateful: the server keeps track of each session | Stateless: the server stores nothing per user and only validates the token |
| Scalability | Harder: session data must be shared across servers (e.g. via Redis) or sessions must be sticky | Easier: no shared session store needed |
| Server storage | Grows with the number of active users | None per user; only the signing keys |
| Per-request cost | A session store lookup | A local signature check |
| Revocation | Easy: delete the session | Hard: wait for expiry, or add a denylist |
| Expiration and renewal | Controlled by the server; can be expired or extended at any time | Controlled by the token's `exp` claim; needs refresh tokens for renewal |
| Data exposure | The session ID is opaque | The payload is only encoded (Base64URL), so anyone holding the token can read it |
| Best suited for | Traditional server-rendered web apps | APIs, microservices, mobile apps, SSO |
| Implementation | Simple with frameworks like Express or Django | Requires careful signing, validation, and token rotation |

## Key trade-offs

Both methods solve the same problem, maintaining authentication on top of stateless HTTP, but in opposite ways:

- **Stateful vs stateless.** Server sessions are stateful: the server remembers each user between requests. JWTs are stateless: all the state is in the token, and the server keeps no per-user memory. Sessions therefore require managing state, which complicates scaling, while JWTs let you scale out easily, since any server can accept a valid token.
- **Scalability and architecture.** With multiple servers, sessions need a centralized session store or sticky sessions. JWTs shine in microservices and APIs, where each service can verify a token independently, without a single source of truth for sessions. This is one reason JWTs are popular for RESTful APIs and mobile apps.
- **Security and control.** With server-side sessions, the server has tight control: it consults the session store on every request and can invalidate sessions at will. JWTs trade some of that control for decentralization: the server trusts a validly signed token until it expires, so applications must design around this (short lifetimes, revocation lists, etc.). Also, a session ID is opaque, whereas a JWT's payload is only encoded and signed, not encrypted, so anyone holding the token can read it. Never put sensitive data in a JWT payload.
- **Performance and overhead.** Sessions add a store lookup on each request. JWT verification needs no lookup, but the token, often several hundred bytes, is sent with every request. In most cases this difference is minor.
- **Implementation complexity.** Session-based authentication is mature and widely supported, but you must run, and keep available, a session store. JWTs require careful handling of keys, expiry, refresh, and client-side storage; this offers more flexibility, but more can go wrong.

## Choosing between them

The choice depends on the application:

- **Traditional (monolithic) web apps.** For a server-rendered website, such as a classic e-commerce site or forum, server sessions are usually ideal: they are easy to use, scale well enough for most user bases, and allow instant invalidation (e.g. on password change or suspicious activity).
- **Microservices and APIs.** When many services need to authenticate the user, JWTs are convenient: the user logs in to an authentication service and receives a token, and every service can trust that token without a central session database. APIs serving mobile apps often use JWTs, so the app just stores a token and sends it with each request.
- **Single-page applications (SPAs).** SPAs that talk to a separate backend API frequently use JWTs. If the SPA and the backend are on the same origin, cookie-based sessions are also a perfectly good, and often safer, choice.
- **Security-sensitive applications.** Banking systems and admin dashboards, which need tight control over sessions and immediate revocation, often prefer server-stored sessions and accept their overhead.
- **Hybrid approaches.** It's not always either-or. For example, a system may issue short-lived JWTs for stateless authentication but keep a server-side record of refresh tokens, so that it can force a logout; or it may use JWTs for low-risk requests and fall back to a server-side check for critical actions.

## Summary

::: {.callout-note title="Summary"}
Server-stored sessions keep authentication state on the server and give the client an opaque session ID; they are simple, small, and easy to revoke, but need a shared session store (or sticky sessions) to scale and must be protected against CSRF. JWTs carry signed claims in the token itself, so any service can verify them locally without a lookup; they scale well and suit APIs, microservices, mobile apps, and SSO, but they are hard to revoke before expiry, larger, readable by anyone holding them, and easy to misconfigure. Short-lived access tokens with refresh tokens, or hybrid designs, combine the strengths of both.
:::

See also [Lesson 37: Authentication & Authorization for APIs](2026-10-06-AUTHENTICATION-AUTHORIZATION.md).

## References

1. [RFC 7519: JSON Web Token (JWT)](https://www.rfc-editor.org/rfc/rfc7519)
2. [OWASP Session Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html)
3. [OWASP JSON Web Token Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html)
