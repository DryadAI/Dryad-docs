# Dryad Operations Runbook — v6.3

> **Node:** madhatter  
> **Last updated:** 2026-04-13  
> Reference: [System_Reference_v6_3.md](System_Reference_v6_3.md)

---

## Daily Health Check

```bash
# Container status
docker ps --format 'table {{.Names}}\t{{.Status}}\t{{.Ports}}'

# System resources
docker stats --no-stream --format 'table {{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}'

# GPU status
nvidia-smi

# Ollama loaded models
ollama ps

# Systemd timers
sudo systemctl list-timers --all | grep dryad

# Portal health
curl -s http://localhost:3080/health
```

---

## Service Management

### Start / Stop All Services

```bash
# Start all Docker services
cd ~/dryad-apps && docker compose up -d

# Stop all Docker services
cd ~/dryad-apps && docker compose down

# Restart all services
cd ~/dryad-apps && docker compose down && docker compose up -d
```

### Restart Individual Container

```bash
docker restart dryad-portal
docker restart dryad-litellm
docker restart open-webui
docker restart dryad-qdrant
docker restart dryad-redis
```

### View Container Logs

```bash
# Follow live
docker logs -f dryad-portal
docker logs -f dryad-litellm

# Last N lines
docker logs --tail 100 dryad-portal
docker logs --tail 50 open-webui
```

### Systemd Services

```bash
# Ollama
sudo systemctl status ollama
sudo systemctl restart ollama

# OpenCode
sudo systemctl status opencode
sudo systemctl restart opencode

# opencode-spun
sudo systemctl status opencode-spun
sudo systemctl restart opencode-spun

# Log sync timer
sudo systemctl status dryad-log-sync.timer
sudo systemctl start dryad-log-sync.service    # manual trigger
```

---

## Inference

### Check Available Models

```bash
# Models loaded in VRAM
ollama ps

# All downloaded models
ollama list

# LiteLLM gateway model list
curl -s http://localhost:4000/models -H "Authorization: Bearer sk-dryad-master" | jq '.data[].id'
```

### Load / Unload Model

```bash
# Load model (keeps in VRAM)
ollama run qwen2.5-coder:7b

# Unload specific model
ollama stop qwen2.5-coder:7b

# Unload all models
ollama ps | awk 'NR>1 {print $1}' | xargs -I{} ollama stop {}
```

### Test LiteLLM Gateway

```bash
curl -s http://localhost:4000/chat/completions \
  -H "Authorization: Bearer sk-dryad-master" \
  -H "Content-Type: application/json" \
  -d '{"model": "qwen-coder-fast", "messages": [{"role": "user", "content": "ping"}]}' \
  | jq '.choices[0].message.content'
```

---

## Dryad Portal

### Build and Deploy

```bash
cd ~/GitHub/dryad-portal

# Build
bun run build

# Lint check
bun run lint

# Rebuild and restart container
docker compose build dryad-portal && docker restart dryad-portal

# Or via compose
docker compose up -d --build dryad-portal
```

### Check Portal Health

```bash
curl -s http://localhost:3080/health
curl -s http://localhost:3080/status   # container status page
```

---

## opencode-spun (Loop Manager)

### View Active Loops

```bash
curl -s http://localhost:7331/api/loops | jq
```

### Trigger Loop via Portal

Visit: `http://madhatter:3080/loops`

### Direct API

```bash
# Start loop
curl -s -X POST http://localhost:7331/api/loops \
  -H "Content-Type: application/json" \
  -d '{"project": "my-project", "goal": "..."}'

# Stop loop
curl -s -X DELETE http://localhost:7331/api/loops/{id}
```

---

## OpenWork (Approval Layer)

### Enable for Ralph

```bash
export RALPH_USE_OPENWORK=true
```

### Sandbox Mode

```bash
# Start sandbox (isolates AI code execution in Docker)
~/bin/openwork-sandbox-start

# Stop sandbox
~/bin/openwork-sandbox-stop
```

### Check OpenWork Health

```bash
curl http://localhost:3456/health
```

### Web UI

- Admin: https://openwork.madhatter.modlin.cloud
- OpenCode: https://openwork-code.madhatter.modlin.cloud

---

## Log Sync

### Manual Trigger

```bash
bash /home/katalyst/dryad-systems/scripts/sync-logs.sh
```

### View Recent Auto-Commits

```bash
cd /home/katalyst/dryad-systems
git log --oneline | grep '\[automated\]' | head -20
```

### View Timer Schedule

```bash
sudo systemctl list-timers dryad-log-sync.timer
```

### View Service Journal

```bash
journalctl -u dryad-log-sync.service -n 50
```

---

## Changelog Updates

### Append CHG Entry

```bash
cd /home/katalyst/dryad-systems
bash scripts/update-changelog.sh
```

### Commit and Push

```bash
git add ACTIVE_CHANGELOG.md
git commit -m 'CHG-NNN: [title]'
git push origin main
```

---

## Maintenance

### Docker Cleanup

```bash
# Remove stopped containers, unused images, networks
docker system prune -a

# Remove unused volumes (caution: data loss)
docker volume prune
```

### Update Services

```bash
cd ~/dryad-apps
git pull
docker compose pull
docker compose up -d
```

### Backup Config

```bash
tar czf ~/backups/dryad-config-$(date +%Y%m%d).tar.gz ~/dryad-apps
```

### GPU Memory Check

```bash
# VRAM usage
nvidia-smi --query-gpu=memory.used,memory.free,memory.total --format=csv

# Processes using GPU
nvidia-smi --query-compute-apps=pid,name,used_memory --format=csv
```

---

## Troubleshooting

### Ralph Loop Stuck

```bash
# Check last error
cat ~/ralph/[project]/LAST_ERROR

# View verbose log
tail -50 ~/ralph/[project]/ralph.log

# Reduce iterations for testing
export RALPH_MAX_ITERATIONS=10
```

### Portal Not Responding

```bash
# Check container
docker ps | grep dryad-portal
docker logs --tail 50 dryad-portal

# Check port
curl -v http://localhost:3080/health

# Restart
docker restart dryad-portal
```

### LiteLLM Errors

```bash
# View gateway logs
docker logs --tail 100 dryad-litellm

# Verify config
cat /config/litellm.yaml

# Test directly
curl http://localhost:4000/health
```

### OpenCode Connection Issues

```bash
# Check service
sudo systemctl status opencode

# Verify port
ss -tlnp | grep 4096

# Test
curl http://localhost:4096/
```

### Ollama OOM / Model Unloaded

```bash
# Check VRAM
nvidia-smi

# Check which models are loaded
ollama ps

# Manually unload to free VRAM
ollama stop [model-name]

# Check Ollama logs
journalctl -u ollama -n 100
```

### Redis Connection Failures

```bash
# Check container
docker ps | grep dryad-redis

# Test connection
docker exec -it dryad-redis redis-cli -a Bongo1102n@t3 ping

# View logs
docker logs --tail 50 dryad-redis
```

---

## Tailscale

```bash
# Status
tailscale status

# Ping madhatter from another device
tailscale ping madhatter

# IP
tailscale ip -4
```
