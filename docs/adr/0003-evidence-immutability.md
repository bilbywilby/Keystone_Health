# ADR-0003: Runtime-Neutral Evidence Immutability

- **Status:** Proposed — approval and legal review pending
- **Date:** 2026-11-03
- **Deciders:** Principal Systems Architect, Lead Security Engineer
- **Scope:** Audit logging, evidence storage, and integrity verification
- **Affects:** `tools/hash_chain_verify.py`, `runbook/ROLLBACK_MATRIX.md`

## Context

Audit logs and compliance snapshot state must be protected from retroactivity or platform lock-in. Storage-level immutability dependent on single-vendor cloud solutions (e.g., AWS S3 Object Lock) provides insufficient proof across multi-cloud environments and does not provide a verifiable chain of custody independent of the provider's API.

## Decision

We establish a **Runtime-Neutral Immutability Architecture** leveraging cryptographic hash chains (SHA-256) combined with RFC 3161 Time-Stamp Protocol tokens.

### Technical Controls

1. **Hash Chain Construction**: Every compliance event record $E_n$ is computed as:
   $$H_n = \text{SHA-256}(H_{n-1} \parallel \text{Payload}_n \parallel T_n)$$
   where $H_{n-1}$ is the previous hash and $T_n$ is the RFC 3161 timestamp token.
2. **WORM Storage Neutrality**: Raw log payloads and hash signatures are committed to Write-Once-Read-Many (WORM) storage abstractions. The verification logic resides in the application layer, making the proof portable across any storage medium.
3. **Verification**: Continuous CI integrity routines (`tools/hash_chain_verify.py`) audit chain continuity. Any break in the chain triggers an immediate FS1 failure state.

## Consequences

### Positive

- Integrity proof is independent of the cloud provider's proprietary locking mechanism.
- Mathematical proof of sequence prevents "gap" attacks where records are deleted from the middle of a log.
- RFC 3161 timestamps provide trusted third-party proof of existence at a specific time.

### Trade-offs

- Increased complexity in the logging pipeline to manage hash state.
- Dependency on a Time Stamping Authority (TSA) endpoint, requiring a secondary failover strategy (see Rollback Matrix FS2).

## Implementation and review

1. Implement the hash chain logic in the core audit stream.
2. Integrate the TSA endpoint and verify token validity in the deployment runbook (Phase 3).
3. Set up hourly CI verification jobs via `.github/workflows/verify-hash-chain.yml`.
