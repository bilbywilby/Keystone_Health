# ADR-0003: Preserve Evidence During Deployment and Rollback

- **Status:** Proposed — approval and legal review pending
- **Date:** 2026-11-03
- **Decision owners:** Systems Architecture; Security and Legal review required
- **Scope:** Deployment, rollback, migration, and recovery procedures affecting evidence records

## Context

A software rollback can restore an earlier application version, but it must not silently rewrite, delete, or truncate forensic records. Evidence is stored outside this Git repository; the repository contains procedures and pointers, not the evidence payloads. Application release state and evidence-store state therefore need separate operational controls.

Hash-linked records can help detect changes to a recorded chain. They do not, on their own, prove that all events were recorded, that a record is true, or that a record is legally admissible. Retention and disposition remain governed by the applicable approved policy, legal holds, and counsel review.

## Decision drivers

- Preserve the original records and chain-of-custody information during incidents.
- Permit application code to be rolled back independently of evidence data.
- Make corrections and recovery actions reviewable and attributable.
- Avoid treating a point-in-time restore or hash check as proof that no evidence was lost or altered.

## Options considered

1. **Roll back application and evidence store together.** Operationally simple, but risks removing or overwriting records created after the restore point.
2. **Restore evidence in place from a prior snapshot.** May overwrite later records and make the incident harder to investigate.
3. **Separate code rollback from evidence preservation.** Roll back executable artifacts where safe; preserve the evidence source and investigate data recovery separately.

## Decision

Adopt option 3 as the default:

- A code rollback must not run destructive evidence-store operations, including `DROP`, `DELETE`, or `TRUNCATE`, as part of the rollback path.
- Do not overwrite or truncate the evidence source to make it match an earlier application version.
- Record operational corrections as new, attributable entries that refer to the affected records where the system supports this. Preserve the originals.
- If integrity is uncertain, halt or restrict automated processing as appropriate, preserve the affected source, and investigate using controlled copies. Do not repair the only source in place.
- Recovery or restoration affecting evidence requires an approved, documented procedure, integrity checks, and review by the responsible security, compliance, and legal roles. A recovery copy is not a substitute for preserving the original source.
- Apply retention, legal-hold, and authorized disposition rules separately. Immutability during rollback does not mean indefinite retention or exemption from approved disposition requirements.

## Consequences

### Positive

- Application recovery is less likely to alter evidence records.
- Incident actions and corrections can be reviewed against preserved originals.
- Evidence preservation is treated as an explicit operational requirement rather than an incidental property of deployment tooling.

### Trade-offs

- Compatible application and evidence schemas may need to be maintained across releases.
- Recovery may require additional storage, access controls, coordination, and documented approvals.
- Incident response may take longer when integrity or provenance is uncertain.

## Implementation and review

1. Ensure rollback automation operates only on application artifacts unless a separately approved data-recovery procedure applies.
2. Document evidence-store access, preservation, legal-hold, and recovery responsibilities in the relevant runbooks.
3. Verify that integrity checks run against preserved data and that their outputs are retained as operational records.
4. Review restore, migration, and retention procedures with Security, Compliance, and Legal before production use.

## Limitations

This ADR establishes an engineering safeguard, not a legal conclusion about retention, admissibility, or compliance. Cryptographic integrity checks detect only the conditions they are designed to test and must be interpreted with the full chain-of-custody and system context.
