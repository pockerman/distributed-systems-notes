# Lesson 73: Secrets Management and Encryption at Rest

::: {.callout-warning title="Work in progress"}
This lesson has not been written yet. The outline below lists what it should cover.
:::

## Overview

Services need passwords, API keys and certificates, and the data they store must stay private even if a disk is stolen. This lesson covers how to store and rotate secrets and how to encrypt data at rest.

## Outline

- Why secrets do not belong in code or environment files
- Secret stores (HashiCorp Vault, cloud secret managers, Kubernetes secrets)
- Dynamic secrets and rotation
- Encryption at rest: disk, database and application level
- Envelope encryption and key management services (KMS)
- Workload identity instead of long-lived credentials

## Related lessons

- [Lesson 7: TLS 101](../2026-04-06-Part-5-TLS-101.md)
- [Lesson 47: TLS vs mTLS](2026-07-25-TLS-mTLS.md)
- [Lesson 38: Authentication & Authorization for APIs](../2026-10-06-AUTHENTICATION-AUTHORIZATION.md)

## References

1. Sam Newman, _Building Microservices_, O'Reilly Media
