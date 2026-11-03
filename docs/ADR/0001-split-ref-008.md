# ADR-0001: Separate HIPAA Requirements from NIST Guidance

- **Status:** Proposed — approval and legal review pending
- **Date:** 2026-11-03
- **Decision owners:** Compliance Architecture; Legal Counsel review required
- **Scope:** Keystone Health Forensic Navigator Compliance Reference Framework

## Context

The reference matrix needs to distinguish binding legal requirements from non-binding technical guidance. Earlier descriptions grouped HIPAA requirements and NIST material under a compound `REF-008`, which could make it unclear whether an assertion was required by law, informed by guidance, or an internal control choice.

The current README identifies `REF-008A` as the HIPAA Security Rule and `REF-008B` as NIST SP 800-122, a guide concerning protection of personally identifiable information. This decision records that separation. It does not silently expand `REF-008B` to include NIST CSF 2.0 or SP 800-53; adding those instruments would require a separately reviewed reference entry and mapping.

## Decision drivers

- Keep legal obligations distinguishable from voluntary guidance and internal design choices.
- Allow assertions and audit artifacts to identify the authority they actually rely on.
- Avoid implying that a NIST publication is itself a HIPAA regulation or that every technical control is legally mandated.
- Keep reference IDs stable enough for validation and traceability.

## Options considered

1. **Keep one compound reference.** Simple, but obscures authority type and makes assertion mapping ambiguous.
2. **Use sub-tags inside one reference.** Preserves the parent ID, but depends on every consumer interpreting the internal sub-tags consistently.
3. **Maintain separate reference IDs.** Use `REF-008A` for HIPAA and `REF-008B` for the specific NIST publication recorded in the source index.

## Decision

Adopt option 3. Keep HIPAA requirements and NIST guidance in separate reference records. Each assertion must identify its applicable authority and basis (for example, `law`, `secondary`, `internal_design`, or `best_practice`) rather than treating the two records as interchangeable.

The title, scope, citation, and URL for `REF-008B` must match the specific NIST publication actually reviewed. Any addition of NIST CSF, SP 800-53, or another publication must be represented explicitly and reviewed before it is used to support an assertion.

## Consequences

### Positive

- Authority type is visible in the reference matrix and audit trail.
- HIPAA-specific assertions can be reviewed without conflating them with guidance mappings.
- Future NIST publications can be added without silently changing the meaning of an existing reference ID.

### Trade-offs

- Existing references and assertion mappings may need updates when IDs are split or clarified.
- The additional records require maintenance so titles, scope, basis, verification status, and source URLs stay aligned.

## Implementation and review

1. Keep `REF-008A` and `REF-008B` separate in the machine-readable reference files and human-readable index.
2. Review every assertion that cites either record; confirm that its stated basis and source support the assertion.
3. Update cross-references and validation fixtures when IDs change.
4. Obtain Compliance and Legal review before marking this ADR accepted or treating the mapping as a legal conclusion.

## Limitations

This ADR documents a reference-management decision; it is not legal advice and does not determine the applicability or interpretation of any law or guidance. Legal counsel must validate legal characterizations and assertion mappings.
