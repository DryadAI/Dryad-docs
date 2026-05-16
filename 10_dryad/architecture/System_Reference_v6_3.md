# Dryad System Reference — v6.3

> **Node:** madhatter  
> **Last updated:** 2026-04-13  
> **Next version:** v6.4 (see ACTIVE_CHANGELOG.md for pending changes)

---

## Hardware

| Component | Spec |
|-----------|------|
| CPU | Intel i7-9700K (8-core) |
| RAM | 16 GB |
| GPU | NVIDIA RTX 3060 12GB VRAM |
| OS | Arch Linux |
| Tailscale IP | 100.72.190.41 |
| Hostname | madhatter |

---

## Service Map

### Core Infrastructure

| Service | Port | Transport | Purpose |
|---------|------|-----------|---------|
| Ollama | 11434 | HTTP | Local LLM inference engine |
| LiteLLM Gateway | 4000 | HTTP (Bearer) | Multi-model inference gateway |
| Redis | 6379 | TCP | Shared key-value store, session state |
| Qdrant | 6333 | HTTP | Vector database for semantic search |
| Caddy | 80 / 443 | HTTP/S | Reverse proxy + TLS termination |

### User Interfaces

| Service | Port | URL | Purpose |
|---------|------|-----|---------|
| Dryad Portal | 3080 | madhatter:3080 | Main system dashboard |
| Open WebUI | 3010 | madhatter:3010 | Chat UI for local models |
| Portainer | 9443 | madhatter:9443 | Container management |
| Netdata | 19999 | madhatter:19999 | Real-time system metrics |

### Development Tools

| Service | Port | Purpose |
|---------|------|---------|
| OpenCode | 4096 | AI-assisted development (VS Code-based) |
| opencode-spun | 7331 | Managed AI coding loops (via Ralph/opencode-spun) |
| OpenWork | 3456 | Approval layer for AI-executed code |
| OpenCode Sidecar | 3457 | Intercepts OpenCode → routes through OpenWork |

### Memory & Knowledge

| Service | Port | Purpose |
|---------|------|---------|
| Mem0 Search | 8020 | Personal knowledge management (search) |
| Mem0 API | 8030 | Programmatic memory access |
| MCP Station | 8586 | Model Context Protocol hub |

### Learning

| Service | Port | Purpose |
|---------|------|---------|
| Dryad Tutorials | 8888 | Interactive notebook learning environment |

---

## Network Architecture

```
Internet / Tailscale
       │
     Caddy (443/80)
       │
   ┌───┴──────────────────────┐
   │                          │
dryad-portal:3080      open-webui:3010
   │                          │
   └─────────┬────────────────┘
             │
     LiteLLM:4000 (Bearer sk-dryad-master)
             │
      ┌──────┴──────┐
  Ollama:11434   External APIs
```

### OpenWork Flow

```
AI Agent (Ralph/OpenCode)
    │
OpenWork:3456 (approval layer)
    │ ← Human approves/denies
    │
Sandboxed Execution (Docker container)
```

---

## Docker Container Names

| Container | Image | Notes |
|-----------|-------|-------|
| `dryad-litellm` | litellm | Model gateway |
| `dryad-redis` | redis | Shared data store |
| `dryad-qdrant` | qdrant/qdrant | Vector DB |
| `open-webui` | open-webui | Chat interface |
| `mcp-search` | custom | Mem0 search |
| `mcp-mem0` | custom | Mem0 API |
| `mcp-station` | custom | MCP hub |
| `openwork` | custom | Approval layer |
| `dryad-portal` | custom | Main dashboard |
| `dryad-tutorials` | custom | Notebook server |

---

## Design Rules

| Rule | Constraint |
|------|-----------|
| R1 | LiteLLM `:4000` is the **only** inference gateway. No direct Ollama calls from app code. |
| R2 | AI model data lives on `/mnt/ai_engine/` only |
| R3 | 16GB RAM hard limit — all containers require `mem_limit` in compose |
| R4 | One Redis instance (`dryad-redis`) shared across all services |
| R5 | Caddy is the sole edge proxy. No other service binds 80/443 |

