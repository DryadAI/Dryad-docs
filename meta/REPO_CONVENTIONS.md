# REPO_CONVENTIONS.md — dryad-docs Conventions

**School:** Cross-cutting
**Pillar(s):** Communication
**Version:** 1.0
**Date:** 2026-04-19
**Authors:** Nathan Modlin (Provost), Claude (Architectural Lead)
**Status:** Canonical
**PARENT:** dryad-docs://core/SOUL.md

---

## Naming Conventions

### Files
- All caps for foundational documents: `PILLARS.md`, `SOUL.md`, `AGENTS.md`, `GOVERNANCE.md`
- Lowercase with hyphens for routine docs: `markdown-lint.yml`, `pull-request-template.md`
- Numeric prefix for ordered sets: `01_PART1_GAP_ANALYSIS.md`, `02_PART1_MANIFEST.md`

### Directories
- Lowercase, no hyphens: `core/`, `roots/`, `schools/`, `subprojects/`, `meta/`, `archive/`, `schemas/`
- Sub-project directories: descriptive, lowercase, hyphens-allowed: `subprojects/neo-lyceum/`

### Document Headers
Every markdown document MUST start with:
```markdown
# <Title> — <Subtitle if needed>

**School:** <Memory | Governance | Execution | Cross-cutting>
**Pillar(s):** <Infrastructure | Communication | Philosophy | Multiple>
**Version:** <semver>
**Date:** YYYY-MM-DD
**Authors:** <names>
**Status:** <Canonical | Proposed | Deprecated | Archived>
**PARENT:** dryad-docs://<parent path>
```

## Commit Format

```
[<Type>] <School: X> <Pillar: Y> <short summary>

<longer description if needed>

Affects: <Schools>; <Pillars>
```

Type tags: `[CORE]`, `[ROOTS]`, `[SCHOOL: X]`, `[SUBPROJECT: X]`, `[META]`, `[ARCHIVE]`, `[FIX]`, `[CHORE]`, `[MILESTONE]`, `[PIVOT]`.

## PR Requirements

- Signed commits (GPG or Sigstore)
- Branch from main, PR back to main
- At least 1 approval from `provost` or `dev` team
- CI passes (markdown lint + link check)
- CHANGELOG.md entry appended

## Versioning

Documents use semantic-ish versioning:
- 1.0 — first canonical
- 1.1, 1.2 — minor changes (clarifications, additions)
- 2.0 — major reformulation (rare; requires Provost approval)

## Archival

When a document is superseded, move it to `archive/<original-path>` with a CHANGELOG `[ARCHIVE]` entry. Never delete.

## Cross-References

Use `dryad-docs://` semantic links between documents (see SEMANTIC_LINKS.md). Plain relative paths are acceptable for adjacent files.

---

**End of REPO_CONVENTIONS.md v1.0**
