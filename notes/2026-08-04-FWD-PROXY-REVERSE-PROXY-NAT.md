# qubit-note: Distributed Systems | Networking | Forward Proxy, Reverse Proxy & NAT

## Overview

A forward proxy sits between a client and the internet to act on the client’s behalf, a reverse proxy sits in front of a server to act on the server’s behalf, and NAT (Network Address Translation) is a router function that transparently remaps IP addresses so multiple devices can share a single public IP address.

Forward proxies, reverse proxies, and NAT are all techniques that involve an intermediary between clients and servers on a network, but they serve very different purposes.

In simple terms, a forward proxy is a server that intermediates outgoing requests from a client, a reverse proxy intermediates incoming requests to a server, and NAT is a network-layer mechanism that modifies IP addresses in transit.


What is a Forward Proxy?
A forward proxy (often just called a “proxy server”) is an intermediary server that sits between client devices and the wider internet.

When a client (such as a web browser) wants to fetch a resource from the internet, it sends the request to the forward proxy first.

The proxy then forwards that request to the destination server on the client’s behalf and returns the server’s response back to the client.

In this way, the client does not directly contact the internet resources; the proxy does so for it.

Use Cases and Benefits
Forward proxies are commonly used to enhance privacy and control access for clients.

For example, in a company or school network, all web traffic might be routed through a forward proxy which can filter content and enforce policies.

If a user is in a region with restricted access, they might use a forward proxy to bypass geo-blocks or censorship by having the proxy make requests from an allowed location.


Forward proxies can also cache frequently accessed content to speed up responses and reduce bandwidth usage.

Another benefit is anonymity. Since the destination server sees the proxy’s IP address instead of the client’s, a forward proxy can hide the client’s identity (though the proxy itself knows it).

In summary, forward proxies act on behalf of clients to retrieve data, often to enforce organizational policies, improve performance through caching, or provide privacy.

What is a Reverse Proxy?
A reverse proxy is essentially the opposite of a forward proxy.

Instead of representing the client, a reverse proxy acts on behalf of the server. It is deployed in front of one or more servers (often web servers) and intercepts incoming requests from clients on the internet, then forwards those requests to the appropriate backend server.

To the external client, the reverse proxy itself appears to be the web server. The client isn’t aware that their requests are being handled by an intermediary.



Purpose and Use Cases
Reverse proxies are typically used to protect and optimize the server side of a network.

By ensuring clients do not communicate directly with the origin servers, a reverse proxy can add a layer of security and performance.

Common uses include: load balancing (distributing incoming requests across multiple servers so no single server is overwhelmed), caching of content (the reverse proxy can serve cached responses for static or frequent requests, reducing load on the backend), and SSL/TLS termination (the reverse proxy can handle encrypting/decrypting HTTPS traffic, taking that load off the backend servers).

Reverse proxies also help hide the identity and structure of your backend servers; clients only know the proxy’s address.

This can protect the actual servers from direct attacks (e.g., the reverse proxy can help absorb DDoS attacks or block malicious traffic).


What is NAT (Network Address Translation)?
Network Address Translation (NAT) is a networking technique (usually implemented on routers or firewalls) that translates private local IP addresses to a public IP address (and vice versa) as traffic passes through.

NAT operates at the IP layer (network layer), modifying the source or destination IP addresses in packet headers while in transit.

The most common use of NAT is to allow multiple devices on a private network (e.g. your home or office LAN with IPs like 192.168.x.x) to share a single public IP when accessing the internet.

The NAT device (such as your Wi-Fi router) keeps track of outbound requests from each internal device and ensures that the responses coming back from the internet get routed to the correct device on the private network.

Why NAT Is Important
NAT was originally introduced as a workaround for the limited supply of IPv4 addresses.

Instead of every computer needing a unique public IP address, NAT lets you use private IP addresses internally and only consumes one public IP for the whole network.

For example, your home router likely has one public IP from your ISP, but allows laptops, phones, and IoT devices in your home (each with a private IP) to all connect out to the internet through that one address.

NAT conserves IPv4 addresses and also adds a layer of privacy/security: external hosts on the internet cannot directly initiate a connection to an internal private IP unless the NAT is specifically configured to allow it (port forwarding).

In essence, NAT hides your entire internal network behind one public-facing IP (a form of IP masquerading).

How NAT Works
When a computer inside a NAT-enabled network sends traffic out, the NAT router replaces the source IP (the private IP) with its own public IP and typically assigns a unique source port to that session.

The external server on the internet sees the router’s public IP and replies to it.

When the reply comes back, the NAT device looks at the combination of port numbers and destination and figures out which internal IP to send the traffic to, then translates the destination address back to that internal IP.

This translation is usually seamless and transparent to both the internal device and the external server. Neither side needs to know NAT is happening.



NAT vs Proxy
It’s worth noting that NAT is not the same as a proxy, though they both involve intermediaries.

NAT operates at a lower level (IP network layer) and transparently rewrites address info in packets.

It doesn’t need any special configuration on client applications. Your computer just knows it can reach the internet via the router.

A proxy, on the other hand, operates at the application level and often requires the client to be aware of it (for a forward proxy, you must configure your browser or app to use the proxy’s address).

Also, proxies can understand and manipulate higher-level protocols (like an HTTP proxy can filter URLs or cache pages), whereas NAT simply cares about IPs and ports, not the content of the traffic.


