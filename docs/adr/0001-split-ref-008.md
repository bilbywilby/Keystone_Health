# ADR-0001: Separate HIPAA Security Rule Requirements from NIST SP 800-122 Guidance

- **Status:** Proposed — approval and legal review pending
- **Date:** 2026-11-03
- **Decision owners:** Compliance Architecture; Legal Counsel review required
- **Scope:** Keystone Health Forensic Navigator Compliance Reference Framework
- **Affects:** `references/REFERENCES.csv`, `references/references.yaml`, assertions Q4.1–Q4.5, evidence path `008A-HIPAA/`

## Context

The reference matrix needs to distinguish binding legal requirements from non-binding technical guidance. Earlier descriptions grouped HIPAA requirements and NIST material under a compound `REF-008`, which could make it unclear whether an assertion was required by law, informed by guidance, or an internal control choice.

The current README identifies `REF-008A` as the HIPAA Security Rule and `REF-008B` as NIST SP 800-122. This decision records that separation. It does not silently expand `REF-008B` to include NIST CSF 2.0 or SP 800-53; adding those instruments would require a separately reviewed reference entry and mapping.

## Decision drivers

- Keep legal obligations distinguishable from voluntary guidance and internal design choices.
- Allow assertions and audit artifacts to identify the authority they actually rely on.
- Avoid implying that a NIST publication is itself a HIPAA regulation.
- Keep reference IDs stable enough for validation and traceability.
- Enable the system to enforce precedence based on recorded priority (e.g., binding rules override advisory guidance) after applicability and conflict have been classified by an authorized reviewer.

## Options considered

1. **Keep one compound reference.** Simple, but obscures authority type and makes assertion mapping ambiguous.
2. **Use sub-tags inside one reference.** Preserves the parent ID, but depends on every consumer interpreting internal sub-tags consistently.
3. **Maintain separate reference IDs.** Use `REF-008A` for HIPAA and `REF-008B` for the specific NIST publication.

## Decision

Adopt option 3. Keep HIPAA requirements and NIST guidance in separate reference records. Each assertion must identify its applicable authority and basis (for example, `law`, `secondary`, `internal_design`, or `best_practice`) rather than treating the two records as interchangeable.

### Recorded scope of each reference

**REF-008A — HIPAA Security Rule.** 45 CFR Part 160 and Subparts A and C of Part 164. Administrative, physical, and technical safeguards for electronic protected health information (ePHI). Binding Federal Regulation.

**REF-008B — NIST SP 800-122**, *Guide to Protecting the Confidentiality of Personally Identifiable Information*. PII identification and confidentiality impact levels. Non-Binding Federal Guidance.

### Scope boundary — what REF-008A does not include

REF-008A covers the **Security Rule only**. It does **not** incorporate:

- the **HIPAA Privacy Rule** — 45 CFR Part 164 **Subpart E**; or
- the **HIPAA Breach Notification Rule** — 45 CFR Part 164 **Subpart D**.

Both were considered during scoping and deliberately excluded to avoid inflating scope beyond billing, collection, and ePHI-safeguard operations. If either is later admitted, it takes a new reference ID.

## Consequences

### Positive

- Authority type is visible in the reference matrix and audit trail.
- HIPAA-specific assertions can be reviewed without conflating them with guidance mappings.
- Future NIST publications can be added without changing the meaning of an existing reference ID.

### Trade-offs

- Existing references and assertion mappings may need updates when IDs are split.
- The additional records require maintenance to keep titles, scope, and URLs aligned.

## Implementation and review

1. Keep `REF-008A` and `REF-008B` separate in the machine-readable reference files.
2. Review every assertion that cites either record to confirm the stated basis supports the assertion.
3. Obtain Compliance and Legal review before marking this ADR accepted.

## Watch item

HHS published a proposed rule on **2025-01-06**, *HIPAA Security Rule to Strengthen the Cybersecurity of Electronic Protected Health Information*. As of the last verification, this remains a proposal; the existing Security Rule remains in effect. If finalized, the scope description of REF-008A will require revision.

## Limitations

This ADR documents a reference-management decision; it is not legal advice. Legal counsel must validate legal characterizations and assertion mappings.
