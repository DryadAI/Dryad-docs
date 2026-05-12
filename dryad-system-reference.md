# Dryad System Reference Manual

## Overview

This document describes the architecture and configuration of the Dryad system, a comprehensive AI development and deployment platform.

## System Components

### Core Services
- **Ollama** (port 11434) - Local LLM inference engine
- **Bifrost Gateway** (port 8080) - Multi-model inference gateway (migrated from LiteLLM :4000)
- **Redis** (port 6379) - High-performance key-value store
- **Qdrant** (port 6333) - Vector database for semantic search
- **Caddy** - Reverse proxy and TLS termination

### User Interfaces
- **Open WebUI** (port 3010) - Chat interface for AI models
- **Portainer** (port 9443) - Container management dashboard
- **Netdata** (port 19999) - Real-time system monitoring
- **Dryad Portal** (port 3080) - Main system dashboard
- **ACE-Step** (port 7860) - Music generation UI; `https://acestep.madhatter`
- **Vocal** (port 6969) - Vocal separation/processing; `https://vocal.madhatter`

### Development Tools
- **Ralph Wiggum** - Autonomous coding agent
- **OpenCode** (port 4096) - AI-assisted development environment
- **Spun Loops** (port 7331) - Managed AI coding sessions

### Memory and Knowledge
- **Mem0 Search** (port 8020) - Personal knowledge management
- **Mem0 API** (port 8030) - Programmatic memory access
- **MCP Station** (port 8586) - Model Context Protocol hub

### Tutorial and Learning
- **Dryad Tutorials** (port 8888) - Interactive learning environment

## Cluster Infrastructure (Phase 7)

### Network

| Node | IP | Role | Status |
|------|----|------|--------|
| madhatter | 10.10.10.1 | Compute host / cluster gateway | Active |
| dryad-sw1 | 10.10.10.2 | FortiSwitch-148F-FPOE (VLAN 10) | Active |
| dryad-forge | 10.10.10.20 | PXE bootstrap server (dnsmasq + matchbox) | Pending install |
| dryad-gate | 10.10.10.21 | Public edge — Caddy + cloudflared | Active |

- Cluster VLAN: **10** (`10.10.10.0/24`)
- Madhatter cluster NIC: **enp2s0f1** (MAC `a0:46:9f:a6:af:95`) — NOT enp2s0f0
- networkd config: `/etc/systemd/network/10-cluster.network` → `Name=enp2s0f1`
- FortiSwitch password: `dryad2026` | console: USB serial at 115200 baud

### dryad-gate

- **IP:** `10.10.10.21/24` on `eno1` (static via systemd-networkd)
- **Services:** `caddy.service`, `cloudflared.service`, `dryad-deploy.timer`
- **Function:** TLS termination + Cloudflare Zero Trust tunnel for `dryadai.net`
- **Switch port:** FortiSwitch port4, VLAN 10 native
- **Repo:** `~/GitHub/dryad-gate/`

### dryad-forge (planned)

- **IP:** `10.10.10.20/24` on `eno1` (static)
- **Services:** `dnsmasq` (DNS + DHCP + TFTP for cluster), `matchbox` (PXE profiles), `tailscaled`
- **Function:** Bootstrap server for K8s node PXE installs
- **Switch port:** FortiSwitch (port TBD), VLAN 10 native
- **Install:** `~/dryad-arch-iso/` on madhatter — rebuild ISO with `mkarchiso` before flashing

## OpenWork Integration

### Overview
OpenWork is an approval layer that sits between AI coding agents and system execution environments. It provides oversight and control over AI-generated code execution.

### Architecture
```
AI Agent → OpenWork Approval Layer → System Execution
              ↑              ↓
         Human Approver    Sandboxed Execution
```

### Components
- **OpenWork Server** (port 3456) - Approval workflow engine
- **OpenCode Sidecar** (port 3457) - Intercept and route OpenCode requests through approval layer

### Configuration Parameters
- Client Token: `dryad-client`
- Host Token: `dryad-host`
- OpenCode Credentials: `opencode` / `dryad-code-pass`

### URLs
- OpenWork Admin: https://openwork.madhatter.modlin.cloud
- OpenCode Interface: https://openwork-code.madhatter.modlin.cloud

### Ralph Integration
Ralph can be configured to route through OpenWork by setting:
```bash
RALPH_USE_OPENWORK=true
```

### Sandbox Mode
OpenWork can execute untrusted code in isolated Docker containers:
```bash
# Start sandbox mode
~/bin/openwork-sandbox-start

# Stop sandbox mode
~/bin/openwork-sandbox-stop
```

### Health Checks
```bash
# Check overall system status
~/bin/dryad-status

# Check OpenWork health specifically
curl http://localhost:3456/health
```

## Environment Variables

### Ralph Configuration
- `RALPH_MODEL` - AI model to use (default: qwen-coder-fast)
- `RALPH_MAX_ITERATIONS` - Maximum coding iterations (default: 500)
- `RALPH_STEP_TIMEOUT` - Timeout per step in seconds (default: 120)
- `RALPH_USE_OPENWORK` - Route through OpenWork approval layer (default: false)
- `RALPH_OPENWORK_USER` - OpenWork username (default: opencode)
- `RALPH_OPENWORK_PASS` - OpenWork password (default: dryad-code-pass)

## Security Considerations

All Dryad services are protected by strong authentication:
- Redis requires password authentication
- Bifrost gateway uses bearer token authentication
- Portainer requires username/password login
- All web services are served over HTTPS via Caddy proxy
- OpenWork provides an additional approval layer for AI-executed code
- Sandbox mode isolates untrusted code execution

## Troubleshooting

### Service Restart Commands
```bash
# Restart all Dryad services
cd ~/dryad-apps && docker-compose down && docker-compose up -d

# Restart individual services
docker-compose restart [service-name]

# View service logs
docker-compose logs [service-name]
```

### Common Issues

#### Ralph Loop Stuck
1. Check `~/ralph/[project]/LAST_ERROR` for error details
2. Check `~/ralph/[project]/ralph.log` for verbose logging
3. Ensure GOAL.md exists in project directory
4. Consider reducing RALPH_MAX_ITERATIONS for testing

#### OpenCode Connection Issues
1. Verify OpenCode is running on port 4096
2. Check authentication credentials
3. Ensure no firewall blocking connections
4. Validate model availability through LiteLLM gateway

### Maintenance Commands
```bash
# System cleanup
docker system prune -a

# Update all services
cd ~/dryad-apps && git pull && docker-compose pull && docker-compose up -d

# Backup configuration
tar czf ~/backups/dryad-config-$(date +%Y%m%d).tar.gz ~/dryad-apps
```