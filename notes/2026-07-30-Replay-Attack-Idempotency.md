# Lesson 39: Replay Attack vs Idempotency Issues

## Overview

A replay attack is a type of attack in which an attacker intercepts a valid data transmission and maliciously reuses it, by repeating or delaying it, to fraudulently repeat an action or gain unauthorized access.
In other words, the attacker "plays back" a previously sent message (such as a login token or a payment request) to trick a system into treating it as a new, legitimate request.

This is distinct from idempotency issues, which are problems that occur when a system processes the same request multiple times because of a design flaw, not necessarily maliciously, leading to unintended duplicate effects.

## Replay attacks

A replay attack, also known as a playback attack, is a security breach in which an attacker captures a legitimate message or transaction and resends it unaltered to reproduce its effect.
Because the message is a valid transmission, often already authenticated or encrypted, the system can be fooled into accepting it again.
It is often described as a simple form of man-in-the-middle attack, and it exploits the fact that the system cannot distinguish the replayed message from the original.

**How it works.** An attacker eavesdrops on the communication between a client and a server, captures a message (for example, a login request or a bank transfer request), and later sends exactly the same data to the server. The server, believing it is a new request, performs the action again. The attacker doesn't need to decrypt or understand the message; resending the captured bytes can be enough.

**Why it matters.** Replay attacks can have serious consequences. An attacker could replay a payment or money transfer request, causing duplicate charges or unauthorized withdrawals. In an authentication scenario, an attacker might capture a password hash or an authentication token and resend it to impersonate a user. In the classic example, Alice sends her hashed password to Bob (the server) to log in, and Eve, an eavesdropper, captures the hash. Later, Eve replays the same hash to Bob, who accepts it and grants access, thinking Eve is Alice.

**Common targets.** Replay attacks target protocols whose messages lack freshness indicators, like timestamps or one-time values. Naive authentication schemes that don't use nonces (one-time random numbers) or timestamped tokens are vulnerable: an attacker can replay a login request or a transaction because the system doesn't realize it is a duplicate. TLS protects against replay of captured records within a connection, but application-level messages (e.g. a signed webhook, or TLS 1.3 0-RTT early data) can still be replayable.

**Prevention.** To defend against replay attacks, systems make sure each request is unique or valid only once. Techniques include nonces, timestamps, and short-lived tokens, rejecting any message that is outside an expected time window or has been seen before. For instance, a server can require a cryptographic nonce in each request and remember it; if it sees the same nonce again, it knows the request is a replay and rejects it. Another approach is to include a timestamp and accept requests only within a short time window, making old messages useless. Secure APIs and webhooks typically sign the message together with a timestamp, so that a replayed message, even with identical content, is recognized as stale or already processed.

## Idempotency

In contrast to a replay attack, idempotency is not an attack but a property of an operation.
An operation is idempotent if performing it multiple times has the same effect as performing it once.
In practical terms, repeating the action doesn't change the result beyond the first execution.

The classic examples come from HTTP API design (see [Lesson 11: HTTP](2026-04-09-Part-8-HTTP.md)): a `GET` request is idempotent (no matter how many times you send it, you retrieve the data without side effects), and so is updating a record with `PUT` (sending the same update twice leaves the record in the same final state).
On the other hand, creating a new entry with `POST` is typically not idempotent: calling it multiple times can create duplicate entries.

### Idempotency issues

When a system does not handle repeated invocations of a non-idempotent operation, we get unintended side effects such as duplicate actions, charges, or records. These are idempotency issues.
They often appear when network glitches or user behaviour cause the same request to be sent more than once.
For example, imagine a user clicking a "Submit payment" button twice because the page was slow.
Without idempotency safeguards, the server might process two payments and charge the user twice.
In a distributed system, if a client times out and [retries](2026-02-04-Downstream-Resiliency.md) a request, a non-idempotent endpoint may perform the action again (e.g. place a duplicate order), since it doesn't know the first attempt succeeded.
Idempotency issues are essentially bugs or design flaws where the system fails to handle repeated messages safely.

