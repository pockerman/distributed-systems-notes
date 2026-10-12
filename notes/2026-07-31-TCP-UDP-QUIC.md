# TCP, UDP & QUIC

## Overview

TCP, UDP, and QUIC are transport protocols that make different trade-offs between reliability and speed: TCP is reliable and connection-oriented, UDP is lightweight and connectionless (no delivery guarantees), and QUIC runs on top of UDP to offer fast, secure connections with TCP-like reliability.
Below, we look at what each protocol is, how they differ, and when to use each.

## TCP

The Transmission Control Protocol (TCP) is a connection-oriented protocol that provides reliable, ordered delivery of a byte stream; see [Part 4 | TCP 101](2026-03-17-part-4-TCP_101.md) for the details.
TCP requires a three-way handshake to establish a connection: the client sends a SYN segment, the server responds with a SYN-ACK, and the client confirms with an ACK. Only after this handshake do the two sides exchange data, which ensures both are ready and have agreed on sequence numbers.
TCP also acknowledges and retransmits lost segments, reorders out-of-order segments, and performs flow and congestion control.

Because of these features, TCP takes longer to start communicating and adds overhead.
However, it is the backbone of most internet applications that need accuracy over speed.

## UDP

The User Datagram Protocol (UDP) is a much simpler, connectionless transport protocol. It does not set up a connection or perform a handshake before sending data.
UDP simply sends packets, called datagrams, to the recipient, without checking whether they arrive or in what order.
This means UDP has minimal overhead and latency: there is no waiting for connection setup or acknowledgments, at the cost of reliability.
With UDP, packets may get lost, arrive out of order, or be duplicated, and UDP won't fix that (it only has an optional checksum to detect corruption).
The protocol is often described as "fire-and-forget".

This makes UDP ideal for applications where low latency matters more than perfect delivery.
Think of a video call or an online game: waiting for lost packets to be retransmitted would hurt the experience more than an occasional dropped frame or missed update.
If a few packets are dropped, the call or game continues with perhaps a minor glitch, but no significant disruption. Real-time media protocols such as RTP and WebRTC are therefore built on UDP.

UDP's lightweight design also suits simple request-response exchanges like DNS lookups, where a quick query and answer are needed with minimal overhead.
If a DNS packet is lost, the client simply retries, rather than paying for a TCP connection on every lookup.

Another unique aspect of UDP is that it supports multicast and broadcast, i.e. sending data to multiple recipients at once, because it doesn't require a one-to-one connection. TCP is strictly one-to-one.

However, because UDP doesn't guarantee delivery, it is unsuitable on its own for applications that require reliability, such as file downloads or database communication; such applications would have to implement reliability themselves.

In short, UDP trades reliability for speed and simplicity, and is used when some data loss is acceptable but low latency is critical.

## QUIC

