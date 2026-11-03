# ADR-0002: Atomic Citation Rows

- **Status:** Proposed — approval and legal review pending
- **Date:** 2026-11-03
- **Decision owners:** Compliance Architecture; source-index maintainers
- **Scope:** Reference indexes, compliance assertions, and citation-bearing artifacts

## Context

A single citation cell containing multiple URLs or authorities is difficult to verify consistently. Automated link checks may not know which URL corresponds to which authority, and reviewers may be unable to tell whether a status applies to every source in the cell or only one of them.

The framework needs traceable citations while preserving distinctions among statutes, regulations, agency material, secondary sources, and internal design choices.

## Decision drivers

- Make each source independently reviewable and link-checkable.
- Keep authority, scope, verification status, and assertion mappings explicit.
- Avoid parser-dependent splitting of comma-separated citations.
- Preserve the distinction between source verification and legal approval.

## Options considered

1. **Allow multiple sources in a row.** Compact, but makes attribution and verification status ambiguous.
2. **Store a delimited list in one field.** Easier to enter, but requires every consumer to parse the same convention correctly.
3. **Use an atomic source record for each authority and URL.** More rows, but each record has a single unambiguous source.

## Decision

Use option 3:

1. Each source-index row represents one authority or source and has one primary canonical URL.
2. When an assertion relies on multiple authorities, record each authority as a separate source row and link the assertion to all applicable source IDs.
3. Apply verification status to the individual source record. A reachable URL does not establish that the source supports an assertion or that counsel has approved the interpretation.
4. If a source has no suitable public URL, identify its access or confidentiality status explicitly; do not insert an unrelated URL to satisfy the format.
5. Keep citation text, authority type, source scope, basis, and verification notes structured separately where the schema supports those fields.

## Consequences

### Positive

- Each source can be checked and updated independently.
- Reviewers can see which authorities support an assertion and which remain unverified.
- Parsers need not infer multiple citations from punctuation within a field.

### Trade-offs

- Reference indexes may contain more rows and require explicit cross-references.
- Maintainers must prevent duplicate records and keep source IDs stable or update dependent mappings when IDs change.

## Implementation and review

1. Update reference data and schemas to represent one source per record.
2. Validate that each public source record has exactly one primary URL and that assertion references resolve to known source IDs.
3. Keep link availability, source review, and legal interpretation as distinct review states.
4. Review exceptions (restricted, offline, or authoritative sources without a stable URL) with the document owner; record the reason without weakening traceability.

## Limitations

Atomic rows improve traceability and automation; they do not prove that a citation is current, authoritative, correctly interpreted, or sufficient to establish compliance. Those judgments require appropriate subject-matter and legal review.
