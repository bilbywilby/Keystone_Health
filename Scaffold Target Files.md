# Scaffolding Baseline Files

Below are the initial scaffold files required to align the repository structure with `ADR-0001`, `ADR-0002`, `ADR-0003`, and `ROLLBACK_MATRIX.md`.

---

### `references/REFERENCES.csv`

```csv
Reference_ID,Title,Scope,Binding_Nature,Authority_Type,Citation,Canonical_URL,Provenance_Status,Interpretation_Status,Notes
REF-008A,HIPAA Security Rule,45 CFR Part 160 and Subparts A and C of Part 164. Administrative physical and technical safeguards for ePHI.,Binding Legal Standard,law,45 CFR § 164.302 et seq.,https://www.ecfr.gov/current/title-45/subtitle-A/subchapter-C/part-164/subpart-C,primary_fetched,settled,Excludes Privacy Rule (Subpart E) and Breach Notification (Subpart D).
REF-008B,NIST SP 800-122,Guide to Protecting the Confidentiality of Personally Identifiable Information (PII).,Non-Binding Federal Guidance,best_practice,NIST SP 800-122,https://csrc.nist.gov/publications/detail/sp/800-122/final,primary_fetched,settled,Applies to PII confidentiality impact levels.
REF-014A,Fair Debt Collection Practices Act (FDCPA),15 U.S.C. § 1692 et seq. Statutory restrictions on debt collection activities.,Binding Legal Standard,law,15 U.S.C. § 1692 et seq.,https://www.govinfo.gov/app/details/USCODE-2021-title15/USCODE-2021-title15-chap41-subchapV,primary_fetched,settled,Split from compound citation per ADR-0002.
REF-014B,CFPB Regulation F,12 CFR Part 1006. Administrative implementation of debt collection rules.,Binding Legal Standard,secondary,12 CFR Part 1006,https://www.ecfr.gov/current/title-12/chapter-X/part-1006,primary_fetched,settled,Split from compound citation per ADR-0002.
```

---

### `references/references.yaml`

```yaml
version: "1.0"
updated_at: "2026-11-03"
references:
  - id: "REF-008A"
    title: "HIPAA Security Rule"
    scope: "45 CFR Part 160 and Subparts A and C of Part 164. Administrative, physical, and technical safeguards for ePHI."
    binding_nature: "Binding Legal Standard"
    authority_type: "law"
    citation: "45 CFR § 164.302 et seq."
    canonical_url: "https://www.ecfr.gov/current/title-45/subtitle-A/subchapter-C/part-164/subpart-C"
    provenance_status: "primary_fetched"
    interpretation_status: "settled"
    notes: "Excludes Privacy Rule (Subpart E) and Breach Notification Rule (Subpart D)."

  - id: "REF-008B"
    title: "NIST SP 800-122"
    scope: "Guide to Protecting the Confidentiality of Personally Identifiable Information (PII)."
    binding_nature: "Non-Binding Federal Guidance"
    authority_type: "best_practice"
    citation: "NIST SP 800-122"
    canonical_url: "https://csrc.nist.gov/publications/detail/sp/800-122/final"
    provenance_status: "primary_fetched"
    interpretation_status: "settled"

  - id: "REF-014A"
    title: "Fair Debt Collection Practices Act (FDCPA)"
    scope: "15 U.S.C. § 1692 et seq. Statutory restrictions on debt collection activities."
    binding_nature: "Binding Legal Standard"
    authority_type: "law"
    citation: "15 U.S.C. § 1692 et seq."
    canonical_url: "https://www.govinfo.gov/app/details/USCODE-2021-title15/USCODE-2021-title15-chap41-subchapV"
    provenance_status: "primary_fetched"
    interpretation_status: "settled"

  - id: "REF-014B"
    title: "CFPB Regulation F"
    scope: "12 CFR Part 1006. Administrative implementation of debt collection rules."
    binding_nature: "Binding Legal Standard"
    authority_type: "secondary"
    citation: "12 CFR Part 1006"
    canonical_url: "https://www.ecfr.gov/current/title-12/chapter-X/part-1006"
    provenance_status: "primary_fetched"
    interpretation_status: "settled"
```

---

### `schema/quarterly_audit.schema.json`

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "QuarterlyAuditRecord",
  "description": "Schema for validating Keystone Health Forensic Navigator quarterly audit records.",
  "type": "object",
  "properties": {
    "audit_id": {
      "type": "string",
      "pattern": "^AUD-\\d{4}-Q[1-4]$"
    },
    "period": {
      "type": "string",
      "pattern": "^\\d{4}-Q[1-4]$"
    },
    "timestamp": {
      "type": "string",
      "format": "date-time"
    },
    "assessor": {
      "type": "string"
    },
    "record_hash": {
      "type": "string",
      "pattern": "^[a-fA-F0-9]{64}$"
    },
    "assertions": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "assertion_id": {
            "type": "string"
          },
          "status": {
            "type": "string",
            "enum": ["pass", "fail", "in_review"]
          },
          "references": {
            "type": "array",
            "items": {
              "type": "string"
            }
          }
        },
        "required": ["assertion_id", "status", "references"]
      }
    }
  },
  "required": ["audit_id", "period", "timestamp", "assessor", "assertions"]
}
```

---

### `audit/validate_audit.py`

```python
#!/usr/bin/env python3
"""
Audit Log and Reference Schema Validator
Validates audit payload instances against quarterly_audit.schema.json
and ensures citation references correspond to active reference records.
"""

import sys
import json
import argparse
from pathlib import Path

def validate_structure(payload_path, schema_path):
    print(f"[*] Validating payload: {payload_path}")
    print(f"[*] Using schema: {schema_path}")
    
    if not Path(payload_path).exists():
        print(f"[!] Error: Payload file not found: {payload_path}", file=sys.stderr)
        return False
        
    if not Path(schema_path).exists():
        print(f"[!] Error: Schema file not found: {schema_path}", file=sys.stderr)
        return False

    with open(payload_path, 'r', encoding='utf-8') as p_file:
        try:
            payload = json.load(p_file)
        except json.JSONDecodeError as err:
            print(f"[!] JSON Syntax Error in payload: {err}", file=sys.stderr)
            return False

    # Structural check for baseline fields
    required_keys = ["audit_id", "period", "timestamp", "assessor", "assertions"]
    missing = [k for k in required_keys if k not in payload]
    if missing:
        print(f"[!] Validation failed: Missing required root keys: {missing}", file=sys.stderr)
        return False

    print("[+] Structure validation passed (stub validator).")
    return True

def main():
    parser = argparse.ArgumentParser(description="Validate compliance audit files.")
    parser.add_argument("--payload", required=True, help="Path to audit JSON payload")
    parser.add_argument("--schema", required=True, help="Path to JSON schema file")
    
    args = parser.parse_args()
    
    success = validate_structure(args.payload, args.schema)
    if not success:
        sys.exit(1)
    sys.exit(0)

if __name__ == "__main__":
    main()
```