---

## Authentication

| Service | Auth method | Credential location |
|---------|------------|-------------------|
| LiteLLM | Bearer token | `sk-dryad-master` in `.env` |
| Redis | Password | `Bongo1102n@t3` in `.env` (never commit) |
| Portainer | Username/password | Local account |
| OpenWork | Token pair | `dryad-client` / `dryad-host` |
| OpenCode via OpenWork | Basic auth | `opencode` / `dryad-code-pass` |

---

## Volume Mounts

| Path | Purpose |
|------|---------|
| `/mnt/ai_engine/` | Ollama models, AI data |
| `/ralph` | Ralph agent workspaces |
| `/github` | Git repos (mapped from `~/GitHub`) |
| `/agents` | Agent configurations |
| `/pipeline` | Consulting pipeline markdown files |
| `/config/litellm.yaml` | LiteLLM model config |
| `/var/run/docker.sock` | Docker API access (ro, portal only) |

---

## Dryad Portal (v6.3)

**Stack:** Bun + Hono (SSR, no JSX) + HTMX  
**Source:** `~/GitHub/dryad-portal/`  
**Container:** `dryad-portal` on port 3080

### Key Paths

| Path | Purpose |
|------|---------|
| `src/routes/` | Route handlers: loops.ts, chat.ts, creator.ts, status.ts, pages.ts |
| `src/config.ts` | Env vars + LLM system prompts |
| `src/layout.ts` | HTML shell, sidebar nav, CSS custom properties |

### Routes

| Route | Handler | Purpose |
|-------|---------|---------|
| `/` | pages.ts | Dashboard |
| `/loops` | loops.ts | opencode-spun loop management |
| `/chat` | chat.ts | Chat interface |
| `/create` | creator.ts | App creation wizard |
| `/status` | status.ts | Container health (via Docker API) |
| `/knowledge` | — | Knowledge base access |

### Portal Environment

```
LITELLM_URL=http://host.docker.internal:4000
LITELLM_KEY=sk-dryad-master
REDIS_URL=redis://:Bongo1102n@t3@dryad-redis:6379
OLLAMA_URL=http://host.docker.internal:11434
OPENCODE_URL=http://host.docker.internal:4096
SPUN_URL=http://host.docker.internal:7331
```

---

## Ralph Wiggum (Autonomous Agent)

**Purpose:** Autonomous coding agent that reads `GOAL.md` and iterates toward completion.

### Configuration

| Env var | Default | Purpose |
|---------|---------|---------|
| `RALPH_MODEL` | qwen-coder-fast | Inference model |
| `RALPH_MAX_ITERATIONS` | 500 | Max coding iterations |
| `RALPH_STEP_TIMEOUT` | 120 | Per-step timeout (seconds) |
| `RALPH_USE_OPENWORK` | false | Route through OpenWork |
| `RALPH_OPENWORK_USER` | opencode | OpenWork username |
| `RALPH_OPENWORK_PASS` | dryad-code-pass | OpenWork password |

### Workspace

```
~/ralph/[project]/
├── GOAL.md         # Task definition (required)
├── ralph.log       # Verbose execution log
└── LAST_ERROR      # Last error (check if stuck)
```

---

## Systemd Services (madhatter)

| Unit | Type | Purpose |
|------|------|---------|
| `ollama.service` | Service | Ollama inference daemon |
| `opencode.service` | Service | OpenCode IDE daemon |
| `opencode-spun.service` | Service | opencode-spun loop manager |
| `dryad-log-sync.service` | Service (oneshot) | Log collection |
| `dryad-log-sync.timer` | Timer | Fires log sync every 60min |

---

## Tailscale

- Hostname: `madhatter`
- IP: `100.72.190.41`
- Used for: remote access, inter-device routing
- All Caddy-proxied services accessible via Tailscale IP

---

## Changelog Tracking

Changes to this document are staged in `ACTIVE_CHANGELOG.md` as CHG-NNN entries before being merged into new doc versions. See root `ACTIVE_CHANGELOG.md` for pending v6.4 changes.
