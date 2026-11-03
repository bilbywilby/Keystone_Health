# Keystone Health

Single-source compliance and reference framework for Pennsylvania health insurance operations, with emphasis on auditability, evidence handling, and regulatory traceability.

This repository is intended as a governance and compliance reference for healthcare billing, collection, PHI handling, and operational controls. It is not legal advice and should be reviewed by counsel before implementation in regulated workflows.

## Purpose

Keystone Health consolidates the core elements needed to operate in a compliance-aware environment:

- Regulatory reference tracking
- Retention and evidence controls
- Audit structure and assertions
- Deployment and rollback guidance
- Verification and governance workflow

## Scope

The framework is designed for:

- U.S. federal regulatory references
- Commonwealth of Pennsylvania-specific requirements
- Operational controls relevant to healthcare billing, collections, and PHI handling
- Evidence and auditability practices aligned with internal governance

## Repository structure

```text
Keystone_Health/
├── README.md
├── LICENSE
├── CHANGELOG.md
├── CODEOWNERS
├── .linkcheckignore
├── .github/
│   └── workflows/
├── schema/
│   ├── quarterly_audit.schema.json
│   ├── evidence_record.schema.json
│   └── retention_class.schema.json
├── audit/
│   ├── quarterly_audit.yaml
│   ├── validate_audit.py
│   └── fixtures/
├── references/
│   ├── REFERENCES.csv
│   ├── references.yaml
│   └── retention_classes.yaml
├── compliance/
│   ├── matrix/
│   ├── evidence/
│   └── mappings/
├── runbook/
│   ├── DEPLOYMENT_RUNBOOK.md
│   ├── ROLLBACK_MATRIX.md
│   └── COMMUNICATION_PLAN.md
├── docs/
│   ├── verification-log.md
│   ├── open-items.md
│   ├── glossary.md
│   ├── case-law/
│   └── adr/
├── tools/
│   ├── hash_chain_verify.py
│   ├── retention_migrate.py
│   ├── expiry_diff.py
│   └── tsa_verify.py
└── .github/workflows/
```

## How to use this repository

1. Start with the audit and reference materials in `audit/` and `references/`.
2. Review the retention classification guidance in `references/retention_classes.yaml`.
3. Use the deployment and rollback guidance in `runbook/` before production changes.
4. Validate schema-driven artifacts with the audit tooling.
5. Treat the repo as the control documentation and decision trail, not as a substitute for legal review.

## Citation and governance model

Every citation in the framework carries a `basis` designation:

- `law` — traced to a statute or regulation
- `secondary` — useful context that should be confirmed against primary authority
- `internal_design` — a control decision, not a legal mandate
- `best_practice` — advisory guidance only

Important: citations marked as unverified have not been confirmed against primary source text. Those items should not be relied on for audit execution without legal review.

## Audit and validation

The repo includes a structured audit model and validation workflow.

Example validation command:

```bash
python audit/validate_audit.py \
  --schema schema/quarterly_audit.schema.json \
  --input audit/quarterly_audit.yaml
```

The validation layer is intended to catch structural issues, missing fields, and schema mismatches before changes are promoted.

## Evidence handling

Evidence is intentionally not stored directly in this repository. The repository holds the framework for identifying, classifying, and verifying evidence while preserving the separation between operational evidence and versioned governance documentation.

This is by design to preserve:

- chain-of-custody integrity
- evidence immutability
- separation of source-of-truth metadata from mutable operational artifacts

## Retention principles

Retention logic is documented in the reference and schema materials and is organized around clear trigger events such as:

- last effective date
- determination date
- creation date
- collection activity date

The framework is designed to prevent common retention mistakes, especially around statutory clocks and superseded record lifecycles.

## Fail-stop and operational controls

The repo includes operational safeguards for high-risk scenarios such as:

- hash-chain mismatches
- missing timestamp validation
- retention classification errors
- plaintext PHI handling
- invalid benchmark or IDR-associated data

These controls are intended to stop automated processing when integrity or legal-compliance conditions are not met.

## Documentation set

Relevant materials include:

- `docs/verification-log.md` — verification status and checks performed
- `docs/open-items.md` — unresolved review items
- `docs/glossary.md` — terminology and definitions
- `docs/adr/` — architecture and governance decisions
- `runbook/DEPLOYMENT_RUNBOOK.md` — deployment, promotion, and rollback procedures
- `runbook/ROLLBACK_MATRIX.md` — emergency rollback decision table

## Governance notes

This repository should be treated as a compliance reference framework, not as a source of legal interpretation. It is most useful when paired with:

- legal review of binding authority
- evidence and records management procedures
- internal policy approval and ownership
- operational controls and monitoring

## Contributing

Contributions should be made with the same rigor expected of regulated documentation:

- preserve traceability to authority
- distinguish legal requirement from internal design
- avoid unverified claims
- update validation and governance artifacts when changing structure or assertions

## License

This project is licensed under the MIT License. See `LICENSE` for details.

## Status

This repository is positioned as a structured compliance framework and audit reference for Pennsylvania healthcare operations. It is intended to support formal review, documentation control, and governance alignment, not to substitute for legal or regulatory analysis.
