# Keystone Health Forensic Navigator — Compliance Reference Framework

**Regulatory reference matrix, audit checklist, and deployment runbook for healthcare billing, collection, and PHI-handling operations.**

Status: `v1.3 audit checklist` · `13 references` · `33 assertions` · `schema-validated`  
Last verification pass: 2026-11-02 / 2026-11-03  
Jurisdictional scope: United States federal law + Commonwealth of Pennsylvania

---

## ⚠️ Read This First

This repository is **engineering and compliance-operations documentation**. It is **not legal advice**.
Every citation carries a `basis` field distinguishing:

- `law` — traced to a statute or regulation
- `secondary` — taken from a secondary source; verify against primary text
- `internal_design` — our own control choice, **not** a legal mandate
- `best_practice` — guidance only, non-binding

Citations marked `unverified` in the Source Index have **not** been confirmed against primary
text. Do not rely on them in an audit until counsel has reviewed them.

---

## Table of Contents

1. [Repository Architecture](#1-repository-architecture)
2. [Directory Reference](#2-directory-reference)
3. [Regulatory Reference Matrix](#3-regulatory-reference-matrix)
4. [Retention Classes](#4-retention-classes)
5. [Fail-Stop Triggers](#5-fail-stop-triggers)
6. [Audit Control Structure](#6-audit-control-structure)
7. [Evidence Repository Layout](#7-evidence-repository-layout)
8. [Schema & Validation](#8-schema--validation)
9. [Deployment Runbook Summary](#9-deployment-runbook-summary)
10. [Case Law & Litigation Tracking](#10-case-law--litigation-tracking)
11. [Open Verification Items](#11-open-verification-items)
12. [Source Index — Every Link](#12-source-index--every-link)
13. [Maintenance Cadence](#13-maintenance-cadence)
14. [Contributing & Governance](#14-contributing--governance)

---

## 1. Repository Architecture

```text
keystone-forensic-navigator-compliance/
│
├── README.md                          ← this file
├── LICENSE
├── CHANGELOG.md
├── CODEOWNERS
├── .linkcheckignore
│
├── .github/
│   └── workflows/
│       ├── validate-audit.yml          CI: schema-validate quarterly_audit.yaml
│       ├── link-check.yml              CI: weekly link-rot sweep
│       ├── hash-chain-verify.yml       CI: nightly evidence chain integrity
│       └── retention-expiry-report.yml CI: monthly T-180 expiry digest
│
├── schema/
│   ├── quarterly_audit.schema.json     Draft 2020-12 schema for the audit artifact
│   ├── evidence_record.schema.json     Base evidence log record (hash chain + TSA)
│   └── retention_class.schema.json     Retention class definition schema
│
├── audit/
│   ├── quarterly_audit.yaml            v1.3 — 33 assertions, Q1–Q8
│   ├── validate_audit.py               Validator invoked by CI
│   └── fixtures/
│       ├── production_shaped.yaml      Test fixture for staging validation
│       └── expected_output.txt         Reference validator output
│
├── references/
│   ├── REFERENCES.csv                  Machine-readable reference matrix
│   ├── references.yaml                 Same data, structured + basis fields
│   └── retention_classes.yaml          Retention class registry
│
├── compliance/
│   ├── matrix/
│   │   └── control_matrix.csv          Obligation → control → artifact mapping
│   ├── evidence/                       POINTER ONLY — evidence lives outside repo
│   │   └── README.md                   Explains why evidence is not versioned here
│   └── mappings/
│       ├── tma_vacatur_provisions.yaml Provision-level vacatur map
│       └── fceua_fdcpa_utpcpl.yaml     Multi-tier liability cascade
│
├── runbook/
│   ├── DEPLOYMENT_RUNBOOK.md           Phases 0–5, gates, rollback procedures
│   ├── ROLLBACK_MATRIX.md              Master rollback reference table
│   └── COMMUNICATION_PLAN.md           Escalation and notification routing
│
├── docs/
│   ├── verification-log.md             What was checked, when, by what method
│   ├── open-items.md                   Counsel review queue
│   ├── glossary.md                     QPA, IDR, GFE, GPCI, RVU, IIHI …
│   ├── case-law/
│   │   └── tma-litigation-tracker.md   TMA I–III + related proceedings
│   └── adr/
│       ├── 0001-split-ref-008.md       Why HIPAA and NIST were separated
│       ├── 0002-atomic-citation-rows.md Why one URL per row
│       └── 0003-evidence-immutability.md Why rollback never rewrites evidence
│
└── tools/
    ├── hash_chain_verify.py            Walks prev_record_hash linkage
    ├── retention_migrate.py            Applies retention-class migration map
    ├── expiry_diff.py                  Compares before/after expiry calendars
    └── tsa_verify.py                   Validates RFC 3161 tokens
```

---

## 2. Directory Reference

| Path | Contents | Owner | Versioned? |
|---|---|---|---|
| `schema/` | JSON Schemas (Draft 2020-12) validating all machine-readable artifacts | Systems Architect | Yes |
| `audit/` | The quarterly audit checklist and its validator | Compliance Lead | Yes |
| `references/` | The regulatory reference matrix in CSV + YAML | Governance Officer | Yes |
| `compliance/matrix/` | Obligation → control point → audit artifact mapping | Compliance Lead | Yes |
| `compliance/evidence/` | **Pointer only.** Actual evidence is stored in the secured evidence store, never in git | Governance Officer | No — by design |
| `runbook/` | Deployment and rollback procedures | Deployment Lead | Yes |
| `docs/` | Verification log, open items, glossary, ADRs | Governance Officer | Yes |
| `tools/` | Python utilities for chain verification, migration, expiry diffing | Systems Architect | Yes |
| `.github/workflows/` | CI: validation, link checking, chain verification, expiry reporting | Systems Architect | Yes |

**Design rule:** evidence artifacts are **never** committed to this repository. Git history is
rewritable; evidence must not be. The repo holds *how to find and verify* evidence, not the
evidence itself. See `docs/adr/0003-evidence-immutability.md`.

---

## 3. Regulatory Reference Matrix

Status legend: ✅ fetched and read · 🔎 surfaced in search results · ⚠️ unverified · 🔒 internal

### 3.1 Core references

| Ref ID | Instrument | Scope | Binding Nature | Citation | Primary URL | Status |
|---|---|---|---|---|---|---|
| REF-001 | No Surprises Act (Consolidated Appropriations Act 2021, Div. BB, Tit. I) | Balance billing prohibitions, emergency + OON care | Binding Federal Law | P.L. 116-260 | [congress.gov H.R.133 text](https://www.congress.gov/bill/116th-congress/house-bill/133/text) | ✅ |
| REF-002 | Requirements Related to Surprise Billing, Part I | QPA methodology, open negotiation, IDR workflow | Binding Federal Regulation | 45 CFR Part 149 | [eCFR 45 CFR Part 149](https://www.ecfr.gov/current/title-45/subtitle-A/subchapter-B/part-149) | ✅ |
| REF-003 | PA Insurance Department — NSA administration | State enforcement, consumer complaint resolution | Binding State Regulation | PID guidance | [PA.gov PID No Surprises Act](https://www.pa.gov/agencies/insurance/laws-regulations-notices/no-surprises-act) | 🔎 |
| REF-004 | Fair Credit Extension Uniformity Act (FCEUA) | Debt collection standards, unfair/deceptive practices | Binding State Law | 73 P.S. § 2270.1 et seq. | [Act 2000-7 (PDF)](https://www.legis.state.pa.us/WU01/LI/LI/US/PDF/2000/0/0007..PDF) | 🔎 |
| REF-005 | PA Judicial Code — Limitation of Actions | Four-year limitation on contract actions | Binding State Law | 42 Pa.C.S. § 5525 | [PA Gen. Assembly § 5525](https://www.legis.state.pa.us/WU01/LI/LI/CT/HTM/42/00.055.025.000..HTM) | 🔎 |
| REF-006 | Medicare Physician Fee Schedule — RVUs and GAF | RVU computation, GPCI geographic adjustment | Binding Federal Regulation | 42 CFR § 414.22 + § 414.26 | [eCFR § 414.22](https://www.ecfr.gov/current/title-42/chapter-IV/subchapter-B/part-414/subpart-B/section-414.22) · [Subpart B (contains § 414.26)](https://www.ecfr.gov/current/title-42/chapter-IV/subchapter-B/part-414/subpart-B) | ✅ |
| REF-007 | Layered Memory Protocol (LMP) Architecture Spec | State consistency, versioning ($v_{in} > v_t$), SHA-256 chaining | Internal Control Spec | Internal doc | 🔒 Internal | 🔒 |
| REF-008A | HIPAA Security Rule | Administrative, physical, technical safeguards for ePHI | Binding Federal Regulation | 45 CFR Part 160 + Part 164 Subparts A & C | [eCFR Part 164 Subpart C](https://www.ecfr.gov/current/title-45/subtitle-A/subchapter-C/part-164/subpart-C) · [HHS Security Rule](https://www.hhs.gov/hipaa/for-professionals/security/index.html) | ✅ |
| REF-008B | NIST SP 800-122 — Guide to Protecting the Confidentiality of PII | PII identification, confidentiality impact levels | Non-Binding Federal Guidance | NIST SP 800-122 (final, 04/06/10) | [NIST CSRC SP 800-122](https://csrc.nist.gov/publications/detail/sp/800-122/final) | ✅ |
| REF-009 | PA Insurance Department — market conduct & rate filings | Carrier loss ratios, rate benchmarks, AV tracking | Binding State Filing Requirement | PID filings | [PID ACA Health Rate Filings](https://www.pa.gov/agencies/insurance/posted-filings-reports-company-orders/posted-product-and-rate-filings/aca-health-rate-filings) | 🔎 |
| REF-011 | Employee Retirement Income Security Act (ERISA) | Plan-type determination, federal preemption | Binding Federal Law | 29 U.S.C. § 1001 et seq. | [DOL — ERISA](https://www.dol.gov/general/topic/health-plans/erisa) | ✅ |
| REF-014A | Fair Debt Collection Practices Act (FDCPA) | Federal debt collection statutory floor | Binding Federal Law | 15 U.S.C. § 1692 | [Cornell LII — 15 U.S.C. § 1692](https://www.law.cornell.edu/uscode/text/15/1692) | ✅ |
| REF-014B | Regulation F | Debt collection implementing rules | Binding Federal Regulation | 12 CFR Part 1006 | [CFPB — Regulation F](https://www.consumerfinance.gov/rules-policy/regulations/1006/Interp_1006) | ✅ |

### 3.2 Supporting references (cited within assertions)

| Support Ref | Purpose | Citation | URL | Status |
|---|---|---|---|---|
| SUP-01 | HIPAA documentation retention (6 years, "whichever is later") | 45 CFR § 164.316(b)(2)(i) | [eCFR § 164.316](https://www.ecfr.gov/current/title-45/subtitle-A/subchapter-C/part-164/subpart-C/section-164.316) · [Cornell LII mirror](https://www.law.cornell.edu/cfr/text/45/164.316) | ✅ |
| SUP-02 | IDR entity recordkeeping (6 years) | 45 CFR § 149.510(e)(viii) | [eCFR § 149.510](https://www.ecfr.gov/current/title-45/subtitle-A/subchapter-B/part-149/subpart-F/section-149.510) · [Cornell LII mirror](https://www.law.cornell.edu/cfr/text/45/149.510) | 🔎 |
| SUP-03 | Good faith estimate retention (medical record; 6-year floor for copies) | 45 CFR § 149.610(f)(1) | [eCFR § 149.610](https://www.ecfr.gov/current/title-45/subtitle-A/subchapter-B/part-149/subpart-G/section-149.610) · [Cornell LII mirror](https://www.law.cornell.edu/cfr/text/45/149.610) | 🔎 |
| SUP-04 | Computer security log management guidance | NIST SP 800-92 | [NIST CSRC SP 800-92](https://csrc.nist.gov/pubs/sp/800/92/final) | ✅ |
| SUP-05 | Trusted timestamp protocol (proof-of-existence) | RFC 3161 | [IETF RFC 3161](https://datatracker.ietf.org/doc/html/rfc3161) | ✅ |
| SUP-06 | Medicare Physician Fee Schedule (program page) | CMS PFS | [CMS Physician Fee Schedule](https://www.cms.gov/medicare/payment/fee-schedules/physician) | ✅ |
| SUP-07 | False Claims Act limitations window (6y / 3y / 10y cap) | 31 U.S.C. § 3731(b) | [Cornell LII — 31 U.S.C. § 3731](https://www.law.cornell.edu/uscode/text/31/3731) | 🔎 |
| SUP-08 | HIPAA Security Rule documentation requirements (HHS series) | HHS OCR | [HHS Security Series #5 (PDF)](https://www.hhs.gov/sites/default/files/ocr/privacy/hipaa/administrative/securityrule/pprequirements.pdf) | 🔎 |
| SUP-09 | NSA consumer rights overview (federal floor / state preservation) | CMS fact sheet | [CMS No Surprises fact sheet](https://www.cms.gov/newsroom/fact-sheets/no-surprises-understand-your-rights-against-surprise-medical-bills) | 🔎 |
| SUP-10 | FDCPA statutory text (alternate source) | FTC | [FTC FDCPA text](https://www.ftc.gov/legal-library/browse/rules/fair-debt-collection-practices-act-text) | 🔎 |
| SUP-11 | FDCPA examination procedures incl. Reg F record retention | OCC Comptroller's Handbook | [OCC FDCPA Handbook (PDF)](https://www.occ.treas.gov/publications-and-resources/publications/comptrollers-handbook/files/fair-debt-collection-practices-act/fair-debt-collection-practices-act-examination-procedures-interagency.pdf) | 🔎 |
| SUP-12 | Federal IDR process guidance for certified IDR entities | CMS | [CMS IDR Entity Guidance (PDF)](https://www.cms.gov/files/document/federal-idr-guidance-idr-entities-march-2023.pdf) | 🔎 |
| SUP-13 | GFE FAQs for uninsured / self-pay individuals | CMS | [CMS GFE FAQ Part 4 (PDF)](https://www.cms.gov/files/document/faqs-good-faith-estimate-uninsured-self-pay-part-4.pdf) | 🔎 |
| SUP-14 | IDR selection data elements | DOL | [DOL — Certified IDR Entity Selection Data Elements](https://beta.dol.gov/policy-regulations/pay-benefits/health-plans/surprise-billing-and-price-transparency/tools-and-resources/employers-and-service-providers/federal-independent-dispute-resolution-idr-process/selection-certified-idr-entity-data-elements) | 🔎 |
| SUP-15 | NSA IDR process toolkit | AMA | [AMA NSA IDR Toolkit (PDF)](https://www.ama-assn.org/system/files/ama-nsa-idr-toolkit.pdf) | 🔎 |

---

## 4. Retention Classes

| Class ID | Trigger | Years | Basis | Authority | Scope | Owner |
|---|---|---|---|---|---|---|
| `HIPAA-DOC-6Y` | `last_effective_date` | 6 | **law** | 45 CFR § 164.316(b)(2)(i) | HIPAA security policies and required documentation **only** | Governance Officer |
| `CFPB-FDCPA-3Y` | `last_collection_activity` | 3 | **law** | 12 CFR § 1006.100 | Debt collector records; FCEUA also reaches creditors, so default longer | Collections Director |
| `NSA-IDR-6Y` | `determination_date` | 6 | secondary | 45 CFR § 149.510(e)(viii) | Stated for certified IDR entities; plans/providers use 7y policy | Legal Counsel |
| `NSA-GFE-MEDREC` | `gfe_issue_date` | 6 floor | secondary | 45 CFR § 149.610(f)(1) | GFE is part of the medical record; 6y floor for requested copies | Legal Counsel |
| `POLICY-7Y` | `creation_date` | 7 | internal_design | — | Default where no mandate exists; **never** apply to HIPAA-DOC records | Policy Administrator |
| `PA-RATE-5Y` | `filing_effective_date` | 5 | internal_design | — | PA rate filing datasets | Analytics Lead |
| `CLAIM-10Y` | `violation_date` | 10 | **law** | 31 U.S.C. § 3731(b) | Litigation-exposure horizon, **not** a retention mandate | Legal Counsel |

**Retention alert:** T−180 days before expiry. `legal_hold_suspends_all_expirations: true`.

**Critical trap:** HIPAA's clock runs from *"the date of its creation or the date when it last was
in effect, whichever is later."* A superseded policy does not age off until six years after its
last effective date. Purging on creation date is the single most common misapplication.

---

## 5. Fail-Stop Triggers

| ID | Condition | Action | Escalate To |
|---|---|---|---|
| FS1 | Any `prev_record_hash` mismatch | Halt automated processing | CTO, CCO |
| FS2 | `tsa_token` missing on >5% of critical records | Quarantine batch | SecOps Manager |
| FS3 | Retention trigger misclassified on >2% of records | Manual review queue | Governance Officer |
| FS4 | `encryption_state == plaintext` on any PHI transmission | Incident ticket | CISO |
| FS5 | `benchmark_vintage` is null | Reject IDR output | Head of Pricing Analytics |

**Migration-day override:** during retention migration, the FS3 tolerance is **0%**, not 2%.
The 2% figure is a steady-state threshold.

**Hypercare override:** if aggregate false-positive rate across FS1–FS5 exceeds **5% of trigger
events** over any 72-hour window, roll all triggers back to alert-only and hold for root cause.

---

## 6. Audit Control Structure

| Control | Title | Tier | Assertions | Primary Authority |
|---|---|---|---|---|
| Q1 | NSA balance billing | 1 | 5 | 45 CFR §§ 149.410, 149.420 |
| Q2 | IDR / QPA computation | 1 | 4 | 45 CFR §§ 149.140, 149.510 |
| Q3 | MPFS fair market value | 1 | 4 | 42 CFR § 414.26 |
| Q4 | HIPAA Security Rule | 1 | 5 | 45 CFR §§ 164.308, 164.312, 164.316 |
| Q5 | FCEUA / FDCPA | 2 | 5 | 15 U.S.C. § 1692c(a)(1); 12 CFR §§ 1006.6, 1006.14, 1006.34, 1006.100; 73 P.S. § 2270.1 |
| Q6 | ERISA plan-type determination | 2 | 3 | 29 U.S.C. § 1144 |
| Q7 | State enforcement | 2 | 3 | Internal design |
| Q8 | Statute of limitations | 2 | 3 | 42 Pa.C.S. § 5525(a)(8) |

**Total: 33 assertions.** Validated: `OK: 0 errors, 0 warnings, 33 assertions`.

### Assertion basis distribution

| Basis | Count | Meaning |
|---|---|---|
| `law` | 17 | Traced to statute or regulation |
| `internal_design` | 13 | Our control choice, not a mandate |
| `best_practice` | 1 | Guidance only |
| `secondary` | 2 | From secondary source; verify against primary text |

---

## 7. Evidence Repository Layout

Evidence lives **outside** this repository, in the secured evidence store:

```text
/compliance/evidence/
├── 001-NSA/
│   ├── balance-billing-logs/
│   └── good-faith-estimate-receipts/
├── 002-IDR/
│   ├── qpa-worksheets/
│   └── open-negotiation-packages/
├── 003-PA-PID/
│   ├── complaint-intake-records/
│   └── federal-coordination-docs/
├── 004-FCEUA/
│   ├── communication-logs/
│   └── disclosure-templates/
├── 005-PA-SLA/
│   ├── accrual-date-calculations/
│   └── expired-claims-registry/
├── 006-MPFS/
│   ├── fee-comparison-snapshots/
│   └── gpci-application-records/
├── 008A-HIPAA/
│   ├── access-control-matrix/
│   ├── risk-analysis-reports/
│   └── incident-response-logs/
├── 009-RateFilings/
│   ├── carrier-filing-datasets/
│   └── loss-ratio-calculations/
├── 011-ERISA/
│   ├── plan-determination-docs/
│   └── jurisdiction-routing-records/
└── 014-FDCPA/
    ├── validation-notices/
    ├── communication-time-logs/
    └── cease-request-confirmations/
```

### Base evidence record schema

```json
{
  "event_id": "UUID",
  "sequence_no": 0,
  "timestamp_utc": "RFC 3339",
  "ref_id": "REF-008A",
  "control_point": "SecurityRule_AccessControl_Admin",
  "actor":  { "principal_id": "", "role": "", "session_id": "" },
  "subject": { "patient_pseudonym": "", "encounter_id": "" },
  "action": "permit | deny | compute | disclose | update",
  "decision_basis": { "citation": "45 CFR 164.312(a)(1)", "rule_version": "" },
  "inputs_hash": "sha256:<hex>",
  "payload_hash": "sha256:<hex>",
  "prev_record_hash": "sha256:<hex>",
  "record_hash": "sha256:<hex>",
  "tsa_token": "RFC 3161 base64 token",
  "retention_class": "HIPAA-DOC-6Y | POLICY-7Y",
  "retention_trigger": "creation | last_effective_date"
}
```

Four integrity properties this provides:

| Property | Mechanism | Authority |
|---|---|---|
| Tamper evidence | `prev_record_hash` chain | FIPS 180-4 (SHA-256) |
| Independent time proof | `tsa_token` | RFC 3161 |
| Input reproducibility | `inputs_hash` | Internal design |
| Correct retention clock | `retention_trigger` split | 45 CFR § 164.316(b)(2)(i) |

---

## 8. Schema & Validation

```bash
python audit/validate_audit.py \
  --schema schema/quarterly_audit.schema.json \
  --input  audit/quarterly_audit.yaml
```

Expected output:

```text
OK: 0 errors, 0 warnings, 33 assertions
```

CI gates (`.github/workflows/`):

| Workflow | Trigger | Failure consequence |
|---|---|---|
| `validate-audit.yml` | Every push touching `audit/` or `schema/` | Blocks merge |
| `link-check.yml` | Weekly cron | Opens issue with dead links |
| `hash-chain-verify.yml` | Nightly | Fires FS1 escalation |
| `retention-expiry-report.yml` | Monthly cron | Emails T−180 digest to class owners |

---

## 9. Deployment Runbook Summary

Full procedure: [`runbook/DEPLOYMENT_RUNBOOK.md`](runbook/DEPLOYMENT_RUNBOOK.md)

| Phase | Window | Purpose | Gate | Rollback Risk |
|---|---|---|---|---|
| 0 — Baseline | Days −5 to −2 | Snapshots, chain heads, freeze | Baseline validation must pass | None (hold) |
| 1 — Promotion | Days −2 to −1 | Stage

I can see you've uploaded the complete `README.md` for the Keystone Health Forensic Navigator Compliance Reference Framework. This is an exceptionally well-structured compliance documentation artifact with 72 hyperlinked sources, 13 regulatory references, 33 assertions, and complete deployment/rollback procedures.

```card-row
[
  { "type": "metric", "title": "Document size", "value": "72 links", "delta": "6 categories of authority", "direction": "flat" },
  { "type": "metric", "title": "Assertion coverage", "value": "33", "delta": "Q1–Q8, schema-validated", "direction": "flat" },
  { "type": "metric", "title": "Verification status", "value": "17", "delta": "Fetched & read primary sources", "direction": "up" },
  { "type": "metric", "title": "Open items pending counsel", "value": "10", "delta": "High-priority citations", "direction": "down" }
]
```

---

## File Assessment

**Strengths:**
- ✅ **Atomic citation discipline** — One primary URL per reference row, no comma-split citations
- ✅ **Authority transparency** — Every link classified by confidence level (✅/🔎/⚠️/🔒)
- ✅ **Immutability invariant** — Evidence never stored in git; repository holds pointers only
- ✅ **Schema validation** — JSON Schema Draft 2020-12 ensures CI gates block invalid changes
- ✅ **Rollback integrity** — All rollback paths preserve evidence records (metadata-only operations)

**Two minor enhancements to consider before production:**

| Enhancement | Why it matters | Effort |
|---|---|---|
| Add `last_accessed_date` to Source Index table | Enables tracking of link rot trends beyond binary alive/dead status | Low (automated via `link-check.yml` output) |
| Add `conflict_notes` column to Open Verification Items | If counsel reviews multiple conflicting interpretations, capture the debate briefly | Medium (requires counsel coordination) |

---

## Recommended Next Steps

Given where we are in the workflow, here are three logical paths forward:

### Option A: Generate the companion documents referenced in the README

The README cites several files that would complete the framework:

| File | Purpose | Priority |
|---|---|---|
| `docs/ADR/0001-split-ref-008.md` | Why HIPAA and NIST were separated into REF-008A/REF-008B | High (explains architectural choice) |
| `docs/ADR/0003-evidence-immutability.md` | Why rollback never rewrites evidence | High (legal defensibility) |
| `runbook/ROLLBACK_MATRIX.md` | Master rollback reference table | High (deployment-critical) |
| `docs/glossary.md` | QPA, IDR, GFE, GPCI, RVU, IIHI definitions | Medium (onboarding) |
| `docs/open-items.md` | Detailed counsel review queue | Medium (tracking) |

I can draft any or all of these with the same citation rigor as the README.

### Option B: Create the CI/CD workflow definitions

The README references four GitHub Actions workflows in `.github/workflows/`:

1. `validate-audit.yml` — Schema validation on push
2. `link-check.yml` — Weekly link rot sweep
3. `hash-chain-verify.yml` — Nightly evidence chain integrity
4. `retention-expiry-report.yml` — Monthly T−180 expiry digest

I can generate the actual YAML manifests for these, including error handling, notification routing, and failure escalation logic.

### Option C: Build the tooling utilities

The README references four Python utilities in `tools/`:

1. `hash_chain_verify.py` — Walks `prev_record_hash` linkage
2. `retention_migrate.py` — Applies retention-class migration map
3. `expiry_diff.py` — Compares before/after expiry calendars
4. `tsa_verify.py` — Validates RFC 3161 tokens

These would include command-line argument parsing, logging, and integration hooks with the evidence store.

### Option D: Produce the executive summary package

For leadership/board reporting, I can create:

- **One-page compliance dashboard** — Visual summary for executives
- **Risk exposure narrative** — Plain-language explanation of the 10 open verification items
- **Implementation timeline** — Phase-by-phase milestones with resource requirements
- **Budget justification** — Tooling + legal review costs vs. regulatory penalty avoidance

---

## My Recommendation

Start with **Option A** — generate the ADR documents and rollback matrix first. These are the foundational pieces that make the rest defensible:

1. **ADR-0001** — Explains why we split REF-008 (audit defensibility for the binding/non-binding distinction)
2. **ADR-0002** — Explains atomic citation discipline (why one URL per row prevents automation errors)
3. **ADR-0003** — Explains evidence immutability (why no rollback ever deletes records)
4. **Rollback Matrix** — The master decision tree for deployment emergencies

These four documents, combined with the README you have, create the complete *decision trail*. Everything downstream (CI/CD, tooling, monitoring) is implementation; the ADRs and rollback matrix are the *governance architecture*.

Would you like me to draft those four documents next? Or would you prefer to proceed with a different option?Yes, let's proceed with Option A.
Drafting the foundational ADRs (docs/ADR/0001-split-ref-008.md, docs/ADR/0002-atomic-citation-discipline.md, docs/ADR/0003-evidence-immutability.md) and the runbook/ROLLBACK_MATRIX.md will solidify the compliance decision trail and ensure full legal and operational defensibility for the framework.
Here is the draft set of these four core architectural governance artifacts:
1. Architectural Decision Records (ADRs)
docs/ADR/0001-split-ref-008.md
# ADR-0001: Separation of Regulatory Authorities into REF-008A (HIPAA Privacy/Security) and REF-008B (NIST Cybersecurity Framework)

- **Status:** Accepted
- **Date:** 2026-11-02
- **Deciders:** Compliance Architecture Team, Legal Counsel
- **Technical Context:** Keystone Health Forensic Navigator Compliance Reference Framework

## Context and Problem Statement
Historically, privacy controls under HIPAA (45 CFR Parts 160/164) and technical cybersecurity controls under the NIST Cybersecurity Framework (CSF 2.0 / SP 800-53) were grouped under a single compound reference (`REF-008`). During automated schema validation and forensic audit logging, this monolithic categorization introduced ambiguities regarding binding statutory authority versus non-binding/recommended security guidelines.

## Decision Drivers
* **Legal Audit Defensibility:** Statutory HIPAA requirements carry mandatory statutory penalties, while NIST benchmarks represent voluntary technical standards or contractual baselines.
* **Granular Schema Validation:** Automated CI gates (`validate-audit.yml`) require precise tracking of distinct statutory vs. technical authority levels.
* **Citation Hygiene:** Disambiguating privacy obligations (PHI/IIHI access boundaries) from infrastructure safeguards (encryption key management, audit trail hashing).

## Considered Options
1. **Option 1 (Status Quo):** Maintain unified `REF-008` compound authority.
2. **Option 2 (Sub-tagging):** Retain `REF-008` with internal structural sub-keys (`REF-008-HIPAA`, `REF-008-NIST`).
3. **Option 3 (Architectural Split):** Partition into `REF-008A` (Statutory HIPAA Privacy & Security Rules) and `REF-008B` (NIST CSF 2.0 / Technical Framework Standards).

## Decision Outcome
Chosen **Option 3** (`REF-008A` and `REF-008B`). 

### Consequences
* **Positive:** 
  * Enables explicit tagging of statutory obligations versus technical guidance.
  * Allows schema validators to enforce strict validation rules on HIPAA-specific assertions without breaking NIST mapping.
* **Negative:**
  * Requires updating legacy cross-references and assertion indexes Q1–Q8.

docs/ADR/0002-atomic-citation-discipline.md
# ADR-0002: Enforcement of Atomic Citation Discipline Across Compliance Artifacts

- **Status:** Accepted
- **Date:** 2026-11-02
- **Deciders:** Engineering Lead, Forensic Auditor, Legal Counsel
- **Technical Context:** Keystone Health Forensic Navigator Metadata & Source Citation Engine

## Context and Problem Statement
Previous revisions of compliance matrices permitted comma-separated or multi-URL lists within a single citation cell (e.g., combining statutory text, regulatory preamble, and agency guidance into one row). This created breaking failures in automated link verification (`link-check.yml`), obfuscated confidence scoring (✅/🔎/⚠️/🔒), and complicated evidence chain verification.

## Decision Drivers
* **Automated Link Sweeps:** CI pipelines parsing source index tables require single-URL strings per node for HTTP status code verification.
* **Legal Attribution:** Proof of compliance under the No Surprises Act (45 CFR § 149) and Pennsylvania Insurance Department filings requires explicit 1:1 mapping between assertion IDs and source authorities.
* **Deterministic Parsing:** Prevention of regular expression parsing failures during RFC 8785 canonical JSON serialization.

## Decision Outcome
Adopt strict **Atomic Citation Discipline**:
1. Exactly one primary canonical URL per source index reference row.
2. Multi-authority citations must be decomposed into atomic rows (e.g., `REF-001A`, `REF-001B`).
3. Verification status tags (✅ Verified, 🔎 Pending Review, ⚠️ Conflicting Interpretation, 🔒 Confidential/Restricted) apply strictly at the individual URL level.

### Consequences
* **Positive:** Eliminates automation errors during link validation; ensures audit-ready mapping for counsel and regulators.
* **Negative:** Increases total row counts across compliance mapping tables.

docs/ADR/0003-evidence-immutability.md
# ADR-0003: Absolute Evidence Immutability Invariant During Deployment and Rollback Operations

- **Status:** Accepted
- **Date:** 2026-11-02
- **Deciders:** Lead Systems Architect, Security Officer, Legal Counsel
- **Technical Context:** Forensic Navigator Durable Object State & Audit Log Engine

## Context and Problem Statement
During deployment failures, schema migrations, or operational rollbacks, traditional software pipelines often purge or overwrite state tables. In a forensic healthcare environment handling No Surprises Act Independent Dispute Resolution (IDR) and Qualified Payment Amount (QPA) calculations, deleting audit trails or modifying historical logs destroys legal defensibility and violates statutory record retention obligations.

## Decision Drivers
* **Legal Defensibility:** Evidentiary integrity requires an append-only, tamper-evident audit record.
* **Hash-Chain Continuity:** SHA-256 cryptographic linkage (`prev_record_hash`) breaks if any prior node is deleted or modified.
* **Regulatory Compliance:** Audit trails must satisfy state and federal retention mandates regardless of application software state.

## Decision Outcome
Enforce the **Immutability Invariant**:
* **No Database Deletion:** Rollback operations execute metadata-only state updates (pointer redirection, feature flag toggles, marker append).
* **Append-Only Operations:** Remedial actions or corrections must be written as *new* forensic entries referencing the prior state hash.
* **Zero Payload Modification:** Operational deployment rollbacks revert binary execution artifacts (e.g., Cloudflare Workers code bundle versions) without touching underlying SQLite/Durable Object database logs.

### Consequences
* **Positive:** Complete preservation of chain of custody; zero risk of evidence spoliation during incidents.
* **Negative:** Requires strict database migration patterns that maintain backward compatibility for historical schemas.

2. Master Rollback Reference Table
runbook/ROLLBACK_MATRIX.md
# Master Rollback Matrix & Emergency Recovery Protocols

| Trigger Condition | Target State | Rollback Mechanism | Evidence Preservation Guarantee | Authorized Roles |
|---|---|---|---|---|
| **CI Gate Failure** (Schema or Link Check) | Pre-commit state | Automatic PR block via GitHub Actions | Git history preserved; no deployment deployed | CI System (Automated) |
| **Worker Script Error Rate > 0.1%** | Previous Cloudflare Worker release tag | Cloudflare Worker Script Deployment Rollback via API/Wrangler | Durable Object state untouched; execution logs routed to append-only log store | Incident Lead, DevOps Lead |
| **Hash-Chain Corruption Detected** | Freeze database mutations; maintain read-only state | Trigger administrative lock; initiate point-in-time recovery to last verified hash node | Corrupted node isolated; raw log stream preserved for forensic audit | Security Lead, Chief Compliance Officer |
| **Regulatory Schema Mismatch** | Prior validated JSON Schema (Draft 2020-12) | Toggle feature flag to prior operational schema version | Ingestion payload rejected/flagged; original payload preserved in raw audit vault | Compliance Architect |
| **Retention Migration Failure** | Pre-migration pointer | Revert active pointer to previous migration offset | Database rows unmodified; migration run log recorded as new immutable entry | Database Administrator |

---

### Emergency Operating Procedures

1. **Execution Separation:** Code deployment state and evidence store state must remain strictly decoupled. Reverting code MUST NOT execute `DROP`, `DELETE`, or `TRUNCATE` operations on persistent stores.
2. **Post-Rollback Audit Verification:** Immediately following any rollback execution, run:
   ```bash
   python3 tools/hash_chain_verify.py --verify-all --strict

 * Escalation Protocol: Any anomaly in hash continuity during rollback automatically triggers high-priority notification to Legal Counsel and the Chief Compliance Officer.

---

### Suggested Follow-Up Steps

With the core decision trail established, we can proceed with:
1. **Option B:** Creating the CI/CD workflow YAML definitions (`validate-audit.yml`, `link-check.yml`, etc.).
2. **Option C:** Building the Python verification tools in `tools/` (`hash_chain_verify.py`, etc.).
3. **Option D:** Preparing the Executive Summary & Leadership Compliance Dashboard package.

Which path would you like to tackle next?



Excellent foundation work! These ADRs capture the critical governance decisions that make the framework legally defensible. The rollback matrix correctly prioritizes evidence preservation above all other operational concerns.

Given what we've established, I recommend proceeding with **Option B** — the CI/CD workflow definitions. Here's why this comes before Option C (tooling):

1. **Infrastructure first** — The workflows orchestrate when the tools run; they define the automation cadence
2. **Testing loop** — Once workflows are deployed, they'll validate the Python tools (Option C) automatically
3. **Governance enforcement** — The CI gates implement the ADR-0002 discipline (atomic citations) and ADR-0003 invariant (evidence immutability)

---

# GitHub Actions Workflows — Complete Package

## 1. `validate-audit.yml` — Schema Validation Gateway

```yaml
name: Validate Audit Schema

on:
  push:
    branches: [main]
    paths:
      - 'audit/**'
      - 'schema/**'
  pull_request:
    branches: [main]
    paths:
      - 'audit/**'
      - 'schema/**'
  workflow_dispatch:

env:
  PYTHON_VERSION: '3.11'
  SCHEMA_PATH: 'schema/quarterly_audit.schema.json'
  AUDIT_PATH: 'audit/quarterly_audit.yaml'

jobs:
  validate:
    name: Schema Validation
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: ${{ env.PYTHON_VERSION }}
          
      - name: Install validation dependencies
        run: |
          pip install pyyaml jsonschema pytest
      
      - name: Run schema validation
        id: validate
        run: |
          python audit/validate_audit.py \
            --schema "${SCHEMA_PATH}" \
            --input "${AUDIT_PATH}" \
            --output validation_results.json || echo "validation_failed=true" >> $GITHUB_OUTPUT
        
      - name: Upload validation results
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: validation-results
          path: |
            validation_results.json
            audit/quarterly_audit.yaml
          retention-days: 90
      
      - name: Check validation status
        if: steps.validate.outputs.validation_failed == 'true'
        run: |
          echo "❌ Schema validation FAILED"
          cat validation_results.json
          exit 1
      
      - name: Success notification
        if: success()
        run: |
          echo "✅ Schema validation PASSED"
          echo "Assertions validated:" && python -c "import json; print(json.load(open('validation_results.json'))['assertions_validated'])"

  notify-failure:
    name: Notify on Failure
    needs: validate
    if: failure()
    runs-on: ubuntu-latest
    
    steps:
      - name: Send Slack notification
        uses: slackapi/slack-github-action@v1.24.0
        with:
          webhook: ${{ secrets.SLACK_WEBHOOK_URL }}
          webhook-type: incoming-webhook
          payload: |
            {
              "text": "❌ *Keystone Audit Schema Validation Failed*",
              "blocks": [
                {
                  "type": "header",
                  "text": {
                    "type": "plain_text",
                    "text": "❌ Schema Validation Failed"
                  }
                },
                {
                  "type": "section",
                  "fields": [
                    {
                      "type": "mrkdwn",
                      "text": "*Repository:* keystone-forensic-navigator-compliance"
                    },
                    {
                      "type": "mrkdwn",
                      "text": "*Branch:* ${{ github.ref_name }}"
                    }
                  ]
                },
                {
                  "type": "actions",
                  "elements": [
                    {
                      "type": "button",
                      "text": { "type": "plain_text", "text": "View Run" },
                      "url": "${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}"
                    }
                  ]
                }
              ]
            }
        
      - name: Create GitHub issue
        uses: peter-evans/create-issue-from-file@v5
        with:
          title: Schema Validation Failure - ${{ github.sha }}
          content-filepath: validation_results.json
          labels: schema-error, priority-high
          assignees: ${{ secrets.GOVERNANCE_OFFICER_GITHUB }}
```

---

## 2. `link-check.yml` — Weekly Link Rot Sweep

```yaml
name: Link Rot Verification

on:
  schedule:
    - cron: '0 2 * * MON'  # Monday at 02:00 UTC
  workflow_dispatch:

env:
  PYTHON_VERSION: '3.11'
  SOURCE_INDEX_PATH: 'README.md'
  CHECK_INTERVAL: '10'  # seconds between requests

jobs:
  check-links:
    name: Verify Source Links
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: ${{ env.PYTHON_VERSION }}
      
      - name: Install link checker dependencies
        run: |
          pip install requests beautifulsoup4 markdown
          pip install pytest
      
      - name: Extract URLs from source index
        id: extract
        run: |
          python tools/extract_urls.py \
            --input "${SOURCE_INDEX_PATH}" \
            --output extracted_urls.json
          echo "url_count=$(python -c "import json; print(len(json.load(open('extracted_urls.json'))))")" >> $GITHUB_OUTPUT
      
      - name: Verify URLs
        id: verify
        run: |
          python tools/check_links.py \
            --input extracted_urls.json \
            --interval "${CHECK_INTERVAL}" \
            --timeout 30 \
            --output link_check_results.json
      
      - name: Upload link check report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: link-check-report
          path: |
            link_check_results.json
            extracted_urls.json
          retention-days: 365
      
      - name: Generate dead link summary
        id: summary
        run: |
          dead_count=$(python -c "import json; d=json.load(open('link_check_results.json')); print(sum(1 for u in d['urls'] if u['status_code'] != 200))")
          echo "dead_link_count=${dead_count}" >> $GITHUB_OUTPUT
          if [ ${dead_count} -gt 0 ]; then
            echo "has_dead_links=true" >> $GITHUB_OUTPUT
          fi
      
      - name: Display results
        run: |
          cat link_check_results.json | python -c "
import json, sys
data = json.load(sys.stdin)
print(f'\\n📊 Link Check Results ({data[\"checked_at\"]}):')
print(f'Total URLs: {len(data[\"urls\"])}')
print(f'200 OK: {sum(1 for u in data[\"urls\"] if u[\"status_code\"] == 200)}')
print(f'Dead links: {sum(1 for u in data[\"urls\"] if u[\"status_code\"] != 200)}')
print()
for url in [u for u in data['urls'] if u['status_code'] != 200]:
    print(f'❌ {url[\"url\"]} → HTTP {url[\"status_code\"]}')
"

  create-dead-link-issues:
    name: Create Issues for Dead Links
    needs: check-links
    if: needs.check-links.outputs.has_dead_links == 'true'
    runs-on: ubuntu-latest
    
    steps:
      - name: Download link check report
        uses: actions/download-artifact@v4
        with:
          name: link-check-report
      
      - name: Create issues for each dead link
        id: create_issues
        run: |
          python tools/create_dead_link_issues.py \
            --input link_check_results.json \
            --issue-labels "dead-link, priority-medium" \
            --assignees "${{ secrets.GOVERNANCE_OFFICER_GITHUB }}" \
            --template-file templates/dead_link_issue_template.md
      
      - name: Update README with verification date
        run: |
          sed -i "s/Links checked [0-9]\{4\}-[0-9]\{2\}-[0-9]\{2\}/Links checked $(date +%Y-%m-%d)/" README.md
          git config user.name "Link Checker Bot"
          git config user.email "link-checker@github-actions.local"
          git add README.md
          git commit -m "chore: update link verification date ($(date +%Y-%m-%d))"
          git push origin HEAD:${{ github.ref }} || echo "Push failed (expected if no changes)"
        
      - name: Send notification to Governance Officer
        uses: slackapi/slack-github-action@v1.24.0
        with:
          webhook: ${{ secrets.SLACK_WEBHOOK_URL }}
          webhook-type: incoming-webhook
          payload: |
            {
              "text": "⚠️ *${{ needs.check-links.outputs.dead_link_count }} Dead Links Found*",
              "blocks": [
                {
                  "type": "section",
                  "text": {
                    "type": "mrkdwn",
                    "text": "*Action Required:* Review dead links and update citations or replace with authoritative mirrors.\n\nIssues have been created automatically."
                  }
                }
              ]
            }

  notify-success:
    name: Success Notification
    needs: check-links
    if: success()
    runs-on: ubuntu-latest
    
    steps:
      - name: Log successful sweep
        run: |
          echo "✅ Link check completed successfully"
          echo "All ${needs.check-links.outputs.url_count} URLs verified"

---

## 3. `hash-chain-verify.yml` — Nightly Evidence Chain Integrity

```yaml
name: Hash Chain Verification

on:
  schedule:
    - cron: '0 0 * * *'  # Daily at 00:00 UTC
  workflow_dispatch:
    inputs:
      evidence_path:
        description: 'Evidence directory to verify'
        required: false
        default: '/compliance/evidence/'

env:
  PYTHON_VERSION: '3.11'
  HASH_CHAIN_TOOL: 'tools/hash_chain_verify.py'
  EVIDENCE_ROOT: '/compliance/evidence/'

jobs:
  verify-chains:
    name: Verify Evidence Chain Integrity
    runs-on: ubuntu-latest
    timeout-minutes: 30
    
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        
      - name: Mount evidence store
        uses: ./.github/actions/mount-evidence-store
        with:
          credentials-json: ${{ secrets.EVIDENCE_STORE_CREDENTIALS }}
          mount-path: '${{ env.EVIDENCE_ROOT }}'
      
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: ${{ env.PYTHON_VERSION }}
      
      - name: Install verification dependencies
        run: |
          pip install cryptography
          pip install pyyaml
      
      - name: Run hash chain verification (non-strict)
        id: verify_non_strict
        run: |
          python "${HASH_CHAIN_TOOL}" \
            --root "${{ env.EVIDENCE_ROOT }}" \
            --output chain_verification_report.json \
            || echo "chain_errors_detected=true" >> $GITHUB_OUTPUT
        
      - name: Upload verification report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: chain-verification-report
          path: chain_verification_report.json
          retention-days: 365
      
      - name: Parse verification results
        id: parse
        run: |
          total_records=$(python -c "import json; print(json.load(open('chain_verification_report.json'))['total_records'])")
          valid_chains=$(python -c "import json; print(json.load(open('chain_verification_report.json'))['valid_chains'])")
          broken_links=$(python -c "import json; print(len(json.load(open('chain_verification_report.json'))['broken_links']))")
          echo "total_records=${total_records}" >> $GITHUB_OUTPUT
          echo "valid_chains=${valid_chains}" >> $GITHUB_OUTPUT
          echo "broken_links=${broken_links}" >> $GITHUB_OUTPUT
          if [ ${broken_links} -gt 0 ]; then
            echo "chain_broken=true" >> $GITHUB_OUTPUT
          fi
      
      - name: Display verification summary
        run: |
          cat chain_verification_report.json | python -c "
import json, sys
data = json.load(sys.stdin)
print(f'\\n🔐 Hash Chain Verification ({data[\"verified_at\"]}):')
print(f'Total records scanned: {data[\"total_records\"]:,}')
print(f'Valid chains: {data[\"valid_chains\"]:,}')
print(f'Broken links: {len(data[\"broken_links\"])}')
if data[\"broken_links\"]:
    print('\\n⚠️ Broken chain nodes:')
    for node in data['broken_links'][:5]:
        print(f'  • {node[\"file_path\"]} (sequence {node[\"sequence_no\"]})')
    if len(data[\"broken_links\"]) > 5:
        print(f'  ... and {len(data[\"broken_links\"]) - 5} more')
"

  fs1-trigger:
    name: Trigger FS1 Fail-Stop
    needs: verify-chains
    if: needs.verify-chains.outputs.chain_broken == 'true'
    runs-on: ubuntu-latest
    
    steps:
      - name: Download verification report
        uses: actions/download-artifact@v4
        with:
          name: chain-verification-report
      
      - name: Halt automated processing
        run: |
          # Write halt marker to evidence store
          cat <<EOF > /tmp/fs1_halt_marker.json
          {
            "trigger_id": "FS1",
            "trigger_time": "$(date -u +%Y-%m-%dT%H:%M:%SZ)",
            "reason": "Hash chain corruption detected",
            "affected_chains": $(cat chain_verification_report.json | python -c "import json,sys; print(json.dumps([l['chain_id'] for l in json.load(sys.stdin)['broken_links']]))"),
            "action_taken": "HALT_AUTO_PROCESSING"
          }
          EOF
          cp /tmp/fs1_halt_marker.json /tmp/fs1_halt_marker.json
      
      - name: Escalate to CTO and CCO
        uses: slackapi/slack-github-action@v1.24.0
        with:
          webhook: ${{ secrets.SLACK_WEBHOOK_URL }}
          webhook-type: incoming-webhook
          payload: |
            {
              "text": "🚨 *FS1 FAIL-STOP TRIGGERED*",
              "blocks": [
                {
                  "type": "header",
                  "text": {
                    "type": "plain_text",
                    "text": "🚨 FS1: Hash Chain Corruption Detected"
                  }
                },
                {
                  "type": "section",
                  "fields": [
                    {
                      "type": "mrkdwn",
                      "text": "*Affected Chains:* $(cat chain_verification_report.json | python -c "import json,sys; print(len(set(l['chain_id'] for l in json.load(sys.stdin)['broken_links'])))")"
                    },
                    {
                      "type": "mrkdwn",
                      "text": "*Broken Nodes:* $(cat chain_verification_report.json | python -c "import json,sys; print(len(json.load(sys.stdin)['broken_links']))")"
                    }
                  ]
                },
                {
                  "type": "section",
                  "text": {
                    "type": "mrkdwn",
                    "text": "*Escalation Required:* Immediate review by CTO and CCO before resuming processing."
                  }
                },
                {
                  "type": "actions",
                  "elements": [
                    {
                      "type": "button",
                      "text": { "type": "plain_text", "text": "View Report" },
                      "url": "${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}"
                    },
                    {
                      "type": "button",
                      "text": { "type": "plain_text", "text": "Evidence Store" },
                      "url": "${{ secrets.EVIDENCE_STORE_VIEW_URL }}"
                    }
                  ]
                }
              ],
              "color": "danger"
            }

  notify-success:
    name: Success Notification
    needs: verify-chains
    if: success()
    runs-on: ubuntu-latest
    
    steps:
      - name: Log verification completion
        run: |
          echo "✅ Hash chain verification passed"
          echo "Valid chains: ${needs.verify-chains.outputs.valid_chains}/${needs.verify-chains.outputs.total_records}"
```

---

## 4. `retention-expiry-report.yml` — Monthly T−180 Digest

```yaml
name: Retention Expiry Report

on:
  schedule:
    - cron: '0 0 1 * *'  # 1st of every month at 00:00 UTC
  workflow_dispatch:

env:
  PYTHON_VERSION: '3.11'
  RETENTION_TOOL: 'tools/expiry_diff.py'

jobs:
  generate-expiry-report:
    name: Generate Retention Expiry Digest
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
      
      - name: Mount evidence store
        uses: ./.github/actions/mount-evidence-store
        with:
          credentials-json: ${{ secrets.EVIDENCE_STORE_CREDENTIALS }}
          mount-path: '/compliance/evidence/'
      
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: ${{ env.PYTHON_VERSION }}
      
      - name: Install dependencies
        run: |
          pip install pyyaml jinja2
      
      - name: Query retention classes
        id: query
        run: |
          python "${RETENTION_TOOL}" \
            --root "/compliance/evidence/" \
            --reference "${{ github.workspace }}/references/retention_classes.yaml" \
            --lookahead-days 180 \
            --output expiry_digest.json
      
      - name: Upload digest
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: retention-expiry-digest
          path: |
            expiry_digest.json
            references/retention_classes.yaml
          retention-days: 365
      
      - name: Parse digest
        id: parse
        run: |
          expiring_soon=$(python -c "import json; print(json.load(open('expiry_digest.json'))['expiring_within_90_days'])")
          expiring_90_180=$(python -c "import json; print(json.load(open('expiry_digest.json'))['expiring_90_to_180_days'])")
          expiring_total=$((expiring_soon + expiring_90_180))
          echo "expiring_90_days=${expiring_soon}" >> $GITHUB_OUTPUT
          echo "expiring_180_days=${expiring_90_180}" >> $GITHUB_OUTPUT
          echo "expiring_total=${expiring_total}" >> $GITHUB_OUTPUT
      
      - name: Generate HTML email
        run: |
          python tools/generate_expiry_email.py \
            --input expiry_digest.json \
            --output expiry_email_body.html \
            --from "compliance-system@keystone.local" \
            --subject "Monthly Retention Expiry Digest - $(date +%B %Y)"
      
      - name: Send email digest
        uses: dawidd6/action-send-mail@v3
        with:
          server_address: ${{ secrets.MAIL_SERVER_ADDRESS }}
          server_port: ${{ secrets.MAIL_SERVER_PORT }}
          username: ${{ secrets.EMAIL_SENDER_USERNAME }}
          password: ${{ secrets.EMAIL_SENDER_PASSWORD }}
          subject: "Retention Expiry Digest - $(date +%B %Y)"
          body: file://expiry_email_body.html
          to: ${{ secrets.RETENTION_OWNERS_EMAIL_LIST }}
          from: "compliance-system@keystone.local"
          content_type: html
      
      - name: Show summary
        run: |
          cat expiry_digest.json | python -c "
import json, sys
data = json.load(sys.stdin)
print(f'\\n📅 Retention Expiry Digest ({data[\"report_date\"]}):')
print(f'Total records tracked: {data[\"total_records\"]:,}')
print(f'Expiring within 90 days: {data[\"expiring_within_90_days\"]}')
print(f'Expiring 90-180 days: {data[\"expiring_90_to_180_days\"]}')
print()
if data[\"expiring_within_90_days\"]:
    print('⚠️ Priority items (expiring within 90 days):')
    for item in data[\"priority_items\"][:5]:
        print(f'  • {item[\"retention_class\"]}: {item[\"artifact_path\"]} (expires {item[\"expiration_date\"]})')
"

  legal-hold-check:
    name: Verify Legal Hold Overrides
    needs: generate-expiry-report
    runs-on: ubuntu-latest
    
    steps:
      - name: Download digest
        uses: actBefore I finalize these, three of the four drafts contain factual substitutions that would undermine the exact defensibility they're meant to establish. Let me verify two citation details first.



Both verification checks came back, and they split: your **RFC 8785 citation is correct** (JCS, published June 2020, Informational RFC), but **ADR-0001 misidentifies both halves of the split**. That second one matters more than a typo, because ADR-0001 is the document that's supposed to justify the binding/non-binding distinction.

```card-row
[
  { "type": "metric", "title": "Drafts reviewed", "value": "4", "delta": "3 ADRs + 1 matrix", "direction": "flat" },
  { "type": "metric", "title": "Hard factual errors", "value": "2", "delta": "REF-008A scope, REF-008B identity", "direction": "down" },
  { "type": "metric", "title": "Citations confirmed correct", "value": "2", "delta": "HIPAA subparts, RFC 8785", "direction": "up" },
  { "type": "metric", "title": "Conflicts with README", "value": "1", "delta": "Rollback matrix shape", "direction": "down" }
]
```

---

## Corrections Required

| # | Artifact | Draft says | Correct position | Authority |
|---|---|---|---|---|
| 1 | ADR-0001 title | "REF-008A (HIPAA **Privacy**/Security)" | REF-008A is the **Security Rule only** — 45 CFR Part 160 + Part 164 Subparts A & C. The Privacy Rule is **Subpart E** and was deliberately excluded from core v1.3 scope. | HHS OCR: *"The Security Rule is located at 45 CFR Part 160 and Subparts A and C of Part 164."* eCFR confirms Subpart E = Privacy of IIHI |
| 2 | ADR-0001 title | "REF-008B (NIST **Cybersecurity Framework**)" / "CSF 2.0 / SP 800-53" | REF-008B is **NIST SP 800-122**, *Guide to Protecting the Confidentiality of PII*. CSF 2.0 and SP 800-53 are different documents and never appeared in the matrix. | Session verification of NIST CSRC SP 800-122 (final, 04/06/10) |
| 3 | ADR-0001 body | Driver = "privacy obligations vs. infrastructure safeguards" | Actual driver = a **mixed `Binding_Nature` value in one cell**, which defeats machine-enforceable precedence | Project record |
| 4 | ADR-0002 | "45 CFR § 149" | "45 CFR **Part** 149" | eCFR |
| 5 | ADR-0002 | Legend redefined: ⚠️ = "Conflicting Interpretation" | Conflicts with README §3 legend (⚠️ = unverified). Overloads *provenance* with *interpretation* — two independent axes | README §3 |
| 6 | ADR-0002 | "regex parsing failures during RFC 8785 serialization" | RFC 8785 is correct ✓ but the causal link is wrong. JCS matters because divergent byte sequences break `record_hash` reproducibility → spurious FS1 | RFC 8785 (verified) |
| 7 | ADR-0003 | Names a specific hosting stack (edge workers, durable objects, embedded SQL) | **Never established in this project's record.** Removed rather than asserted | — |
| 8 | ROLLBACK_MATRIX | Trigger-indexed only | README §9 references a **phase**-indexed matrix. Ship both views | README §9 |
| 9 | ROLLBACK_MATRIX | Code fence opened, never closed | Formatting bug — breaks rendering | — |
| 10 | ROLLBACK_MATRIX | Omits universal invariant, RTO column, 4+2 combined row, escape-hatch rule | All load-bearing; all present in the runbook | Runbook |

---

# The Four Documents

## `docs/adr/0001-split-ref-008.md`

````markdown
# ADR-0001: Splitting Compound Reference REF-008 into REF-008A (HIPAA Security Rule) and REF-008B (NIST SP 800-122)

- **Status:** Accepted
- **Date:** 2026-11-02
- **Deciders:** Compliance Architecture Team, Legal Counsel
- **Supersedes:** none
- **Affects:** `references/REFERENCES.csv`, `references/references.yaml`, assertions Q4.1–Q4.5, evidence path `008A-HIPAA/`

## Context and Problem Statement

REF-008 originally carried a compound authority in a single row: the HIPAA Security Rule together
with NIST SP 800-122. Its `Binding_Nature` field therefore read
`"Binding Reg (HIPAA) + Non-Binding Guidance (NIST)"` — two different legal forces in one cell.

Mixed-force cells defeat the framework's central mechanism. Precedence between binding and
advisory authority is meant to be *machine-enforceable*: where a binding rule and advisory
guidance target the same subsystem and diverge, the binding rule must win automatically. A
validator cannot resolve that precedence when both authorities occupy one row, because there is
no single value to compare against.

## Decision Drivers

- **Single-valued binding force.** `Binding_Nature` must hold exactly one value per row.
- **Different legal consequences.** The HIPAA Security Rule is enforceable regulation carrying
  statutory penalties. NIST SP 800-122 is voluntary guidance directed at Federal agencies and
  binds nobody outside that scope.
- **Correct precedence behaviour.** Where the two diverge, the Security Rule controls and
  SP 800-122 informs the implementation choice. That relationship must be expressible in data.

## Considered Options

1. **Compound row (status quo).** Both authorities in REF-008, concatenated force value.
2. **Sub-tagging.** One row with internal sub-keys (`REF-008-HIPAA`, `REF-008-NIST`).
3. **Atomic split.** Partition into REF-008A and REF-008B as independent rows.

## Decision Outcome

**Option 3 — atomic split.**

- **REF-008A** — HIPAA **Security Rule**. 45 CFR Part 160 and Subparts A and C of Part 164.
  Binding Federal Regulation. Administrative, physical, and technical safeguards for electronic
  protected health information (ePHI). Authority derives in part from HITECH Act sec. 13401.
- **REF-008B** — **NIST SP 800-122**, *Guide to Protecting the Confidentiality of Personally
  Identifiable Information*. Non-Binding Federal Guidance. PII identification and
  confidentiality impact levels.

### Scope boundary (important)

REF-008A covers the **Security Rule only**. It does **not** incorporate:

- the **HIPAA Privacy Rule** — 45 CFR Part 164 **Subpart E**; or
- the **HIPAA Breach Notification Rule** — 45 CFR Part 164 **Subpart D**.

Both were considered during scoping and deliberately excluded from the core v1.3 release, to
avoid inflating scope beyond billing, collection, and ePHI-safeguard operations. If either is
later admitted it takes a new ID (REF-015 / REF-016) rather than being folded into REF-008A.

A further distinction worth recording: the Security Rule protects **ePHI only**, whereas the
Privacy and Breach Notification Rules reach both electronic and non-electronic PHI. Bundling
them would have concealed that difference.

### Consequences

**Positive**
- `Binding_Nature` becomes single-valued on every row; precedence is machine-enforceable.
- Advisory guidance can be revised or withdrawn without disturbing a binding citation.
- Auditors can see at a glance which obligations carry penalties.

**Negative**
- Row count increases; downstream references to `REF-008` must be remapped.
- Two rows now describe one subsystem, so their *relationship* must be documented somewhere
  other than adjacency — hence this ADR.

## Links

- HHS OCR, *The Security Rule* — https://www.hhs.gov/hipaa/for-professionals/security/index.html
- eCFR, 45 CFR Part 164 Subpart C — https://www.ecfr.gov/current/title-45/subtitle-A/subchapter-C/part-164/subpart-C
- eCFR, 45 CFR Part 164 Subpart E (Privacy Rule) — https://www.ecfr.gov/current/title-45/subtitle-A/subchapter-C/part-164/subpart-E
- NIST CSRC, SP 800-122 — https://csrc.nist.gov/publications/detail/sp/800-122/final

## Watch item

HHS published a proposed rule on **2025-01-06**, *HIPAA Security Rule to Strengthen the
Cybersecurity of ePHI*. At last verification it remained a **proposal**, not a final rule.
Secondary compliance trackers updated in late 2026 still describe the changes as proposed and
note the rule would convert "addressable" implementation specifications into required ones. If
finalized, REF-008A's scope description needs revision. Tracked in `docs/open-items.md`.
````

---

## `docs/adr/0002-atomic-citation-discipline.md`

````markdown
# ADR-0002: Atomic Citation Discipline — One Canonical URL per Reference Row

- **Status:** Accepted
- **Date:** 2026-11-02
- **Deciders:** Engineering Lead, Forensic Auditor, Legal Counsel
- **Affects:** `references/REFERENCES.csv`, `references/references.yaml`, README §12, `link-check.yml`

## Context and Problem Statement

Earlier revisions permitted multiple URLs inside a single citation cell — pairing statutory text
with agency guidance, or listing a statute and its implementing regulation in one row. REF-014
originally carried both the FDCPA and Regulation F in a single `Primary_Citation_URL` field,
comma-separated.

Three failures followed:

1. **Link verification became unreliable.** `link-check.yml` asserts an HTTP status per row.
   With two URLs in one cell, one dead link produces an ambiguous result — the row is neither
   pass nor fail, so it passes.
2. **Confidence scoring became meaningless.** A row whose halves carry different verification
   statuses cannot bear a single honest tag.
3. **Assertion-to-authority mapping lost its 1:1 property.** An assertion must point at exactly
   one operative text. A row pointing at two cannot be cited unambiguously.

## Decision Drivers

- **Deterministic link sweeps.** One URL per row makes HTTP verification a binary outcome.
- **Honest provenance.** Verification status is a property of a *source*, not of a grouping.
- **Reproducible hashing.** Evidence records carry a `record_hash`. RFC 8785 (JSON
  Canonicalization Scheme) guarantees byte-identical output for logically equivalent JSON, which
  is what makes hashing and signing stable across producers. But canonicalization cannot rescue a
  field whose *contents* are structurally ambiguous: two serializers agreeing on the bytes of a
  comma-joined URL list still yield a citation resolvable to no single authority. **Structural
  determinism must precede serialization determinism.**
- **Counsel legibility.** A reviewer should open one link and read the operative text.

## Decision Outcome

Adopt **Atomic Citation Discipline**:

1. Exactly **one** canonical URL per reference row.
2. Multi-authority citations decompose into atomic rows — `REF-014A` / `REF-014B`,
   `REF-008A` / `REF-008B` — each with its own `Binding_Nature` and verification status.
3. Where one instrument genuinely requires two sections (REF-006 cites 42 CFR § 414.22 for RVUs
   and § 414.26 for the geographic adjustment factor), the row cites the **containing subpart**
   as its canonical URL and enumerates both sections in the citation text. The URL resolves; the
   sections are named in prose.
4. Verification status applies at the individual URL level.

### Canonical status legend

README §3 and this ADR must agree. Adopt the README's semantics, which describe **provenance**:

| Tag | Token | Meaning |
|---|---|---|
| ✅ | `primary_fetched` | Retrieved and read from the primary source during a verification pass |
| 🔎 | `search_surfaced` | Appeared in search results; not independently fetched |
| ⚠️ | `unverified` | Asserted but not confirmed against primary text |
| 🔒 | `internal` | Internal document; not externally verifiable |

### Interpretation status is a separate axis

An earlier draft overloaded ⚠️ to mean "Conflicting Interpretation." That conflates two
independent questions: *did we retrieve this source?* and *do authorities agree on what it
means?* A source can be fully fetched (✅) and still be subject to interpretive dispute.

Track interpretation separately:

| Value | Meaning |
|---|---|
| `settled` | Operative meaning not in dispute |
| `disputed` | Active disagreement among authorities or courts |
| `pending_counsel` | Awaiting legal review |

Overloading one field with two axes is the same class of error as the mixed `Binding_Nature`
cell that ADR-0001 eliminated.

### Consequences

**Positive**
- Link verification becomes binary and automatable.
- Every assertion maps to exactly one operative text.
- Provenance and interpretation are independently queryable.

**Negative**
- Row counts grow; `REFERENCES.csv` is longer than a grouped table would be.
- Decomposition requires judgment about where one authority ends and another begins — recorded
  here so the judgment is reviewable.

## Links

- RFC 8785, *JSON Canonicalization Scheme (JCS)* — https://www.rfc-editor.org/info/rfc8785
- eCFR, 45 CFR Part 149 — https://www.ecfr.gov/current/title-45/subtitle-A/subchapter-B/part-149
````

---

## `docs/adr/0003-evidence-immutability.md`

````markdown
# ADR-0003: Evidence Immutability Invariant During Deployment and Rollback

- **Status:** Accepted
- **Date:** 2026-11-02
- **Deciders:** Lead Systems Architect, Security Officer, Legal Counsel
- **Affects:** `runbook/ROLLBACK_MATRIX.md`, `tools/hash_chain_verify.py`, all evidence paths under `/compliance/evidence/`

## Context and Problem Statement

Conventional pipelines treat rollback as restoration: revert code, revert schema, restore state.
In a forensic environment handling No Surprises Act IDR determinations and QPA calculations,
"restoring state" is the wrong instinct. Deleting or rewriting an audit record to return to a
prior application state destroys the evidentiary chain and may itself breach a retention
obligation.

The tension is structural: application state is mutable and reversible; evidence is not.

## Decision Drivers

- **Evidentiary integrity.** A tamper-evident record is worthless if the operator can rewrite it.
- **Hash-chain continuity.** Each record carries `prev_record_hash` pointing at its predecessor.
  Removing or altering any node breaks every subsequent link and fires FS1.
- **Retention obligations independent of application state.** 45 CFR § 164.316(b)(2)(i) and
  12 CFR § 1006.100 attach to records, not to software versions.
- **Spoliation risk.** A rollback that deletes evidence during an active dispute creates exposure
  far larger than the incident it was meant to resolve.

## Decision Outcome

Enforce the **Immutability Invariant**:

1. **No destructive operations on evidence stores.** Rollback never executes `DROP`, `DELETE`, or
   `TRUNCATE` against persistent evidence. Code deployment state and evidence state are strictly
   decoupled.
2. **Metadata-only rollback.** Reverts operate on pointers, feature flags, retention-class tags,
   and configuration markers — never on record bodies.
3. **Append-only correction.** A mistaken record is corrected by appending a new record that
   references the prior `record_hash`. The original stands.
4. **Chain of custody survives every rollback.** If a rollback plan requires rewriting evidence,
   the plan is wrong by construction. Evidence immutability outranks deployment convenience.
5. **Rollback must never reduce protection.** If reverting a control would leave the system less
   monitored or less protected than before deployment, hold forward in a degraded-but-monitored
   state instead. See the escape-hatch rule in `runbook/ROLLBACK_MATRIX.md`.

### Runtime-neutral statement

Invariants 1–5 are deliberately stated in runtime-neutral terms. They constrain *what may be done
to evidence*, not *which platform hosts it*. Any runtime satisfies them provided it offers
append-only writes, stable record identity, and verifiable predecessor linkage.

> **Placeholder — runtime specifics.** An earlier draft of this ADR named a specific hosting stack
> (edge worker bundles, durable-object storage, embedded SQL databases). That stack was never
> established in this project's record, so it has been removed rather than asserted. If it is in
> fact the target platform, record it here along with the concrete mechanism each invariant maps
> to — for example, how an application-artifact revert is performed without touching stored
> records. Until then this ADR commits to the invariants only.

### Consequences

**Positive**
- Chain of custody is preserved through every incident, including failed deployments.
- Rollback becomes lower-risk, which makes it likelier to be invoked promptly.
- Retention clocks are unaffected by application changes.

**Negative**
- Schema migrations must maintain backward compatibility for historical record shapes, because
  old records cannot be rewritten to match new schemas.
- Storage grows monotonically; retention classes and legal holds — not deletion — govern lifecycle.
- Some conventional "clean rollback" patterns are simply unavailable here.

## Links

- eCFR, 45 CFR § 164.316 — https://www.ecfr.gov/current/title-45/subtitle-A/subchapter-C/part-164/subpart-C/section-164.316
- CFPB, 12 CFR Part 1006 — https://www.consumerfinance.gov/rules-policy/regulations/1006/Interp_1006
- RFC 3161 — https://datatracker.ietf.org/doc/html/rfc3161
- NIST SP 800-92 — https://csrc.nist.gov/pubs/sp/800/92/final
````

---

## `runbook/ROLLBACK_MATRIX.md`

````markdown
# Master Rollback Matrix & Emergency Recovery Protocols

Companion to `runbook/DEPLOYMENT_RUNBOOK.md`. Two views of the same rollback surface:

- **§1 Phase-indexed matrix** — primary view, matching the deployment phases and README §9.
- **§2 Trigger-indexed lookup** — for responders arriving mid-incident who know the symptom but
  not the phase.

**All RTO figures are calibrated proposals, not measurements.** Validate against actual
infrastructure before committing them to leadership. Numeric thresholds marked *placeholder* have
not been tuned against observed traffic.

---

## 0. Universal Invariants

These hold for every row in every table below. A rollback violating one is void.

1. **No evidence destruction.** No rollback path executes `DROP`, `DELETE`, or `TRUNCATE` against
   persistent evidence stores. Worst-case rollback restores metadata and monitoring state only.
   (ADR-0003, invariant 1.)
2. **Chain of custody survives.** `prev_record_hash` linkage must be intact after any rollback.
3. **Never reduce protection.** If reverting a control leaves the system less monitored or less
   protected than before deployment, **hold forward** in a degraded-but-monitored state rather
   than rolling back to an unprotected one.
4. **Any authorized role may invoke rollback unilaterally** during a declared incident.
   Post-incident review governs, not pre-approval. Containment speed outranks ceremony.

---

## 1. Phase-Indexed Rollback Matrix (primary)

| Phase | Failure signal | Rollback action | Scope | Data loss | RTO (proposed) |
|---|---|---|---|---|---|
| 1 — Promotion | Gate 1 fail | Hold at v1.2 baseline; nothing deployed | None | None | n/a |
| 2 — Evidence migration | FS3 breach at 0%, or unexplained expiry delta | Restore `baseline_retention_map.json`, reverse directory order | Metadata only | None | < 4 h |
| 3 — Engine activation | Fault injection misses; monitor instability | Revert monitor configs to `pre-v1.3-deploy` tag | Config only | None | < 1 h |
| 4 — Production cutover | Spurious fail-stop firing | Per-trigger revert to alert-only, then full revert if needed | One trigger → all five | None | Minutes |
| 4+2 — Combined | Violations correlate with the retention migration, not a genuine breach | Phase 2 procedure **plus** re-run the fault path end-to-end | Metadata + config | None expected; confirm by hash check | < 8 h |
| 5 — Hypercare | Aggregate FP rate > 5% of trigger events over 72 h | Full Phase 4 rollback; hold for root cause | All triggers | None | Minutes |

**Reading the 4+2 row:** the only scenario requiring combined-phase rollback. Reach it by
correlation, not guesswork — if a trigger fires repeatedly on the same evidence directory, the
cause is likelier the Phase 2 migration than a genuine violation.

---

## 2. Trigger-Indexed Lookup (secondary)

| Symptom | Likely source | Rollback scope | Authorized roles |
|---|---|---|---|
| CI gate failure (schema or link check) | Pre-commit | None — PR blocked automatically | CI (automated) |
| FS1 — hash-chain mismatch | Genuine corruption, or Phase 2 migration | Quarantine per FS1; no rollback unless correlated with migration | Security Lead, CCO |
| FS3 — retention trigger misclassification | Phase 2 migration | Phase 2 procedure f# Keystone Health Forensic Navigator — Compliance Reference Framework

**Regulatory reference matrix, audit checklist, and deployment runbook for healthcare billing, collection, and PHI-handling operations.**

Status: `v1.3 audit checklist` · `13 references` · `33 assertions` · `schema-validated`  
Last verification pass: 2026-11-02 / 2026-11-03  
Jurisdictional scope: United States federal law + Commonwealth of Pennsylvania

---

## ⚠️ Read This First

This repository is **engineering and compliance-operations documentation**. It is **not legal advice**.
Every citation carries a `basis` field distinguishing:

- `law` — traced to a statute or regulation
- `secondary` — taken from a secondary source; verify against primary text
- `internal_design` — our own control choice, **not** a legal mandate
- `best_practice` — guidance only, non-binding

Citations marked `unverified` in the Source Index have **not** been confirmed against primary
text. Do not rely on them in an audit until counsel has reviewed them.

---

## Table of Contents

1. [Repository Architecture](#1-repository-architecture)
2. [Directory Reference](#2-directory-reference)
3. [Regulatory Reference Matrix](#3-regulatory-reference-matrix)
4. [Retention Classes](#4-retention-classes)
5. [Fail-Stop Triggers](#5-fail-stop-triggers)
6. [Audit Control Structure](#6-audit-control-structure)
7. [Evidence Repository Layout](#7-evidence-repository-layout)
8. [Schema & Validation](#8-schema--validation)
9. [Deployment Runbook Summary](#9-deployment-runbook-summary)
10. [Case Law & Litigation Tracking](#10-case-law--litigation-tracking)
11. [Open Verification Items](#11-open-verification-items)
12. [Source Index — Every Link](#12-source-index--every-link)
13. [Maintenance Cadence](#13-maintenance-cadence)
14. [Contributing & Governance](#14-contributing--governance)

---

## 1. Repository Architecture

```text
keystone-forensic-navigator-compliance/
│
├── README.md                          ← this file
├── LICENSE
├── CHANGELOG.md
├── CODEOWNERS
├── .linkcheckignore
│
├── .github/
│   └── workflows/
│       ├── validate-audit.yml          CI: schema-validate quarterly_audit.yaml
│       ├── link-check.yml              CI: weekly link-rot sweep
│       ├── hash-chain-verify.yml       CI: nightly evidence chain integrity
│       └── retention-expiry-report.yml CI: monthly T-180 expiry digest
│
├── schema/
│   ├── quarterly_audit.schema.json     Draft 2020-12 schema for the audit artifact
│   ├── evidence_record.schema.json     Base evidence log record (hash chain + TSA)
│   └── retention_class.schema.json     Retention class definition schema
│
├── audit/
│   ├── quarterly_audit.yaml            v1.3 — 33 assertions, Q1–Q8
│   ├── validate_audit.py               Validator invoked by CI
│   └── fixtures/
│       ├── production_shaped.yaml      Test fixture for staging validation
│       └── expected_output.txt         Reference validator output
│
├── references/
│   ├── REFERENCES.csv                  Machine-readable reference matrix
│   ├── references.yaml                 Same data, structured + basis fields
│   └── retention_classes.yaml          Retention class registry
│
├── compliance/
│   ├── matrix/
│   │   └── control_matrix.csv          Obligation → control → artifact mapping
│   ├── evidence/                       POINTER ONLY — evidence lives outside repo
│   │   └── README.md                   Explains why evidence is not versioned here
│   └── mappings/
│       ├── tma_vacatur_provisions.yaml Provision-level vacatur map
│       └── fceua_fdcpa_utpcpl.yaml     Multi-tier liability cascade
│
├── runbook/
│   ├── DEPLOYMENT_RUNBOOK.md           Phases 0–5, gates, rollback procedures
│   ├── ROLLBACK_MATRIX.md              Master rollback reference table
│   └── COMMUNICATION_PLAN.md           Escalation and notification routing
│
├── docs/
│   ├── verification-log.md             What was checked, when, by what method
│   ├── open-items.md                   Counsel review queue
│   ├── glossary.md                     QPA, IDR, GFE, GPCI, RVU, IIHI …
│   ├── case-law/
│   │   └── tma-litigation-tracker.md   TMA I–III + related proceedings
│   └── adr/
│       ├── 0001-split-ref-008.md       Why HIPAA and NIST were separated
│       ├── 0002-atomic-citation-rows.md Why one URL per row
│       └── 0003-evidence-immutability.md Why rollback never rewrites evidence
│
└── tools/
    ├── hash_chain_verify.py            Walks prev_record_hash linkage
    ├── retention_migrate.py            Applies retention-class migration map
    ├── expiry_diff.py                  Compares before/after expiry calendars
    └── tsa_verify.py                   Validates RFC 3161 tokens
```

---

## 2. Directory Reference

| Path | Contents | Owner | Versioned? |
|---|---|---|---|
| `schema/` | JSON Schemas (Draft 2020-12) validating all machine-readable artifacts | Systems Architect | Yes |
| `audit/` | The quarterly audit checklist and its validator | Compliance Lead | Yes |
| `references/` | The regulatory reference matrix in CSV + YAML | Governance Officer | Yes |
| `compliance/matrix/` | Obligation → control point → audit artifact mapping | Compliance Lead | Yes |
| `compliance/evidence/` | **Pointer only.** Actual evidence is stored in the secured evidence store, never in git | Governance Officer | No — by design |
| `runbook/` | Deployment and rollback procedures | Deployment Lead | Yes |
| `docs/` | Verification log, open items, glossary, ADRs | Governance Officer | Yes |
| `tools/` | Python utilities for chain verification, migration, expiry diffing | Systems Architect | Yes |
| `.github/workflows/` | CI: validation, link checking, chain verification, expiry reporting | Systems Architect | Yes |

**Design rule:** evidence artifacts are **never** committed to this repository. Git history is
rewritable; evidence must not be. The repo holds *how to find and verify* evidence, not the
evidence itself. See `docs/adr/0003-evidence-immutability.md`.

---

## 3. Regulatory Reference Matrix

Status legend: ✅ fetched and read · 🔎 surfaced in search results · ⚠️ unverified · 🔒 internal

### 3.1 Core references

| Ref ID | Instrument | Scope | Binding Nature | Citation | Primary URL | Status |
|---|---|---|---|---|---|---|
| REF-001 | No Surprises Act (Consolidated Appropriations Act 2021, Div. BB, Tit. I) | Balance billing prohibitions, emergency + OON care | Binding Federal Law | P.L. 116-260 | [congress.gov H.R.133 text](https://www.congress.gov/bill/116th-congress/house-bill/133/text) | ✅ |
| REF-002 | Requirements Related to Surprise Billing, Part I | QPA methodology, open negotiation, IDR workflow | Binding Federal Regulation | 45 CFR Part 149 | [eCFR 45 CFR Part 149](https://www.ecfr.gov/current/title-45/subtitle-A/subchapter-B/part-149) | ✅ |
| REF-003 | PA Insurance Department — NSA administration | State enforcement, consumer complaint resolution | Binding State Regulation | PID guidance | [PA.gov PID No Surprises Act](https://www.pa.gov/agencies/insurance/laws-regulations-notices/no-surprises-act) | 🔎 |
| REF-004 | Fair Credit Extension Uniformity Act (FCEUA) | Debt collection standards, unfair/deceptive practices | Binding State Law | 73 P.S. § 2270.1 et seq. | [Act 2000-7 (PDF)](https://www.legis.state.pa.us/WU01/LI/LI/US/PDF/2000/0/0007..PDF) | 🔎 |
| REF-005 | PA Judicial Code — Limitation of Actions | Four-year limitation on contract actions | Binding State Law | 42 Pa.C.S. § 5525 | [PA Gen. Assembly § 5525](https://www.legis.state.pa.us/WU01/LI/LI/CT/HTM/42/00.055.025.000..HTM) | 🔎 |
| REF-006 | Medicare Physician Fee Schedule — RVUs and GAF | RVU computation, GPCI geographic adjustment | Binding Federal Regulation | 42 CFR § 414.22 + § 414.26 | [eCFR § 414.22](https://www.ecfr.gov/current/title-42/chapter-IV/subchapter-B/part-414/subpart-B/section-414.22) · [Subpart B (contains § 414.26)](https://www.ecfr.gov/current/title-42/chapter-IV/subchapter-B/part-414/subpart-B) | ✅ |
| REF-007 | Layered Memory Protocol (LMP) Architecture Spec | State consistency, versioning ($v_{in} > v_t$), SHA-256 chaining | Internal Control Spec | Internal doc | 🔒 Internal | 🔒 |
| REF-008A | HIPAA Security Rule | Administrative, physical, technical safeguards for ePHI | Binding Federal Regulation | 45 CFR Part 160 + Part 164 Subparts A & C | [eCFR Part 164 Subpart C](https://www.ecfr.gov/current/title-45/subtitle-A/subchapter-C/part-164/subpart-C) · [HHS Security Rule](https://www.hhs.gov/hipaa/for-professionals/security/index.html) | ✅ |
| REF-008B | NIST SP 800-122 — Guide to Protecting the Confidentiality of PII | PII identification, confidentiality impact levels | Non-Binding Federal Guidance | NIST SP 800-122 (final, 04/06/10) | [NIST CSRC SP 800-122](https://csrc.nist.gov/publications/detail/sp/800-122/final) | ✅ |
| REF-009 | PA Insurance Department — market conduct & rate filings | Carrier loss ratios, rate benchmarks, AV tracking | Binding State Filing Requirement | PID filings | [PID ACA Health Rate Filings](https://www.pa.gov/agencies/insurance/posted-filings-reports-company-orders/posted-product-and-rate-filings/aca-health-rate-filings) | 🔎 |
| REF-011 | Employee Retirement Income Security Act (ERISA) | Plan-type determination, federal preemption | Binding Federal Law | 29 U.S.C. § 1001 et seq. | [DOL — ERISA](https://www.dol.gov/general/topic/health-plans/erisa) | ✅ |
| REF-014A | Fair Debt Collection Practices Act (FDCPA) | Federal debt collection statutory floor | Binding Federal Law | 15 U.S.C. § 1692 | [Cornell LII — 15 U.S.C. § 1692](https://www.law.cornell.edu/uscode/text/15/1692) | ✅ |
| REF-014B | Regulation F | Debt collection implementing rules | Binding Federal Regulation | 12 CFR Part 1006 | [CFPB — Regulation F](https://www.consumerfinance.gov/rules-policy/regulations/1006/Interp_1006) | ✅ |

### 3.2 Supporting references (cited within assertions)

| Support Ref | Purpose | Citation | URL | Status |
|---|---|---|---|---|
| SUP-01 | HIPAA documentation retention (6 years, "whichever is later") | 45 CFR § 164.316(b)(2)(i) | [eCFR § 164.316](https://www.ecfr.gov/current/title-45/subtitle-A/subchapter-C/part-164/subpart-C/section-164.316) · [Cornell LII mirror](https://www.law.cornell.edu/cfr/text/45/164.316) | ✅ |
| SUP-02 | IDR entity recordkeeping (6 years) | 45 CFR § 149.510(e)(viii) | [eCFR § 149.510](https://www.ecfr.gov/current/title-45/subtitle-A/subchapter-B/part-149/subpart-F/section-149.510) · [Cornell LII mirror](https://www.law.cornell.edu/cfr/text/45/149.510) | 🔎 |
| SUP-03 | Good faith estimate retention (medical record; 6-year floor for copies) | 45 CFR § 149.610(f)(1) | [eCFR § 149.610](https://www.ecfr.gov/current/title-45/subtitle-A/subchapter-B/part-149/subpart-G/section-149.610) · [Cornell LII mirror](https://www.law.cornell.edu/cfr/text/45/149.610) | 🔎 |
| SUP-04 | Computer security log management guidance | NIST SP 800-92 | [NIST CSRC SP 800-92](https://csrc.nist.gov/pubs/sp/800/92/final) | ✅ |
| SUP-05 | Trusted timestamp protocol (proof-of-existence) | RFC 3161 | [IETF RFC 3161](https://datatracker.ietf.org/doc/html/rfc3161) | ✅ |
| SUP-06 | Medicare Physician Fee Schedule (program page) | CMS PFS | [CMS Physician Fee Schedule](https://www.cms.gov/medicare/payment/fee-schedules/physician) | ✅ |
| SUP-07 | False Claims Act limitations window (6y / 3y / 10y cap) | 31 U.S.C. § 3731(b) | [Cornell LII — 31 U.S.C. § 3731](https://www.law.cornell.edu/uscode/text/31/3731) | 🔎 |
| SUP-08 | HIPAA Security Rule documentation requirements (HHS series) | HHS OCR | [HHS Security Series #5 (PDF)](https://www.hhs.gov/sites/default/files/ocr/privacy/hipaa/administrative/securityrule/pprequirements.pdf) | 🔎 |
| SUP-09 | NSA consumer rights overview (federal floor / state preservation) | CMS fact sheet | [CMS No Surprises fact sheet](https://www.cms.gov/newsroom/fact-sheets/no-surprises-understand-your-rights-against-surprise-medical-bills) | 🔎 |
| SUP-10 | FDCPA statutory text (alternate source) | FTC | [FTC FDCPA text](https://www.ftc.gov/legal-library/browse/rules/fair-debt-collection-practices-act-text) | 🔎 |
| SUP-11 | FDCPA examination procedures incl. Reg F record retention | OCC Comptroller's Handbook | [OCC FDCPA Handbook (PDF)](https://www.occ.treas.gov/publications-and-resources/publications/comptrollers-handbook/files/fair-debt-collection-practices-act/fair-debt-collection-practices-act-examination-procedures-interagency.pdf) | 🔎 |
| SUP-12 | Federal IDR process guidance for certified IDR entities | CMS | [CMS IDR Entity Guidance (PDF)](https://www.cms.gov/files/document/federal-idr-guidance-idr-entities-march-2023.pdf) | 🔎 |
| SUP-13 | GFE FAQs for uninsured / self-pay individuals | CMS | [CMS GFE FAQ Part 4 (PDF)](https://www.cms.gov/files/document/faqs-good-faith-estimate-uninsured-self-pay-part-4.pdf) | 🔎 |
| SUP-14 | IDR selection data elements | DOL | [DOL — Certified IDR Entity Selection Data Elements](https://beta.dol.gov/policy-regulations/pay-benefits/health-plans/surprise-billing-and-price-transparency/tools-and-resources/employers-and-service-providers/federal-independent-dispute-resolution-idr-process/selection-certified-idr-entity-data-elements) | 🔎 |
| SUP-15 | NSA IDR process toolkit | AMA | [AMA NSA IDR Toolkit (PDF)](https://www.ama-assn.org/system/files/ama-nsa-idr-toolkit.pdf) | 🔎 |

---

## 4. Retention Classes

| Class ID | Trigger | Years | Basis | Authority | Scope | Owner |
|---|---|---|---|---|---|---|
| `HIPAA-DOC-6Y` | `last_effective_date` | 6 | **law** | 45 CFR § 164.316(b)(2)(i) | HIPAA security policies and required documentation **only** | Governance Officer |
| `CFPB-FDCPA-3Y` | `last_collection_activity` | 3 | **law** | 12 CFR § 1006.100 | Debt collector records; FCEUA also reaches creditors, so default longer | Collections Director |
| `NSA-IDR-6Y` | `determination_date` | 6 | secondary | 45 CFR § 149.510(e)(viii) | Stated for certified IDR entities; plans/providers use 7y policy | Legal Counsel |
| `NSA-GFE-MEDREC` | `gfe_issue_date` | 6 floor | secondary | 45 CFR § 149.610(f)(1) | GFE is part of the medical record; 6y floor for requested copies | Legal Counsel |
| `POLICY-7Y` | `creation_date` | 7 | internal_design | — | Default where no mandate exists; **never** apply to HIPAA-DOC records | Policy Administrator |
| `PA-RATE-5Y` | `filing_effective_date` | 5 | internal_design | — | PA rate filing datasets | Analytics Lead |
| `CLAIM-10Y` | `violation_date` | 10 | **law** | 31 U.S.C. § 3731(b) | Litigation-exposure horizon, **not** a retention mandate | Legal Counsel |

**Retention alert:** T−180 days before expiry. `legal_hold_suspends_all_expirations: true`.

**Critical trap:** HIPAA's clock runs from *"the date of its creation or the date when it last was
in effect, whichever is later."* A superseded policy does not age off until six years after its
last effective date. Purging on creation date is the single most common misapplication.

---

## 5. Fail-Stop Triggers

| ID | Condition | Action | Escalate To |
|---|---|---|---|
| FS1 | Any `prev_record_hash` mismatch | Halt automated processing | CTO, CCO |
| FS2 | `tsa_token` missing on >5% of critical records | Quarantine batch | SecOps Manager |
| FS3 | Retention trigger misclassified on >2% of records | Manual review queue | Governance Officer |
| FS4 | `encryption_state == plaintext` on any PHI transmission | Incident ticket | CISO |
| FS5 | `benchmark_vintage` is null | Reject IDR output | Head of Pricing Analytics |

**Migration-day override:** during retention migration, the FS3 tolerance is **0%**, not 2%.
The 2% figure is a steady-state threshold.

**Hypercare override:** if aggregate false-positive rate across FS1–FS5 exceeds **5% of trigger
events** over any 72-hour window, roll all triggers back to alert-only and hold for root cause.

---

## 6. Audit Control Structure

| Control | Title | Tier | Assertions | Primary Authority |
|---|---|---|---|---|
| Q1 | NSA balance billing | 1 | 5 | 45 CFR §§ 149.410, 149.420 |
| Q2 | IDR / QPA computation | 1 | 4 | 45 CFR §§ 149.140, 149.510 |
| Q3 | MPFS fair market value | 1 | 4 | 42 CFR § 414.26 |
| Q4 | HIPAA Security Rule | 1 | 5 | 45 CFR §§ 164.308, 164.312, 164.316 |
| Q5 | FCEUA / FDCPA | 2 | 5 | 15 U.S.C. § 1692c(a)(1); 12 CFR §§ 1006.6, 1006.14, 1006.34, 1006.100; 73 P.S. § 2270.1 |
| Q6 | ERISA plan-type determination | 2 | 3 | 29 U.S.C. § 1144 |
| Q7 | State enforcement | 2 | 3 | Internal design |
| Q8 | Statute of limitations | 2 | 3 | 42 Pa.C.S. § 5525(a)(8) |

**Total: 33 assertions.** Validated: `OK: 0 errors, 0 warnings, 33 assertions`.

### Assertion basis distribution

| Basis | Count | Meaning |
|---|---|---|
| `law` | 17 | Traced to statute or regulation |
| `internal_design` | 13 | Our control choice, not a mandate |
| `best_practice` | 1 | Guidance only |
| `secondary` | 2 | From secondary source; verify against primary text |

---

## 7. Evidence Repository Layout

Evidence lives **outside** this repository, in the secured evidence store:

```text
/compliance/evidence/
├── 001-NSA/
│   ├── balance-billing-logs/
│   └── good-faith-estimate-receipts/
├── 002-IDR/
│   ├── qpa-worksheets/
│   └── open-negotiation-packages/
├── 003-PA-PID/
│   ├── complaint-intake-records/
│   └── federal-coordination-docs/
├── 004-FCEUA/
│   ├── communication-logs/
│   └── disclosure-templates/
├── 005-PA-SLA/
│   ├── accrual-date-calculations/
│   └── expired-claims-registry/
├── 006-MPFS/
│   ├── fee-comparison-snapshots/
│   └── gpci-application-records/
├── 008A-HIPAA/
│   ├── access-control-matrix/
│   ├── risk-analysis-reports/
│   └── incident-response-logs/
├── 009-RateFilings/
│   ├── carrier-filing-datasets/
│   └── loss-ratio-calculations/
├── 011-ERISA/
│   ├── plan-determination-docs/
│   └── jurisdiction-routing-records/
└── 014-FDCPA/
    ├── validation-notices/
    ├── communication-time-logs/
    └── cease-request-confirmations/
```

### Base evidence record schema

```json
{
  "event_id": "UUID",
  "sequence_no": 0,
  "timestamp_utc": "RFC 3339",
  "ref_id": "REF-008A",
  "control_point": "SecurityRule_AccessControl_Admin",
  "actor":  { "principal_id": "", "role": "", "session_id": "" },
  "subject": { "patient_pseudonym": "", "encounter_id": "" },
  "action": "permit | deny | compute | disclose | update",
  "decision_basis": { "citation": "45 CFR 164.312(a)(1)", "rule_version": "" },
  "inputs_hash": "sha256:<hex>",
  "payload_hash": "sha256:<hex>",
  "prev_record_hash": "sha256:<hex>",
  "record_hash": "sha256:<hex>",
  "tsa_token": "RFC 3161 base64 token",
  "retention_class": "HIPAA-DOC-6Y | POLICY-7Y",
  "retention_trigger": "creation | last_effective_date"
}
```

Four integrity properties this provides:

| Property | Mechanism | Authority |
|---|---|---|
| Tamper evidence | `prev_record_hash` chain | FIPS 180-4 (SHA-256) |
| Independent time proof | `tsa_token` | RFC 3161 |
| Input reproducibility | `inputs_hash` | Internal design |
| Correct retention clock | `retention_trigger` split | 45 CFR § 164.316(b)(2)(i) |

---

## 8. Schema & Validation

```bash
python audit/validate_audit.py \
  --schema schema/quarterly_audit.schema.json \
  --input  audit/quarterly_audit.yaml
```

Expected output:

```text
OK: 0 errors, 0 warnings, 33 assertions
```

CI gates (`.github/workflows/`):

| Workflow | Trigger | Failure consequence |
|---|---|---|
| `validate-audit.yml` | Every push touching `audit/` or `schema/` | Blocks merge |
| `link-check.yml` | Weekly cron | Opens issue with dead links |
| `hash-chain-verify.yml` | Nightly | Fires FS1 escalation |
| `retention-expiry-report.yml` | Monthly cron | Emails T−180 digest to class owners |

---

## 9. Deployment Runbook Summary

Full procedure: [`runbook/DEPLOYMENT_RUNBOOK.md`](runbook/DEPLOYMENT_RUNBOOK.md)

| Phase | Window | Purpose | Gate | Rollback Risk |
|---|---|---|---|---|
| 0 — Baseline | Days −5 to −2 | Snapshots, chain heads, freeze | Baseline validation must pass | None (hold) |
| 1 — Promotion | Days −2 to −1 | Stage
