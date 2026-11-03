# ADR-0002: Atomic Citation Discipline — One Canonical URL per Source Row

- **Status:** Proposed — approval and legal review pending
- **Date:** 2026-11-03
- **Decision owners:** Compliance Architecture; source-index maintainers
- **Scope:** Reference indexes, compliance assertions, and citation-bearing artifacts
- **Affects:** `references/REFERENCES.csv`, `references/references.yaml`, README source index, `tools/check_links.py`

## Context

A single citation cell containing multiple URLs or authorities is difficult to verify consistently. Automated link checks may not know which URL corresponds to which authority, and reviewers may be unable to tell whether a status applies to every source in the cell or only one of them.

## Decision drivers

- Make each source independently reviewable and link-checkable.
- Keep authority, scope, verification status, and assertion mappings explicit.
- Avoid parser-dependent splitting of comma-separated citations.
- Support reproducible hashing: while RFC 8785 (JSON Canonicalization Scheme) guarantees deterministic serialization for a JSON data model that satisfies JCS constraints, it cannot repair structurally ambiguous content. Structural determinism must precede serialization determinism.

## Options considered

1. **Allow multiple sources in a row.** Compact, but makes attribution and verification status ambiguous.
2. **Store a delimited list in one field.** Easier to enter, but requires consistent parser implementation.
3. **Use an atomic source record for each authority and URL.** More rows, but each record has a single unambiguous source.

## Decision

Use option 3:

1. Each source-index row represents one authority or source and has one primary canonical URL.
2. When an assertion relies on multiple authorities, record each as a separate source row and link the assertion to all applicable source IDs.
3. Apply verification status to the individual source record. A reachable URL does not establish that the source supports an assertion or that counsel has approved the interpretation.

### Worked examples

**Split by authority.** REF-014 originally carried both the FDCPA and Regulation F in one field. It decomposes into `REF-014A` (15 U.S.C. § 1692) and `REF-014B` (12 CFR Part 1006), each with its own binding nature.

**Exception — one instrument, two sections.** If multiple sections are cited from the same instrument (e.g., 42 CFR § 414.22 and § 414.26), the row cites the **containing subpart** as its canonical URL and enumerates both sections in the citation text.

### Two independent status axes

To avoid conflating retrieval with legal agreement, the framework tracks provenance and interpretation separately.

**Axis 1 — Provenance (Retrieval Status):**

| Tag | Token | Meaning |
|---|---|---|
| ✅ | `primary_fetched` | Retrieved and read from the primary source |
| 🔎 | `search_surfaced` | Appeared in search results; not independently fetched |
| ⚠️ | `unverified` | Asserted but not confirmed against primary text |
| 🔒 | `internal` | Internal document; not externally verifiable |

**Axis 2 — Interpretation (Legal Status):**

| Value | Meaning |
|---|---|
| `settled` | Operative meaning not in dispute |
| `disputed` | Active disagreement among authorities or courts |
| `pending_counsel` | Awaiting legal review |

## Consequences

### Positive

- Each source can be checked and updated independently.
- Reviewers can see which authorities support an assertion and which remain unverified.
- Provenance and interpretation are independently queryable.

### Trade-offs

- Reference indexes contain more rows and require explicit cross-references.
- Decomposition requires judgment about where one authority ends and another begins.

## Implementation and review

1. Update reference data to represent one source per record.
2. Validate that each public source record has exactly one primary URL.
3. Keep link availability, source review, and legal interpretation as distinct review states.

## Limitations

Atomic rows improve traceability; they do not prove that a citation is current or sufficient to establish compliance. Those judgments require legal review.
