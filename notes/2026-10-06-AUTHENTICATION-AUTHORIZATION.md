# Lesson 37: Authentication & Authorization for APIs

## Overview

In [Lesson 10: APIs](2026-04-09-Part-7-API.md) we discussed the options for exposing a service to the outside world.
By doing so, however, we open the door to a range of security problems. In this lesson we discuss how to protect an API,
namely with authentication and authorization.

## Authentication vs authorization

Let's start by separating the two, since authentication is different from authorization:

- **Authentication** answers the question: _who_ is making the call?
- **Authorization** answers the question: is this identity _allowed_ to perform this operation on this resource?

There are various mechanisms for authenticating API callers, e.g. server-side sessions and JSON Web Tokens (JWTs); see [Server-Stored Sessions vs JWTs](2026-07-16-Security-Server-Stored-Session-vs-JWT.md).
For programmatic API access, the most common credentials are API keys, opaque access tokens, and signed access tokens.

## Credential types

### API keys

An API key is a long, random, opaque secret issued to a program (rather than to a human user).
Good practice is to:

- store only a protected verifier of the key (e.g. a hash), never the key itself;
- show the full secret only once, when it is created;
- allow two keys to be active at the same time, so that a client can rotate keys without downtime.

API keys have costs: long-lived secrets are valuable if leaked, rotation must be supported, a key shared by a team gives weak attribution to individual people, and keys are often granted far more power than they need.

### Opaque access tokens

An opaque access token is a random value whose meaning (identity, permissions, expiry) is looked up by the service, e.g. in a token store or by asking the authorization server (token introspection).
Its lifetime is a policy choice.
Verification requires a lookup, or a cached result of one, so the availability of the token store and the cache duration determine how quickly a revocation takes effect.

### Signed access tokens

A signed access token, such as a JWT, carries claims like the subject, tenant, scopes, and expiry, signed by the issuer.
Verification must check the signature using an allowed algorithm and a trusted key, and must also check the expected issuer, audience, and time constraints (expiry, not-before). A valid signature alone is not enough.
Local verification avoids a central lookup on every request, but revoking a token before it expires needs additional mechanisms.

