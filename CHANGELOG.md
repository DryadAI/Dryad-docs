# CHANGELOG — dryad-docs

Append-only history of changes to the dryad-docs repository.

---

## [2026-04-19] — Audit Gap Closure (Forensic Integrity Fix)

**Type:** [FIX] [REPO_INIT]
**Authors:** Claude (Architectural Lead, build), <human reviewer>
**Affects:** All Schools, All Pillars
**Status:** Canonical

### Context

Path-2 audit (April 19, 2026) found that the prior `[2026-04-19] — Part 1 v1.1 Foundation Drop + Repo Initialization` entry claimed certain files existed when they did not. This violated R15 ("when rules conflict with reality, reality wins"). This entry documents and closes that gap by making reality match the prior claim.

### Files Created (closing prior gap)

- `schools/governance/README.md` v1.0
- `schools/execution/README.md` v1.0
- `subprojects/_template/` directory with full template (README, SOUL, AGENTS, GOAL, CLAUDE, CHANGELOG, docs/.gitkeep, src/.gitkeep, tests/.gitkeep, audit/timeline.jsonl)
- `meta/REPO_CONVENTIONS.md` v1.0
- `meta/SEMANTIC_LINKS.md` v1.0
- `meta/TEMPORAL_TIMELINE.md` v1.0
- `archive/README.md`
- `CONTRIBUTING.md`
- `SEMANTIC_LINKS.md` (root pointer)
- `schemas/README.md`
- `schemas/timeline.jsonl.schema.json`
- `schemas/trace.schema.json` (placeholder)
- `schemas/audit-entry.schema.json` (placeholder)
- `schemas/promotion-proposal.schema.json` (placeholder)
- `.github/PULL_REQUEST_TEMPLATE.md`
- `.github/CODEOWNERS`
- `.github/workflows/markdown-lint.yml`
- `.github/workflows/link-check.yml`

### Files Edited

- `core/PILLARS.md` Section 14: removed stale "once Gemini template lands" reference

### Forensic Note

The prior entry remains as written. This `[FIX]` entry is the corrective record. Per repo convention, prior CHANGELOG entries are never edited, only superseded.

---

## [2026-04-19] — Part 1 v1.1 Foundation Drop + Repo Initialization

**Type:** [INIT]
**Authors:** Nathan Modlin (Provost), Claude (Architectural Lead)
**Affects:** All Schools, All Pillars
**Status:** Canonical

### Added
- Initial repository structure.
- Canonical documents in `core/` (PILLARS, SOUL, DESIGN_RULES, AGENTS, SUBPROJECT_PATTERN, ZERO_TRUST_POSTURE).
- README.md and dryad-system-reference.md.
- schools/memory/README.md (canonical School template).

---
