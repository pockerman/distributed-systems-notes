# Lesson 74: Design a URL Shortener

::: {.callout-warning title="Work in progress"}
This lesson has not been written yet. The outline below lists what it should cover.
:::

## Overview

A classic warm-up design: turn long URLs into short ones and redirect quickly at very high read volume.

## Outline

- Requirements and back-of-the-envelope estimates
- API design and 301 vs 302 redirects
- Generating short codes: hashing vs base-62 IDs
- Storage choice and data model
- Caching the hot links
- Analytics and abuse prevention

## Related lessons

- [Lesson 40: Design a Unique ID Generation System](../2025-08-14-unique-id-generation.md)
- [Lesson 16: Caching Methods](../2026-01-20-Caching.md)
- [Lesson 27: Partitioning](../2025-08-04-partitioning.md)

## References

1. Alex Xu, _System Design Interview – An Insider's Guide_
