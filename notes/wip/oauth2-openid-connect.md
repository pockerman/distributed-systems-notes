# Lesson 72: OAuth 2.0 and OpenID Connect

::: {.callout-warning title="Work in progress"}
This lesson has not been written yet. The outline below lists what it should cover.
:::

## Overview

OAuth 2.0 lets a user grant an application limited access to their data on another service. OpenID Connect builds on it to tell the application who the user is.

## Outline

- Roles: resource owner, client, authorization server, resource server
- Authorization code flow with PKCE
- Client credentials flow for service-to-service calls
- Access tokens, refresh tokens and scopes
- OpenID Connect: ID tokens and the userinfo endpoint
- Common mistakes: implicit flow, token leakage, missing state/nonce

## Related lessons

- [Lesson 38: Authentication & Authorization for APIs](../2026-10-06-AUTHENTICATION-AUTHORIZATION.md)
- [Lesson 39: Server-Stored Sessions vs JWTs for Authentication](../2026-07-16-Security-Server-Stored-Session-vs-JWT.md)

## References

1. Dick Hardt (ed.), _The OAuth 2.0 Authorization Framework_, RFC 6749
2. _OAuth 2.0 Security Best Current Practice_, RFC 9700
3. _OpenID Connect Core 1.0_, OpenID Foundation
