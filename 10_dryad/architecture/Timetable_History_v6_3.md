# Dryad Build Timeline & History — v6.3

> **Node:** madhatter  
> **Last updated:** 2026-04-13  
> Tracks major build events, version bumps, and infrastructure milestones.

---

## Version History

| Version | Date | Summary |
|---------|------|---------|
| v1.0 | — | Initial Ollama + Open WebUI setup |
| v2.0 | — | LiteLLM gateway added; multi-model routing |
| v3.0 | — | Redis + Qdrant added; memory/vector infrastructure |
| v4.0 | — | Ralph Wiggum autonomous agent introduced |
| v5.0 | — | OpenWork approval layer; OpenCode integration |
| v5.x | — | Dryad Portal (Bun+Hono+HTMX) initial build |
| v6.0 | — | opencode-spun loop manager; Mem0 integration |
| v6.1 | — | Portal routes: loops, chat, creator, status, pages |
| v6.2 | — | Portal status.ts Docker API fix (dockerode + host.docker.internal) |
| v6.3 | 2026-03 | CHG-020 through CHG-025; systemd log sync design |
| v6.4 | TBD | dryad-systems repo + automated changelog (CHG-026+) |

---

## Milestone Log

### Phase 1 — Local Inference

- Installed Ollama on madhatter
- Configured NVIDIA drivers for RTX 3060 (12GB VRAM)
- First local model run: Llama 2
- Added Open WebUI as chat frontend

### Phase 2 — Gateway & Storage

- Deployed LiteLLM on port 4000 as unified inference gateway
- Established R1 rule: all app code routes through LiteLLM only
- Added Redis (`dryad-redis`) for shared state
- Added Qdrant for vector search
- Configured Caddy as edge proxy with TLS

### Phase 3 — Autonomous Agents

- Built Ralph Wiggum: reads `GOAL.md`, iterates to completion
- Defined workspace structure: `~/ralph/[project]/`
- Tuned `RALPH_MAX_ITERATIONS` and `RALPH_STEP_TIMEOUT`

### Phase 4 — OpenWork + Safety

- Built OpenWork approval layer (port 3456)
- Integrated OpenCode sidecar (port 3457) to intercept AI-executed code
- Implemented sandbox mode: untrusted code runs in isolated Docker containers
- Established `RALPH_USE_OPENWORK` toggle

### Phase 5 — Dryad Portal

- Built dryad-portal: Bun + Hono SSR + HTMX (no JSX)
- Routes: loops, chat, creator (app wizard), status (container health), pages (dashboard)
- Portal reads Docker API via `/var/run/docker.sock` (ro mount)
- `creator.ts`: 414-line app creation wizard with UUID in URL, Redis state persistence
- Fixed status.ts: migrated from direct docker socket to dockerode + `host.docker.internal`

### Phase 6 — Memory & MCP

- Added Mem0 Search (port 8020) and Mem0 API (port 8030)
- Deployed MCP Station (port 8586) as Model Context Protocol hub
- Added opencode-spun (port 7331) for managed coding loop sessions
- Integrated Dryad Tutorials (port 8888)

### Phase 6.3 — Infrastructure Hardening

- CHG-020: [archived — see v6.3 docs]
- CHG-021: [archived — see v6.3 docs]
- CHG-022: [archived — see v6.3 docs]
- CHG-023: [archived — see v6.3 docs]
- CHG-024: [archived — see v6.3 docs]
- CHG-025: [archived — see v6.3 docs]

### Phase 6.4 (current) — dryad-systems Repo

- CHG-026: dryad-systems GitHub repo created with full structure
  - Automated log sync via systemd timer (hourly)
  - Interactive CHG entry tool (`update-changelog.sh`)
  - GitHub Actions changelog validation
  - Full docs suite (System Reference, Runbook, Timeline, Phase 7 plan)

---

## Design Rule Evolution

| Rule | Added | Trigger |
|------|-------|---------|
| R1: LiteLLM only | Phase 2 | Prevent direct Ollama calls bypassing auth |
| R2: `/mnt/ai_engine/` | Phase 3 | Centralize model data, easier backup |
| R3: `mem_limit` required | Phase 4 | OOM killed several containers on 16GB node |
| R4: One Redis | Phase 5 | Prevented split-brain from multiple cache instances |
| R5: Caddy sole proxy | Phase 2 | Port conflicts when other services tried to bind 443 |

---

## Notable Incidents

### Ollama OOM During Ralph Loop

- **When:** Phase 4
- **Cause:** Ralph triggered multiple model loads simultaneously; VRAM exhausted
- **Fix:** Added `RALPH_STEP_TIMEOUT`; implemented explicit `ollama stop` between tasks

### Portal status.ts Docker Failure

- **When:** v6.2
- **Cause:** Portal tried to read `/var/run/docker.sock` directly from Node.js; permissions + path issues inside container
- **Fix:** Migrated to dockerode library; changed socket path to `host.docker.internal` Docker API endpoint

### Redis Split-Brain

- **When:** Phase 5 early
- **Cause:** Two Redis instances running (one from dryad-apps, one leftover from test)
- **Fix:** R4 rule enforced; single `dryad-redis` container; others removed

---

## Phase 7 Preview

See [dryad-phase7-talos-plan.md](dryad-phase7-talos-plan.md) for the planned Kubernetes cluster bootstrap using Talos Linux.

Key goals:
- Multi-node cluster (madhatter + future nodes)
- GPU workload scheduling
- Persistent storage for AI data
- Automated rolling updates for Dryad services
