# ADR 0001: Choose Message Digest Algorithm for Checksum Verification

- Status: Accepted
- Date: 2025-11-29

## Context

The application must generate a checksum for a unique data string (includes
first and last name) and expose it via a REST endpoint. The checksum must
minimize collision risk so that a published public key can be reliably verified
by clients. Java's `MessageDigest` supports multiple algorithms (for example,
MD5, SHA-1, SHA-256, SHA-512), and the course requirements call for a
collision-resistant choice.

## Decision

Use SHA-256 as the checksum algorithm. In code, this is captured in the
`HASH_ALGORITHM` constant (`"SHA-256"`) and passed to
`MessageDigest.getInstance(HASH_ALGORITHM)`. SHA-256 offers strong collision
resistance, is widely supported in Java, and balances security with
performance. We considered SHA-512 for a larger security margin but rejected it
to avoid longer hashes and higher compute cost that are unnecessary for this
use case. MD5 and SHA-1 are excluded due to known collision vulnerabilities.

## Consequences

- Produces 256-bit hashes suitable for integrity verification of the public key
  download.
- Keeps compatibility and performance good across typical Java deployments.
- If future requirements demand a larger margin, SHA-512 can be substituted
  with minimal code changes.
- Implementation details:
  - Uses UTF-8 to turn the data string into bytes, then calls
    `MessageDigest.digest` and converts the result to hex with `bytesToHex`.
  - The `/hash` endpoint returns the input data, algorithm name, and checksum.
  - `NoSuchAlgorithmException` remains declared on the handler method; if the
    algorithm name ever changes to an unsupported value, the runtime will
    surface that error instead of silently degrading security.

## References

- Assignment requirements:
  `doc/requirements/checkSum-Verification/requirements.md`
- Recommendation text: `doc/templates/ChecksumVerificationTemplate.md`
- Implementation: `src/main/java/com/snhu/sslserver/ServerApplication.java`
  (`/hash` endpoint using SHA-256)
