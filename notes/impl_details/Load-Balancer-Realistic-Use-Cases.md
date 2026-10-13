# Load Balancer Use Cases

Load balancers are dynamic and adaptable components that serve many purposes in managing network traffic and server workloads. See [Load Balancing](../2025-04-21-load-balancing.md) for an introduction.

Some common use cases are:

- **Failure handling:** automatically redirect traffic away from malfunctioning instances, to keep the service running and reduce interruptions.
- **Instance health checks:** continuously check the instances and send requests only to those that are healthy (see [Liveness and Readiness Checks](../2026-05-13-Liveness-Readiness-Checks.md)).
- **Platform-specific routing:** route requests from different device types (e.g. mobile or desktop) to specialized backends that tailor the response to the platform.
- **TLS termination:** handle the encryption and decryption of TLS (SSL) traffic, reducing the processing burden on the backend servers.
- **Cross-zone load balancing:** distribute traffic across availability zones or regions, increasing the system's resilience and its capacity to handle large volumes of requests.
- **User stickiness:** route the requests of a given user to the same backend server to preserve session state (see [Sticky Sessions](../2026-05-11-Sticky-Sessions.md)).
