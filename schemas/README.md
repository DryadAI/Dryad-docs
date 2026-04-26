# schemas/

**Purpose:** Canonical machine-readable schemas (JSON Schema, YAML) for dryad-docs and Dryad runtime data structures.

## Files

| Schema | Purpose | Authority |
|--------|---------|-----------|
| `timeline.jsonl.schema.json` | Sub-project temporal forensic log | dryad-docs://meta/TEMPORAL_TIMELINE.md |
| `trace.schema.json` | Structured per-episode trace | dryad-docs://core/TRACE_FORMAT.md |
| `audit-entry.schema.json` | Redis audit stream entry | dryad-docs://core/GOVERNANCE.md |
| `promotion-proposal.schema.json` | Curator → Cowork handoff | dryad-docs://schools/memory/README.md |

## Maintenance

- Schemas are machine-readable. Their human-readable counterparts live in `core/` or `meta/`.
- When the human-readable doc changes, the schema must be updated in the same PR.
- Schema versioning is tracked in the `$id` field.

---

**Last updated:** 2026-04-19