### Why idempotency matters

Idempotency is crucial for building reliable and user-friendly systems, especially for financial transactions and calls to external APIs.
Duplicate processing harms user trust (nobody wants to be charged twice) and leads to data inconsistencies or extra work (issuing refunds, cleaning up duplicate records, etc.).
In distributed systems, failures and retries are common, and idempotency ensures that reprocessing a request doesn't cause unintended side effects.
In short, designing idempotent operations, or handling duplicates explicitly, e.g. with idempotency keys, makes systems robust against both user error and network issues.

## Replay attacks vs idempotency issues: key differences

Both involve repeated requests, so the two concepts are easy to confuse, but they are different:

- **Nature of the problem.** A replay attack is a security attack: a malicious actor deliberately reuses data to deceive a system. An idempotency issue is a design flaw: the system doesn't safely handle identical requests that arrive more than once, usually accidentally (e.g., because of retries or user actions). There is no malice in an idempotency issue, although an attacker could exploit a non-idempotent endpoint to amplify the effect of a replay.
- **Context.** Replay attacks are discussed in security and cryptography, in relation to authentication tokens, network protocols, or transactions whose messages can be intercepted. Idempotency is discussed in software and API design, in relation to the reliability of operations in REST APIs, webhooks, and distributed systems. For example, webhook receivers are made idempotent because the sender re-sends an event when it doesn't receive an acknowledgment; this is about normal retries, not attackers.
- **Goal and consequence.** The attacker's goal in a replay attack is to gain an unauthorized benefit, e.g. get a service twice for the price of one, duplicate a payment to themselves, or log in as someone else. The consequence of an idempotency issue is an accidental duplication, which causes errors (duplicate data, extra charges) through oversight rather than intent. The outcome may look the same (two payments instead of one), but the cause is different.
- **Prevention.** Replay attacks are prevented with security measures, such as nonces, one-time tokens, timestamps, and signatures, which ensure each request is unique and can't be replayed out of context. Idempotency issues are prevented by designing operations to be idempotent or by handling duplicates explicitly, e.g. with idempotency keys, by checking whether an action has already been performed, or by using naturally idempotent operations (`PUT`, `DELETE`). Replay prevention keeps attackers from abusing your system; idempotency makes it robust against unintentional repeats.

The two concepts do interact: designing for idempotency limits the damage of some replay attacks.
If an attacker replays a request that carries an idempotency key, e.g. a payment, against an endpoint that deduplicates on that key, the replay has no additional effect, because the system treats it as already handled.
Similarly, webhook handlers are made idempotent so that a replayed event is processed only once.

However, idempotency is not a substitute for security. Deduplication only catches exact repeats within the window for which keys are remembered, and many replayed requests are harmful even when processed once more; a replayed login request, for example, still grants the attacker access.
Both security measures (to stop malicious replays) and idempotent design (to handle repeats safely) are needed.

## Summary

::: {.callout-note title="Summary"}
A replay attack is a deliberate security attack in which captured, valid messages are resent to repeat an action or impersonate a user; it is prevented with freshness mechanisms such as nonces, timestamps, short-lived tokens, and signatures. Idempotency is a design property: an idempotent operation has the same effect no matter how many times it is performed. Idempotency issues are accidental duplicate effects, caused by retries or repeated user actions on non-idempotent operations, and are prevented with idempotent designs and idempotency keys. Idempotency limits the damage of some replays, but it does not replace security controls; robust systems need both.
:::

## References

1. [Replay attack](https://en.wikipedia.org/wiki/Replay_attack)
2. [Idempotence](https://en.wikipedia.org/wiki/Idempotence#Computer_science_meaning)
3. [RFC 9110: HTTP Semantics, Idempotent Methods](https://www.rfc-editor.org/rfc/rfc9110#name-idempotent-methods)
