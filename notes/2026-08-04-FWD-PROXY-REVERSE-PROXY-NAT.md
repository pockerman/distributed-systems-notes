# Lesson 12: Forward Proxy, Reverse Proxy & NAT

## Overview

A forward proxy sits between clients and the internet and acts on the clients' behalf; a reverse proxy sits in front of servers and acts on the servers' behalf; and NAT (Network Address Translation) is a router function that transparently remaps IP addresses so that multiple devices can share a single public IP address.

All three involve an intermediary between clients and servers, but they serve very different purposes.
In simple terms, a forward proxy intermediates outgoing requests from clients, a reverse proxy intermediates incoming requests to servers, and NAT is a network-layer mechanism that rewrites IP addresses in transit.

## Forward proxy

A forward proxy (often just called a "proxy server") is an intermediary server that sits between client devices and the wider internet.
When a client, such as a web browser, wants to fetch a resource from the internet, it sends the request to the forward proxy.
The proxy forwards the request to the destination server on the client's behalf and returns the server's response to the client.
The client therefore never contacts the internet resources directly; the proxy does so for it.

### Use cases and benefits

Forward proxies are commonly used to control access and enhance privacy for clients:

- **Policy enforcement and filtering.** In a company or school network, all web traffic can be routed through a forward proxy, which filters content and enforces policies.
- **Bypassing restrictions.** A user in a region with restricted access might use a forward proxy to bypass geo-blocking or censorship, by having the proxy make the requests from an allowed location.
- **Caching.** Forward proxies can cache frequently accessed content to speed up responses and reduce bandwidth usage.
- **Anonymity.** The destination server sees the proxy's IP address instead of the client's, so a forward proxy hides the client's identity from the server (though the proxy itself knows it).

In summary, forward proxies act on behalf of clients, to enforce organizational policies, improve performance through caching, or provide privacy.

## Reverse proxy

A reverse proxy is essentially the opposite of a forward proxy.
Instead of representing the client, it acts on behalf of the server. It is deployed in front of one or more servers (often web servers), receives incoming requests from clients on the internet, and forwards them to the appropriate backend server.
To the client, the reverse proxy appears to be the web server itself; the client isn't aware that its requests are being handled by an intermediary.

### Purpose and use cases

Reverse proxies protect and optimize the server side.
Because clients never communicate directly with the origin servers, a reverse proxy adds a layer of security and performance. Common uses include:

- **Load balancing:** distributing incoming requests across multiple servers so that no single server is overwhelmed (see [Lesson 34: Load Balancing](2025-04-21-load-balancing.md)).
- **Caching:** serving cached responses for static or frequent requests, reducing the load on the backend.
- **TLS termination:** handling the encryption and decryption of HTTPS traffic, taking that work off the backend servers.
- **Security:** hiding the identity and structure of the backend; clients only know the proxy's address. This protects the actual servers from direct attacks, e.g. the reverse proxy can absorb DDoS traffic or block malicious requests.

NGINX, HAProxy, Envoy, and CDNs are common examples of reverse proxies.

## NAT (Network Address Translation)

Network Address Translation (NAT) is a networking technique, usually implemented on routers or firewalls, that translates private IP addresses to a public IP address (and back) as traffic passes through.
NAT operates at the network (IP) layer, rewriting the source or destination IP addresses (and, usually, ports) in packet headers in transit.
Its most common use is to let multiple devices on a private network (e.g. a home or office LAN with addresses like `192.168.x.x`) share a single public IP address when accessing the internet.
The NAT device (such as your Wi-Fi router) keeps track of the outbound connections of each internal device and makes sure that the responses coming back from the internet are routed to the right device.

### Why NAT is important

NAT was originally introduced as a workaround for the limited supply of IPv4 addresses.
Instead of every computer needing a unique public IP address, NAT lets you use private addresses internally and consume only one public address for the whole network.
For example, your home router likely has one public IP address from your ISP, but lets laptops, phones, and IoT devices, each with a private IP address, connect to the internet through it.
Besides conserving IPv4 addresses, NAT has a side effect that adds some protection: hosts on the internet cannot initiate a connection to an internal device unless the NAT is explicitly configured to allow it (port forwarding). NAT is not a firewall, though, and should not be relied on as one.
In essence, NAT hides an entire internal network behind one public IP address (also known as IP masquerading).

### How NAT works

When a device inside a NAT-enabled network sends traffic out, the NAT router replaces the source (private) IP address with its own public IP address, and typically assigns a unique source port to the connection.
The external server sees the router's public IP address and replies to it.
When the reply arrives, the NAT device looks up the destination port in its translation table, finds the internal device the connection belongs to, and rewrites the destination address back to that device's private IP address.
This translation is transparent to both the internal device and the external server; neither needs to know that NAT is happening.

## NAT vs proxy

NAT and proxies both involve intermediaries, but they are not the same:

- NAT operates at a lower level (the network layer) and transparently rewrites address information in packets.
It doesn't need any configuration in client applications; your computer just knows it can reach the internet via the router.
- A proxy operates at the application level and, in the case of a forward proxy, often requires clients to be configured to use it.
Proxies also understand and can manipulate higher-level protocols (e.g. an HTTP proxy can filter URLs or cache pages), whereas NAT only cares about IP addresses and ports, not the content of the traffic.

## Summary

::: {.callout-note title="Summary"}
Forward proxies, reverse proxies, and NAT all place an intermediary between clients and servers. A forward proxy acts on behalf of clients: it forwards their outgoing requests, enabling content filtering, caching, bypassing restrictions, and anonymity. A reverse proxy acts on behalf of servers: it receives incoming requests and forwards them to backends, enabling load balancing, caching, TLS termination, and protection of the backend. NAT works at the network layer, rewriting IP addresses and ports so that many devices with private addresses can share one public IP address; it conserves IPv4 addresses and blocks unsolicited inbound connections as a side effect. Unlike proxies, NAT is transparent to applications and does not look at the content of the traffic.
:::

## References

1. [Proxy server](https://en.wikipedia.org/wiki/Proxy_server)
2. [Reverse proxy](https://en.wikipedia.org/wiki/Reverse_proxy)
3. [RFC 3022: Traditional IP Network Address Translator (Traditional NAT)](https://www.rfc-editor.org/rfc/rfc3022)