::: {.callout-note title="Remark"}
A valid signed token normally cannot be withdrawn before it expires without adding a lookup, a denylist, or a key-rotation mechanism. See [Withdrawing a credential early](#withdrawing-a-credential-early) below.
:::

## Credentials belong in the `Authorization` header

Send credentials in the `Authorization` header:

```http
GET /payments/pay_81M HTTP/1.1
Host: api.example.com
Authorization: Bearer <credential>
```

Do not put them in the URL, e.g. `GET /payments?access_token=<credential>`.
URLs end up in browser history, proxy and server logs, monitoring systems, analytics tools, screenshots, copied links, and `Referer` headers. A secret in a query parameter spreads into all the systems that were built to record URLs.

The request body is also the wrong place: authentication happens before many servers parse the operation-specific body, and every operation would have to duplicate the field.

Transport security (TLS) is mandatory. A bearer credential grants its power to whoever possesses it, so sending it over an unprotected connection defeats every later check.

## 401 vs 403

- Return **`401 Unauthorized`** when valid authentication credentials are missing, invalid, expired, or revoked. A `401` response must include a `WWW-Authenticate` header that tells the client how to authenticate.
- Return **`403 Forbidden`** when the caller is authenticated, but the request is not permitted, e.g. because the token lacks a required scope. A different grant may fix a `403`; re-authenticating with the same grant will not.

The name "401 Unauthorized" is historically awkward: in practice, `401` means "authentication is needed" and `403` means "the authenticated identity is not allowed".

For example, a request whose token lacks the `refunds:create` scope could receive:

```http
HTTP/1.1 403 Forbidden
Content-Type: application/json

{
  "error": "insufficient_scope",
  "required_scope": "refunds:create"
}
```

When it is safe to disclose, name the missing permission, so the caller can request the right grant instead of guessing. OAuth bearer-token implementations follow the error format of [RFC 6750](https://www.rfc-editor.org/rfc/rfc6750) (e.g. `error="insufficient_scope"` in the `WWW-Authenticate` header).

## Scopes are the authorization contract

Use a clear resource-and-action name for each scope:

```
payments:read
payments:create
refunds:create
webhook_registrations:write
```

This vocabulary tells the caller exactly what access it is requesting. Avoid vague scopes such as `admin`, `full_access`, or `write`, which hide which actions are granted and make least privilege difficult.

Grant the smallest useful set of scopes. A reporting integration may need `payments:read` and `refunds:read`, but not `payments:create`. An internal refund tool may need `payments:read` and `refunds:create`, but not the ability to rotate webhook secrets. Least privilege limits what a leaked or misused credential can do.

## Scope is necessary but not sufficient

A credential with `payments:read` may read the payments owned by _its_ merchant; it may not read every payment in the service.
Authorization therefore needs at least two checks:

1. Does the credential grant the operation (scope)?
2. Does the addressed resource belong to the owner or tenant the credential is allowed to act for (ownership)?

Do not assume that an unguessable ID replaces the ownership check. Opaque IDs reduce accidental disclosure and enumeration; they do not grant permission.

Some actions also need additional policy checks. Even an owner with `refunds:create` may be refused because the payment is not refundable, the refundable amount is zero, the tenant has disabled manual refunds, or an internal approval is required above a threshold. These are authorization or current-state decisions, and each should return a stable, specific error code rather than one generic "forbidden".

## Private resources return 404

Suppose payment `pay_81M` exists and belongs to merchant A, and merchant B requests it.
Returning `403` would confirm that the guessed ID exists. For private resources, return the same response as for an ID that doesn't exist:

```http
HTTP/1.1 404 Not Found
Content-Type: application/json

{
  "error": "payment_not_found"
}
```

The cost is deliberate ambiguity: a legitimate caller cannot distinguish a typo from a resource outside its ownership scope. That is acceptable, because revealing the existence of another customer's resources is worse.

Use `403` for a permission failure on a resource whose existence the caller is already allowed to know. Use `404` when the existence of the resource is itself sensitive.

## Short-lived credentials for browsers

A browser must never receive a merchant's long-lived secret key. Instead, the merchant's server creates (or obtains from the API) a short-lived token bound to:

- one merchant,
- one payment or payment session,
- a small set of scopes,
- an expiry,
- and any permitted amount or action.

For example, the claims of such a token could look like:

```json
{
  "sub": "merchant_A",
  "payment": "pay_81M",
  "scope": "payments:confirm",
  "exp": 1767225600
}
```

The browser cannot swap `pay_81M` for another payment ID, because the server reads the binding from the credential and still checks it against the request path. This lets browser code act on one payment without exposing the server's full authority to client-side code.

## Withdrawing a credential early

How a credential is revoked depends on its type:

- **API key:** mark it revoked in the credential store; subsequent verification fails with `401`. Support two active keys during rotation, so that a client can deploy the replacement before the old key is removed.
- **Opaque token:** remove it, or mark it inactive. Since verification includes a lookup, withdrawal is immediate, subject to any documented cache lifetime.
- **Signed token:** there are four options, and all of them cost something:
  1. keep expiry short and wait it out: this limits exposure but does not revoke the token immediately;
  2. add a revocation lookup or a denylist: this adds a lookup to every request;
  3. rotate the signing key: this invalidates many tokens at once, and depends on how quickly verifiers refresh their keys;
  4. embed a credential-version claim and check it against the current account state: this also adds a lookup.

Choose the option that matches the security requirement, and document the intended behaviour, e.g. "access tokens live for 10 minutes; revoking an account takes effect immediately because the account version is checked on each request".

## Putting it together

The complete request decision looks like this:

1. Credentials are sent only in the `Authorization` header, over TLS.
2. A missing, invalid, expired, or withdrawn credential returns `401`.
3. A valid identity that lacks the required scope returns `403`.
4. Scopes are verbs on resources, such as `payments:create`, and callers receive the smallest set that does their job.
5. The scope check is followed by ownership and policy checks.
6. Another tenant's private resource returns the same `404` as a missing ID.
7. API keys and opaque tokens can be withdrawn through a lookup; signed tokens avoid that lookup, but need short expiry or a revocation mechanism for early withdrawal.

## Summary

::: {.callout-note title="Summary"}
Authentication establishes who is calling; authorization decides whether that identity may perform the operation on the resource. APIs commonly use API keys, opaque tokens (verified by lookup and easy to revoke), or signed tokens such as JWTs (verified locally, but hard to revoke before expiry). Credentials belong in the `Authorization` header over TLS, never in URLs. Missing or invalid credentials yield `401`, insufficient permissions yield `403`, and private resources the caller doesn't own yield `404` to avoid leaking their existence. Scopes should be narrow resource-and-action names granted on a least-privilege basis, and they must be complemented by ownership and policy checks. Browsers should only receive short-lived, narrowly bound tokens, and every credential type needs a defined revocation story.
:::

## References

1. [RFC 9110: HTTP Semantics, 401 and 403 status codes](https://www.rfc-editor.org/rfc/rfc9110#name-401-unauthorized)
2. [RFC 6750: The OAuth 2.0 Authorization Framework: Bearer Token Usage](https://www.rfc-editor.org/rfc/rfc6750)
3. [RFC 7519: JSON Web Token (JWT)](https://www.rfc-editor.org/rfc/rfc7519)
4. [RFC 7662: OAuth 2.0 Token Introspection](https://www.rfc-editor.org/rfc/rfc7662)
5. [OWASP API Security Top 10](https://owasp.org/API-Security/)