QUIC is a modern transport protocol designed to combine the best of TCP and UDP.
It was originally developed at Google and later standardized by the IETF in 2021 ([RFC 9000](https://www.rfc-editor.org/rfc/rfc9000)), and it is the basis of HTTP/3. QUIC runs on top of UDP but adds TCP-like features such as reliable, ordered delivery and congestion control, while reducing latency and improving security.
In simple terms, QUIC is a next-generation TCP implemented over UDP, with encryption built in.

### Faster connection setup

One of QUIC's big advantages is connection setup.
QUIC integrates the transport handshake and the TLS 1.3 handshake into a single step, so a new connection is established in one round trip (1-RTT), compared to two round trips for TCP plus TLS 1.3 (and three with TLS 1.2).
When a client reconnects to a server it has talked to before, it can even send data in its very first packet (0-RTT). Note that 0-RTT data can be replayed by an attacker, so it should only be used for idempotent requests (see [Replay Attack vs Idempotency Issues](2026-07-30-Replay-Attack-Idempotency.md)).
The result is lower latency, which is especially beneficial for web pages and API calls.

### Encryption by default

QUIC has encryption built in: all QUIC connections use TLS 1.3, and there is no unencrypted mode.
QUIC also encrypts most of its packet header, unlike TCP, whose headers travel in the clear. This improves privacy and prevents middleboxes on the network from inspecting or interfering with the transport, which in turn makes it easier to evolve the protocol.

### Stream multiplexing without head-of-line blocking

QUIC supports multiple independent streams within a single connection, so an application can send parallel streams of data that don't block each other.
With TCP, if you multiplex several streams over one connection (as HTTP/2 does), a single lost packet stalls all of them, because TCP delivers bytes strictly in order. This is called head-of-line blocking.
QUIC avoids this by handling streams independently: a lost packet only delays its own stream. This makes better use of the connection, especially for web pages that load many resources simultaneously.

### Connection migration

QUIC also handles network changes gracefully.
A QUIC connection is identified by a connection ID that is not tied to the client's IP address and port, which allows connection migration.
For example, when a mobile device switches from Wi-Fi to cellular data, a QUIC connection can continue without a new handshake, whereas a TCP connection, identified by its IP addresses and ports, would break.
This makes QUIC attractive for mobile applications whose users frequently change networks.

In summary, QUIC is a UDP-based, encrypted protocol that provides TCP-like reliability, designed to reduce latency on the modern web.
Major services such as Google's and Meta's use it heavily, and browsers and CDNs that support HTTP/3 use QUIC to load pages faster and more reliably on poor networks.

## Key differences

| Aspect | TCP | UDP | QUIC |
|---|---|---|---|
| Connection setup | Connection-oriented; 3-way handshake (plus a separate TLS handshake) | Connectionless; no handshake | Combined transport + TLS handshake: 1-RTT, or 0-RTT on resumption |
| Reliability and ordering | Reliable, in-order byte stream | None: packets may be lost, duplicated, or reordered | Reliable and ordered per stream |
| Head-of-line blocking | Yes, across everything multiplexed on the connection | N/A | No, between streams |
| Congestion and flow control | Yes, mature algorithms in the kernel | None; the application must handle it | Yes, implemented in user space, so it can evolve without OS updates |
| Security | None built in; add TLS on top | None built in; add DTLS or application-level encryption | Always encrypted (TLS 1.3); most of the header is encrypted too |
| Multiplexing | Single byte stream per connection | Independent datagrams | Multiple independent streams per connection |
| Connection identity | 4-tuple (source IP and port, destination IP and port); changes break the connection | No connection | Connection ID; survives IP and port changes |
| Multicast / broadcast | No | Yes | No |
| Maturity | Decades old, in every OS kernel | Decades old, in every OS kernel | Standardized in 2021; usually provided by user-space libraries (e.g. quiche, msquic, ngtcp2) |

A few practical caveats:

- **Latency.** UDP has the lowest latency, since it doesn't wait for anything. QUIC's faster handshake and lack of head-of-line blocking often let it match or beat TCP for web traffic, but because it runs in user space it typically costs more CPU per byte than kernel TCP.
- **Congestion.** TCP and QUIC slow down automatically when the network is congested; UDP does not, so applications using UDP must implement their own rate control to avoid overwhelming the network.
- **Middleboxes.** Some corporate networks and firewalls block or throttle UDP, and NAT devices often expire idle UDP mappings faster than TCP ones. QUIC clients therefore usually fall back to TCP (e.g. HTTP/2) when QUIC is blocked.

In essence, TCP gives you tried-and-true reliability, UDP gives you raw speed and simplicity, and QUIC aims to give you both speed and reliability, with modern features, by building on UDP.

## Choosing the right protocol

Choosing between TCP, UDP, and QUIC depends on the needs of your application.

**Use TCP** when you need reliable, ordered delivery and cannot tolerate data loss, and when accuracy is more important than speed. Typical use cases:

- Web pages and file downloads: every byte must arrive intact, since a missing packet could break a page or corrupt a file.
- Email and messaging: protocols like SMTP and IMAP use TCP so that messages arrive complete.
- Financial transactions and critical data: banking, payments, and database connections, where data integrity is paramount.
- General client-server APIs: TCP (with TLS) is the default for request/response APIs because it handles retransmission and ordering for you.

**Use UDP** when you need fast, lightweight communication and can tolerate some loss or reordering:

- Real-time media: video conferencing and VoIP use UDP (via RTP/WebRTC) to keep delay low; a few lost frames or a brief drop in quality are acceptable. (Note that most video streaming to viewers, e.g. HLS or DASH, actually runs over HTTP, i.e. TCP or QUIC, because buffering hides the latency.)
- Online gaming: games often send player actions and state over UDP. If a state update is lost, the next one corrects the state, so the game keeps going rather than freezing.
- DNS and other small queries: it is faster to send a query over UDP and retry on timeout than to open a TCP connection for each lookup.
- Broadcast and multicast: LAN discovery protocols and some IoT use cases send data to many recipients at once, which only UDP supports.

**Use QUIC** when you want TCP's reliability with better performance, especially on the modern web and on mobile:

- HTTP/3 and web services: sites and APIs served over HTTP/3 use QUIC, which reduces handshake time and improves load times on high-latency or lossy networks (like mobile networks).
- Mobile apps on changing networks: connection migration lets transfers survive switches between Wi-Fi and cellular.
- Applications with many parallel streams: QUIC multiplexes streams without head-of-line blocking.
- When encryption should be mandatory: QUIC is always encrypted.

QUIC is still newer than TCP and UDP, so using it directly means relying on a QUIC library or an HTTP/3 stack on both client and server, and having a fallback for networks that block UDP.

Many applications combine protocols. A video conferencing app, for example, typically sends audio and video over UDP for responsiveness, falls back to TCP when UDP is blocked, and uses HTTPS for chat messages and file transfers, where nothing may be lost.

## Summary

::: {.callout-note title="Summary"}
TCP, UDP, and QUIC serve different needs. TCP is the reliable workhorse for when accuracy and ordering matter more than speed: it is connection-oriented, retransmits lost data, and controls congestion, at the cost of handshake latency and head-of-line blocking. UDP is the lightweight courier for when speed matters and occasional loss is acceptable: it is connectionless, has no delivery guarantees or congestion control, and supports multicast. QUIC is the modern hybrid: built on UDP, it provides reliable, encrypted, multiplexed streams with a faster handshake, no head-of-line blocking between streams, and connection migration, which makes it the transport of HTTP/3. Choosing between them depends on the application's needs (real-time vs. bulk data), network conditions, and client support.
:::

## References

1. [RFC 9293: Transmission Control Protocol](https://www.rfc-editor.org/rfc/rfc9293)
2. [RFC 768: User Datagram Protocol](https://www.rfc-editor.org/rfc/rfc768)
3. [RFC 9000: QUIC: A UDP-Based Multiplexed and Secure Transport](https://www.rfc-editor.org/rfc/rfc9000)
4. Roberto Vitillo, _Understanding Distributed Systems: What every developer should know about large distributed applications_
