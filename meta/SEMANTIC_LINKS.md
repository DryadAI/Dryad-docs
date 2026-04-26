# SEMANTIC_LINKS.md — dryad-docs:// Anchor Specification

**School:** Cross-cutting
**Pillar(s):** Communication
**Version:** 1.0
**Date:** 2026-04-19
**Authors:** Nathan Modlin (Provost), Claude (Architectural Lead)
**Status:** Canonical
**PARENT:** dryad-docs://core/SOUL.md

---

## What dryad-docs:// Is

A **semantic anchor scheme** for cross-referencing documents within and beyond the dryad-docs repository. It enables future agents and humans to traverse the knowledge graph from any starting point back to canonical truth.

## Format

```
dryad-docs://<path>[#<anchor>]
```

Examples:
- `dryad-docs://core/PILLARS.md` — entire document
- `dryad-docs://core/PILLARS.md#3-the-pillars--schools-matrix-framing-c` — specific section
- `dryad-docs://schools/memory/README.md` — School README
- `dryad-docs://subprojects/neo-lyceum/AGENTS.md` — sub-project agents file

## Resolution Rules

1. The path after `dryad-docs://` is interpreted relative to the dryad-docs repo root.
2. Anchors after `#` follow GitHub-flavored markdown anchor rules (lowercase, hyphens for spaces, special characters dropped).
3. If a path resolves to a directory, the implicit target is `README.md` in that directory.

## Required Use Cases

Sub-project READMEs **MUST** declare their parent with a top-of-document directive:

```markdown
**PARENT:** dryad-docs://core/PILLARS.md
```

This enables an agent landing in any sub-project to walk back to core in a single hop.

## Optional Use Cases

- Inline references: "See dryad-docs://core/DESIGN_RULES.md#r14"
- Cross-school references: "Memory's interpretation diverges from dryad-docs://schools/governance/README.md"
- Schema references: "Per dryad-docs://schemas/trace.schema.json"

## Validation

In Part 1: convention-only enforcement. Authors maintain consistency manually.

In Part 2: a CI check (`scripts/validate-semantic-links.py`) will:
- Parse all markdown files for `dryad-docs://` references
- Verify each resolves to an existing file/anchor
- Fail the PR if any link is broken

## External References

For references **outside** the dryad-docs repo (e.g., to DryadAI/dryad-core code), use plain HTTPS URLs. The `dryad-docs://` scheme is internal-only.

---

**End of SEMANTIC_LINKS.md v1.0**
