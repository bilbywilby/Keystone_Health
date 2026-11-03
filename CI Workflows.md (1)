# GitHub Actions Workflow Definitions

Below are the workflow definitions designed to run against the scaffolded files and enforce link integrity and audit validation.

---

### `.github/workflows/validate-audit.yml`

```yaml
name: Validate Audit Record & Schemas

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

jobs:
  validate-schema-and-references:
    runs-on: ubuntu-latest
    steps:
      - name: Check out repository
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          if [ -f requirements.txt ]; then pip install -r requirements.txt; fi

      - name: Validate baseline audit schema against validator stub
        run: |
          # If a sample quarterly payload exists, validate it; otherwise validate schema path accessibility
          if [ -f audit/sample_quarterly_audit.json ]; then
            python audit/validate_audit.py --payload audit/sample_quarterly_audit.json --schema schema/quarterly_audit.schema.json
          else
            echo "[*] No sample audit payload found. Running structural schema verification step."
            python -c "import json; json.load(open('schema/quarterly_audit.schema.json'))"
            echo "[+] Schema JSON syntax is valid."
          fi
```

---

### `.github/workflows/link-check.yml`

```yaml
name: Check Links and Markdown References

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]
  schedule:
    # Run weekly on Sundays at 00:00 UTC to catch broken external citations
    - cron: '0 0 * * 0'

jobs:
  link-checker:
    runs-on: ubuntu-latest
    steps:
      - name: Check out repository
        uses: actions/checkout@v4

      - name: Link Checker
        uses: lycheeverse/lychee-action@v1.9.0
        with:
          args: --verbose --no-progress 'docs/adr/*.md' 'README.md' 'runbook/*.md'
          fail: true
```