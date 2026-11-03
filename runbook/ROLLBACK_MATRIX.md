# Rollback Matrix and Evidence-Preservation Runbook

- **Status:** Draft — validate against deployed systems and approved incident procedures before production use
- **Owner:** Deployment Lead, with Security and Compliance consultation
- **Related decision:** [ADR-0003: Preserve Evidence During Deployment and Rollback](../docs/ADR/0003-evidence-immutability.md)

> This runbook is an operational template, not legal advice. Confirm system-specific commands, approvals, notification routes, retention rules, and recovery capabilities before relying on it. Do not put evidence payloads, PHI, secrets, or credentials in this repository or in CI artifacts.

## Governing rules

1. Separate application release state from evidence-store state.
2. A normal code rollback must not delete, truncate, rewrite, or restore over evidence.
3. Preserve originals when integrity or completeness is in doubt. Investigate using access-controlled copies; document who created and examined each copy.
4. Treat an integrity-check result as a signal requiring investigation, not by itself as proof of loss, tampering, or legal compliance.
5. Legal holds and approved retention/disposition procedures continue to apply. Do not dispose of or alter records to resolve an operational incident.
6. Use only deployment commands and evidence-store procedures approved for the actual environment. This matrix intentionally does not assume a cloud vendor, database, or storage API.

## Rollback decision matrix

| Trigger | Immediate containment | Recovery target and action | Evidence-preservation requirements | Minimum authorization / escalation |
|---|---|---|---|---|
| CI validation or link-check failure before release | Block the merge or release; capture the failed check output without sensitive payloads | Correct the change and rerun required checks; no production rollback is needed if nothing was deployed | Keep the failed run metadata and reviewable source changes under normal repository controls; never upload evidence payloads to CI | Repository maintainers; Compliance owner for compliance-data changes |
| Application regression after release | Follow the incident process; disable or limit the affected feature if safe | Roll back the application artifact to a known-good release using the approved platform procedure; verify service behavior | Do not run evidence-schema rollback or destructive data cleanup as part of code rollback; record release IDs, times, operator, and checks | Incident Lead and Deployment Lead; involve Security/Compliance if evidence processing was affected |
| Hash-chain or integrity check reports a mismatch | Stop or constrain automated processing that depends on the affected data; restrict access as needed and preserve logs | Preserve the source and relevant metadata; verify the checker, inputs, and scope; investigate on a controlled copy. Do not reset the chain or restore over the source | Record affected identifiers and verification output without placing sensitive payloads in Git; preserve the original source and any relevant snapshots | Security Lead and Compliance Officer; notify Legal Counsel under the approved incident plan |
| Schema mismatch or invalid audit artifact | Reject or quarantine the affected input according to the deployed control; retain it in the secured evidence environment | Revert application/schema configuration only through a compatible, reviewed release; validate a corrected or replayed artifact without modifying the original | Preserve the original input, validation result, schema version, and any replay as distinct records; do not silently rewrite historical artifacts | Compliance owner and Deployment Lead; Security/Legal consultation if regulated records may be affected |
| Retention migration fails or results are uncertain | Pause the migration and any associated disposition; preserve job logs, input/output counts, and configuration | Return processing to a previously approved operational mode only if safe; assess affected records from preserved copies and use a reviewed forward-fix or recovery plan | Do not delete, overwrite, or “roll back” rows in place to conceal partial progress; retain migration attempts and record any correction as a new event | Data/System owner and Compliance Officer; Legal Counsel if a hold, expiry, or disposition decision is implicated |
| Evidence store unavailable or suspected compromise | Follow incident response; preserve access and system logs; prevent unapproved writes where feasible | Restore service only using the approved continuity/recovery plan after the source and recovery point have been assessed | Preserve originals and recovery artifacts; document source, timestamps, custody, and integrity checks; never assume a prior snapshot includes all later records | Security Lead and Incident Lead; escalate to Compliance and Legal under the incident plan |

## Procedure

### 1. Declare and contain

- Open an incident record and assign an Incident Lead.
- Record the trigger, detection time, affected service or data scope, current release/configuration, and actions taken.
- Preserve relevant logs and metadata in approved secured systems. Do not copy PHI, evidence payloads, credentials, or secrets into GitHub issues, chat, or workflow artifacts.
- Restrict processing or access when needed to prevent further impact, following the approved incident process.

### 2. Select the recovery path

- Determine whether the incident concerns application code, schema/configuration, evidence storage, or more than one of these.
- For an application-only regression, use the platform's approved release rollback and leave evidence state unchanged.
- For a suspected evidence integrity or retention issue, do not perform a normal data rollback. Preserve the source and consult Security, Compliance, and Legal before any recovery operation that could affect records.
- If the required tool or procedure is unavailable, stop and escalate rather than improvising a destructive operation.

### 3. Execute and record

- Use a second-person review for high-impact production recovery where the approved incident procedure requires it.
- Record the operator, approver, command or change identifier, start/end times, before/after release or configuration identifiers, scope, and outcome in the secured operational record.
- Keep evidence payloads in the designated evidence store. Store only non-sensitive references and procedural documentation in this repository.

### 4. Verify and close

- Confirm service behavior and that the recovery did not alter the evidence source.
- Run the approved integrity verification against the intended scope. If the referenced verifier is unavailable or not yet implemented, mark verification as pending and escalate; do not report success based on an unrun command.
- Retain verification output and incident decisions in the approved secured record system.
- Obtain incident-owner approval before restoring normal automated processing. Document unresolved scope, corrective actions, and required follow-up reviews.

## Production-readiness checklist

Before adopting this matrix, document and test:

- The actual application deployment and rollback mechanism, with least-privilege roles.
- Evidence-store owners, access controls, preservation method, backup/recovery behavior, and legal-hold handling.
- Which integrity tools exist, their accepted inputs/outputs, and where reports are securely retained.
- Incident contacts and notification timelines from the approved communication plan.
- How to test recovery without using production PHI or overwriting the evidence source.
- Review and approval requirements for retention changes, restoration, and disposition.
