# TEMPORAL_TIMELINE.md — Forensic Log Format Specification

**School:** Cross-cutting (Governance authority; Execution producer)
**Pillar(s):** Communication
**Version:** 1.0
**Date:** 2026-04-19
**Authors:** Nathan Modlin (Provost), Claude (Architectural Lead)
**Status:** Canonical
**PARENT:** dryad-docs://core/SOUL.md

---

## Purpose

Every Dryad sub-project maintains a **temporal forensic log** at `audit/timeline.jsonl`. This is the chronological ground truth of what happened, when, and why. It supports post-incident forensic analysis and identity-drift detection.

## Format

JSONL (newline-delimited JSON). One event per line. Append-only. No deletes, no edits.

The first line of every timeline.jsonl is a schema header (see schema below). All subsequent lines are events.

## Event Schema

```json
{
  "timestamp": "2026-04-19T14:30:00Z",
  "event_id": "evt_<uuid>",
  "actor": "<name or service id>",
  "actor_role": "human | agent | system",
  "action": "<verb_in_snake_case>",
  "target": "<file path | service | concept>",
  "school": "Memory | Governance | Execution | Cross-cutting",
  "pillar": "Infrastructure | Communication | Philosophy | Multiple",
  "details": { },
  "outcome": "success | failure | partial",
  "audit_ref": "<optional Redis stream entry id>"
}
```

Required fields: `timestamp`, `event_id`, `actor`, `action`, `target`, `outcome`.

## Event Categories

- **Sub-project lifecycle:** `created`, `archived`, `revived`
- **File operations:** `file_created`, `file_modified`, `file_deleted` (deletion is rare; archival preferred)
- **Build operations:** `build_started`, `build_completed`, `build_failed`
- **Decisions:** `decision_made`, `escalation_raised`, `vote_cast`
- **Audit:** `gate_passed`, `gate_failed`, `rollback_invoked`
- **External integration:** `external_call_made`, `external_dep_added`

## Reading the Timeline

```bash
# Pretty-print today's events
jq -r 'select(.timestamp | startswith("2026-04-19")) | "\(.timestamp) \(.actor) \(.action) \(.target)"' audit/timeline.jsonl

# Find all decisions
jq 'select(.action == "decision_made")' audit/timeline.jsonl

# Find failures
jq 'select(.outcome == "failure")' audit/timeline.jsonl

# Find specific actor
jq 'select(.actor == "Nathan")' audit/timeline.jsonl
```

## Writing to the Timeline

```bash
# Use the dryad CLI when available
dryad audit log --school <s> --action <a> --target <t> --outcome success

# Or append directly (for build scripts)
echo '{"timestamp": "2026-04-19T14:30:00Z", "event_id": "evt_abc123", "actor": "Claude", ...}' >> audit/timeline.jsonl
```

## Retention

- Indefinite for the timeline itself
- Once timeline grows beyond 10MB, oldest entries archived to `audit/timeline-<year>.jsonl.gz` (Part 2 work)
- Never deleted

## Relationship to Redis Audit Streams

The Redis streams `audit:memory`, `audit:governance`, `audit:execution` are the **live** audit trail (last 90 days). The `timeline.jsonl` is the **persistent** sub-project trail. Important Redis events should be mirrored to the appropriate timeline.

## Validation

Part 1: convention only.
Part 2: CI check that timeline.jsonl conforms to schema and is append-only (no edits to prior lines).

---

**End of TEMPORAL_TIMELINE.md v1.0**
