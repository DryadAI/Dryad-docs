---
type: reference
node: all
date: 2026-05-18
status: active
---

# Frontmatter Schema

All notes in dryad-vault use this contract.

## Required fields

- type: changelog | runbook | post-mortem | session | rule | reference | spec
- node: madhatter | dryad-forge | dryad-gate | cluster | precision | all
- date: YYYY-MM-DD
- status: draft | active | archived | superseded

## Optional fields

- id: CHG-NNN | RUN-NNN | PM-NNN
- affects: [list of stacks]

## Type definitions

- changelog: CHG-NNN entries. Append-only.
- runbook: Operational procedures.
- post-mortem: Incident retrospectives.
- session: Work session notes (Scribe writes these in Phase 2).
- rule: R-series rules.
- reference: Static facts (network, ports).
- spec: Design docs.
