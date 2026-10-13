# Lesson 51: Checksums & ETags

::: {.callout-warning title="Work in progress"}
This lesson is incomplete.
:::

## Overview

Checksums are short digital fingerprints of data used to verify its integrity. ETags (entity tags) are identifiers, often derived from checksums or hashes, that web and storage systems use to track versions of a resource and detect changes.

## What is a checksum?

A checksum is a small value calculated from a block of data in order to detect errors or alterations. By recomputing the checksum later and comparing it with the original value, you can verify whether the data has changed.
If the checksums match, the data is very likely unaltered.

Checksums are widely used to ensure data integrity in storage and transmission, e.g. to make sure a file wasn't corrupted during a download, or to detect disk errors. They don't by themselves prove authenticity (i.e. who created the data); for that, you need a cryptographic signature or a MAC. But they are excellent at catching accidental corruption.

Checksum algorithms range from simple ones, like parity bits or sums of bytes, to error-detecting codes such as CRC32, to cryptographic hash functions such as SHA-256.
Cryptographic hash functions have the avalanche property: changing even a single bit of the input produces a completely different output. Simpler checksums don't have this property, but they are designed so that common errors, such as single-bit flips or burst errors in the case of CRCs, are guaranteed to change the checksum.

<!-- TODO: ETags (strong vs weak, If-None-Match / If-Match, conditional requests, optimistic concurrency), checksums in storage systems (e.g. S3 ETags, end-to-end checksums). -->

## References

1. [Checksum](https://en.wikipedia.org/wiki/Checksum)
2. [HTTP ETag](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/ETag), MDN Web Docs
