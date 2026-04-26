# CONTRIBUTING.md

**Last updated:** 2026-04-19

This file describes how to contribute to dryad-docs.

## Required Reading

Before opening a PR, read:

1. [README.md](README.md)
2. [core/PILLARS.md](core/PILLARS.md) — the framework
3. [core/SOUL.md](core/SOUL.md) — identity & rules
4. [core/ZERO_TRUST_POSTURE.md](core/ZERO_TRUST_POSTURE.md) — security posture
5. [meta/REPO_CONVENTIONS.md](meta/REPO_CONVENTIONS.md) — naming, headers, commits
6. [meta/SEMANTIC_LINKS.md](meta/SEMANTIC_LINKS.md) — dryad-docs:// anchors

## Workflow (Zero-Trust Option B)

1. Branch from main: `git checkout -b <type>/<short-description>`
2. Make changes following REPO_CONVENTIONS.md
3. Append CHANGELOG.md entry
4. Sign commits (GPG or Sigstore): `git commit -S`
5. Push branch and open PR
6. Request review from `@provost` or `@dev` team
7. Address review feedback
8. CI must pass (markdown lint + link check)
9. Approver merges (squash merge preferred)

## Authority Levels

| Document | Approval Required |
|----------|-------------------|
| `core/PILLARS.md` | Provost (Nathan) |
| `core/SOUL.md` | Provost |
| `core/DESIGN_RULES.md` | Provost |
| `core/<other>` | Provost or Lead Dev |
| `roots/<frozen versions>` | Frozen — cannot edit; only add new versions |
| `schools/<x>/README.md` | Owner of that School |
| `subprojects/<x>/*` | Sub-project owner + Lead Dev |
| `meta/*` | Lead Dev |
| `schemas/*` | Lead Dev |

## What Not To Do

- Do not commit secrets (API keys, tokens, passwords)
- Do not push to main directly
- Do not delete files (archive instead)
- Do not edit CHANGELOG entries after they are merged (append `[FIX]` instead)
- Do not break dryad-docs:// semantic links without updating references

## Questions

Open a GitHub issue tagged `question`. Or escalate to Nathan/Criss directly per the operating loop.

---

**End of CONTRIBUTING.md**
