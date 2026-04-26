# <SUB-PROJECT NAME>

**Pillar(s):** <Infrastructure | Communication | Philosophy | Multiple>
**School(s):** <Memory | Governance | Execution | Cross-cutting>
**Version:** 0.1
**Date:** YYYY-MM-DD
**Status:** Active | Proposed | Deprecated
**Authors:** <names>
**PARENT:** dryad-docs://core/PILLARS.md

> *"<one-sentence purpose>"*

---

## What This Sub-Project Is

<2–4 paragraph description>

## Why This Sub-Project Exists

<connection back to PILLARS.md framework — which School(s)/Pillar(s) does this serve?>

## Quick Start

### Read Before Touching
1. dryad-docs://core/PILLARS.md
2. dryad-docs://core/SOUL.md
3. dryad-docs://core/AGENTS.md
4. This sub-project's SOUL.md (if it diverges from core)
5. This sub-project's AGENTS.md
6. This sub-project's current GOAL.md

### Run Locally
```bash
# placeholder
```

## Directory Structure
```
<sub-project-name>/
├── README.md           — you are here
├── SOUL.md             — inherits from core; overrides only where divergent
├── AGENTS.md           — sub-project commands & constraints
├── GOAL.md             — current task
├── CLAUDE.md           — Claude Code session pointer (often symlinks to AGENTS.md)
├── CHANGELOG.md        — sub-project history
├── docs/               — sub-project documentation
├── src/                — source code
├── tests/              — test suite
└── audit/
    └── timeline.jsonl  — temporal forensic log (append-only)
```

## Audit Trail

Every action on this sub-project is logged to `audit/timeline.jsonl`. See `dryad-docs://meta/TEMPORAL_TIMELINE.md` for the schema.

Query examples:
```bash
# Show all actions today
jq 'select(.timestamp >= "YYYY-MM-DDT00:00:00Z")' audit/timeline.jsonl

# Show actions by author
jq 'select(.actor == "Nathan")' audit/timeline.jsonl
```

## Sub-Project Conventions

This sub-project follows `dryad-docs://core/SUBPROJECT_PATTERN.md`. Any deviation must be documented in this README and approved by the Provost.

## License

<inherited from parent>

---

**Maintained by:** <author(s)>
**Last updated:** <date>
