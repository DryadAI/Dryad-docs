# dryad-systems — ACTIVE CHANGELOG

> **Rules:**
> - Only **append** entries. Never modify or delete existing entries.
> - Use sequential CHG-NNN numbering. Check last entry before adding new one.
> - Required fields: **Node**, **What changed**, **Affects**, **Status**
> - Timestamp format: `YYYY-MM-DD`
> - Status values: `DONE` | `READY FOR DEPLOYMENT` | `PENDING` | `BLOCKED`
> - **Never touch archived session blocks.**
> - New entries go above the `## Archived Sessions` section.

---

## Active Session (v6.3 → v6.4)

---

#### CHG-037 · madhatter cluster VLAN connectivity made persistent — 2026-05-15
- **Node:** madhatter
- **What changed:**
  - Confirmed `enp2s0f1` (cluster NIC, 10.10.10.1/24) is already persistent via a NetworkManager connection profile with `autoconnect=yes`. Earlier in the session the interface had no IP at runtime; assigning it manually restored cluster reachability. Persistence is via NM, not systemd-networkd as previously assumed.
  - Made the cluster VLAN MASQUERADE rule persistent:
    - `sudo iptables -t nat -A POSTROUTING -s 10.10.10.0/24 -o enp2s0f0 -j MASQUERADE` confirmed at runtime
    - `sudo iptables-save | sudo tee /etc/iptables/iptables.rules`
    - `sudo systemctl enable iptables.service` (Arch's iptables-restore unit)
  - NAT rule now restored at boot before Docker initializes. Docker re-adds its own MASQUERADE entries for `docker0` and the per-stack bridges after that, no conflict observed.
  - Gate (`10.10.10.21`) had no `hostname` binary; installed `inetutils` package (provides `/usr/bin/hostname`).
- **Affects:**
  - forge (10.10.10.20) and gate (10.10.10.21) keep internet + cluster connectivity through madhatter across reboot
  - Phase 7 cluster bootstrap can rely on madhatter's cluster gateway being stable
  - Smoke test pending: confirm no Docker / iptables ordering issue on next planned madhatter reboot
- **Status:** DONE. Reboot smoke test PENDING (opportunistic — next time madhatter restarts for any reason).

---

#### CHG-036 · dryad-forge brought up via stock ISO + scripted SSH install — 2026-05-14
- **Node:** madhatter (build host); dryad-forge (target, 10.10.10.20, MAC `7c:d3:0a:78:c4:a8`)
- **What changed:**
  - Abandoned the in-progress custom offline ISO build (`/home/katalyst/dryad-arch-iso/`) as the bring-up vector for forge after 6 failed iterations.
  - Booted forge from the official upstream Arch ISO (`/home/katalyst/iso/stock-archlinux.iso`, 1.5 G), set root password, started sshd, captured DHCP address (10.0.4.101).
  - Pushed `install.sh` and post-install scripts from madhatter to forge over SSH (`rsync -rz`, payloads staged in `/tmp` to dodge the 256 MB airootfs COW limit).
  - Ran an **online** pacstrap (240-package closure) instead of the planned offline `pacstrap -U`. Configured hostname, fstab, locale, users (`root` + `katalyst`, both `dryad2026`, `wheel` NOPASSWD), `systemd-networkd` 10-cluster.network for `eno1` → 10.10.10.20/24, masked NetworkManager, enabled networkd/resolved/sshd/docker/tailscaled, installed GRUB EFI, wrote `/etc/dryad-installed` marker, deposited `post-forge.sh` and `post-gate.sh` in `/root`.
  - System rebooted into Arch on `/dev/nvme0n1` at static 10.10.10.20.
  - Subsequent `post-forge.sh` run installed matchbox v0.11.0 (listening on :8080), authenticated Tailscale (`dryad-forge` / `100.68.243.9`). dnsmasq install was silently skipped by the script (logs "Configuring dnsmasq..." but does nothing) — installed and configured separately: `pacman -S dnsmasq` + `/etc/dnsmasq.d/cluster-pxe.conf` (DHCP 10.10.10.50-200, TFTP root, PXE boot chain), service enabled.
- **Affects:**
  - dryad-forge node existence (now reachable from any host on 10.10.10.0/24)
  - Bring-up procedure of record — the custom offline ISO is **no longer** the documented path; see new runbook `docs/runbooks/new-node-bringup.md`
  - `post-forge.sh` source script bug — needs patch so next node doesn't hit the same dnsmasq silent-skip
  - dryad-gate (10.10.10.21) already running Caddy + cloudflared + dryad-deploy.timer; verified healthy this session
- **Status:** DONE. Forge is production-ready as the PXE bootstrap node for Phase 7. Open follow-up: patch `post-forge.sh` to actually install + configure dnsmasq instead of just logging it.

---

#### CHG-035 · Persistent NAT for cluster VLAN + dryad-forge post-install gap audit — 2026-05-14
- **Node:** madhatter (NAT host); dryad-forge (10.10.10.20)
- **What changed:**
  - Added `/etc/systemd/system/dryad-cluster-nat.service` (oneshot, `RemainAfterExit=yes`) that idempotently installs `iptables -t nat -A POSTROUTING -s 10.10.10.0/24 -o enp2s0f0 -j MASQUERADE` after `network-online.target` and `docker.service`. Enabled + started. Survives reboot without conflicting with docker's runtime chain management (we don't `iptables-restore` a full dump, just append our one rule).
  - Verified `net.ipv4.ip_forward = 1` was already persisted via `/etc/sysctl.d/99-tailscale.conf` and `/etc/sysctl.d/99-forwarding.conf` (two separate sources, both set it to 1). No new sysctl drop-in needed.
  - Closed out the dryad-forge bring-up by re-running `post-forge.sh` after NAT was in place: matchbox v0.11.0 binary installed at `/usr/local/bin/matchbox`, `matchbox.service` enabled + listening on `:8080`, tailscale authorized (`dryad-forge` joined at `100.68.243.9`).
  - Audited the three remaining "gaps" against the Phase 7 (Talos) plan:
    - `dnsmasq` on forge — post-forge.sh has a stub that only echoes "Configuring dnsmasq" and never writes config or starts the service. Forge's intended PXE-server role (matchbox + dnsmasq) is **obsolete under Phase 7's Talos pivot** (Talos boots per-node from ISO, not via PXE). Stub kept dormant; no work scheduled.
    - `tailscale --accept-routes` on forge — no concrete consumer; Phase 7 mentions advertising TS routes, not accepting peer routes. Deferred indefinitely (YAGNI).
- **Affects:**
  - Internet path for any future host on 10.10.10.0/24 now survives a madhatter reboot.
  - Phase 7 PXE plan (now superseded): if Talos is the decided path, matchbox/dnsmasq lines in `post-forge.sh` should be removed in a future change to avoid confusing future operators. Not done in this CHG.
- **Status:** DONE.

---

#### CHG-034 · dryad-forge brought up via stock ISO + scripted SSH install — 2026-05-14
- **Node:** madhatter (build host); dryad-forge (target, 10.10.10.20, MAC `7c:d3:0a:78:c4:a8`)
- **What changed:**
  - Abandoned the in-progress custom offline ISO build (`/home/katalyst/dryad-arch-iso/`) as the bring-up vector for forge after 6 failed iterations.
  - Booted forge from the official upstream Arch ISO (`/home/katalyst/iso/stock-archlinux.iso`, 1.5 G), set root password, started sshd, captured DHCP address (10.0.4.101).
  - Pushed `install.sh` and post-install scripts from madhatter to forge over SSH (`rsync -rz`, payloads staged in `/tmp` to dodge the 256 MB airootfs COW limit).
  - Ran an **online** pacstrap (240-package closure) instead of the planned offline `pacstrap -U`. Configured hostname, fstab, locale, users (`root` + `katalyst`, both `dryad2026`, `wheel` NOPASSWD), `systemd-networkd` 10-cluster.network for `eno1` → 10.10.10.20/24, masked NetworkManager, enabled networkd/resolved/sshd/docker/tailscaled, installed GRUB EFI, wrote `/etc/dryad-installed` marker, deposited `post-forge.sh` and `post-gate.sh` in `/root`.
  - System rebooted into Arch on `/dev/nvme0n1` at static 10.10.10.20.
- **Affects:**
  - dryad-forge node existence (now reachable from any host on 10.10.10.0/24)
  - Bring-up procedure of record — the custom offline ISO is **no longer** the documented path; see new runbook `docs/runbooks/new-node-bringup.md`
  - Open: `post-forge.sh` has not yet been executed; cluster role (dnsmasq + matchbox) is not yet active
  - Open: dryad-gate (MAC `9c:7b:ef:77:89:79`, 10.10.10.21) will follow the same procedure
- **Status:** DONE (base install + reboot confirmed). Service bring-up via `post-forge.sh` is PENDING and requires console access or a host on 10.10.10.0/24.

---

#### CHG-029 · Fix HTTPS outage: stale ACME challenge records + Cloudflare anycast negative-cache — 2026-04-28

- **Node:** madhatter
- **What changed:**
  - Root cause: 15 stale `_acme-challenge.*` TXT records left in Cloudflare DNS from a killed Caddy session. New ACME orders generated new tokens but old tokens remained in DNS; propagation check always failed (last error: `<nil>`).
  - Secondary cause: Cloudflare anycast auth NSes have 900-second negative-cache TTL. After bulk deletion+recreation cycles, resolvers cached NODATA for all challenge names for up to 15 minutes, preventing CertMagic from verifying any newly-placed tokens.
  - Fix: deleted all stale TXT records via Cloudflare API, wiped staging ACME state (`/data/caddy/acme/acme-staging-v02.*`, `/data/caddy/certificates/acme-staging-v02.*`), and updated Caddyfile `cloudflare_tls` snippet with `propagation_delay 60s` + `propagation_timeout 20m` to outlast the 900-second negative-cache window.
  - Caddyfile bumped Rev 6.3 → Rev 6.4.
- **Affects:** All 19 `*.madhatter.modlin.cloud` public HTTPS domains
- **Status:** DONE

---

#### CHG-026 · dryad-systems GitHub repo structure created — 2026-04-13

- **Node:** madhatter (via Claude)
- **What changed:** Created `DryadAI/dryad-systems` repository with full directory structure including README, ACTIVE_CHANGELOG, .gitignore, GitHub Actions workflows, automation scripts (update-changelog.sh, sync-logs.sh, setup-systemd-timer.sh), and log directory scaffold. Systemd service and timer files included for automated hourly log collection.
- **Commands/scripts:**
  ```bash
  mkdir -p /home/katalyst/dryad-systems/{.github/workflows,docs,logs/madhatter,scripts}
  # Files created: README.md, ACTIVE_CHANGELOG.md, .gitignore
  # scripts/: update-changelog.sh, sync-logs.sh, setup-systemd-timer.sh
  # .github/workflows/: validate-changelog.yml
  ```
- **Affects:** System Reference | Operations Runbook | Timetable History
- **Status:** READY FOR DEPLOYMENT

---

#### CHG-027 · Migrate all clients from LiteLLM :4000 to Bifrost :8080 — 2026-04-15

- **Node:** madhatter (via Claude)
- **What changed:** Replaced LiteLLM gateway with Bifrost across all clients. Updated `STN_AI_BASE_URL` in dryad-mcp, `LITELLM_URL` env vars in both dryad-portal copies, port/service labels in status routes, inline comments in config.ts, pages.ts display strings, and opencode-spun opencode.json baseURL. Configured open-webui server-side OpenAI connection via direct DB write (`webui.db` config table) since UI form required container-resolvable hostname (`bifrost:8080`) not accessible from browser.
- **Affects:** dryad-mcp (mcp-station), dryad-portal (live + GitHub), GitHub/opencode-spun, open-webui
- **Repos committed:** `GitHub/dryad-portal` (f893178), `GitHub/opencode-spun` (5f8b18d)
- **Status:** DONE

---

#### CHG-028 · Add ACE-Step + Vocal to Caddy, portal, and docs — 2026-04-19

- **Node:** madhatter (via Claude)
- **What changed:** Added `acestep.madhatter` Caddy reverse proxy entry (host port 7860) and `/acestep*` IP-fallback handle. Added ACE-Step (🎵) and Vocal (🎤) shortcuts to dryad-portal sidebar and mobile bottom-nav. Added ports 6969 (vocal) and 7860 (acestep) to portal status port monitor. Updated `dryad-system-reference.md`: ACE-Step + Vocal added to User Interfaces section, LiteLLM → Bifrost reference corrected.
- **Affects:** dryad-core/Caddyfile, dryad-portal/src/layout.ts, dryad-portal/src/routes/status.ts, GitHub/dryad-docs/dryad-system-reference.md
- **Status:** DONE

#### CHG-030 · dryad-gate VLAN diagnosis: port 4 not on VLAN 10, physical console required — 2026-05-12

- **Node:** madhatter / FortiSwitch (dryad-sw1)
- **What changed:**
  - Diagnosed root cause of gate connectivity failure after port-4 move.
  - `ip neigh` shows `10.10.10.2 FAILED` and `10.10.10.21 FAILED` — ARP never resolves. Gate MAC (`9c:7b:ef:77:89:79`) last seen at `10.0.4.102` (STALE), indicating it received DHCP from upstream router on wrong VLAN.
  - Madhatter's own port on the FortiSwitch is also not on VLAN 10 — FortiSwitch mgmt at `10.10.10.2` unreachable via SSH from all paths (no route).
  - Gate's `systemd-networkd` static config (`10.10.10.21/24` on eno1) is correct but irrelevant until switch VLAN is fixed.
  - **Fix required (physical):** Console into FortiSwitch (RJ45), set port4 and madhatter's port to `native-vlan=10 allowed-vlans=10`. See `~/GitHub/dryad-gate/install-config/fortiswitch-port4.md` for exact CLI.
  - Snapshotted gate config to `~/GitHub/dryad-gate/` (scripts, network config, switch runbook).
- **Affects:** dryad-gate network reachability, cluster VLAN 10
- **Status:** BLOCKED — requires physical FortiSwitch console access

---

#### CHG-031 · Fix dryad-arch-iso: add keyring init + reflector before pacstrap — 2026-05-12

- **Node:** madhatter (ISO build host)
- **What changed:**
  - Root cause of archiso failures identified: GPG keyring not initialized before `pacstrap`, causing "GPG no data" errors; mirrorlist contained dead/slow mirrors causing SSL cert errors and malformed responses.
  - Fix: added 3 lines to `airootfs/root/install.sh` before pacstrap call:
    ```bash
    pacman-key --init
    pacman-key --populate archlinux
    reflector --latest 10 --sort rate --protocol https --save /etc/pacman.d/mirrorlist
    ```
  - `reflector` is already in `packages.x86_64` (live ISO environment). This generates a clean HTTPS-only mirrorlist ranked by speed, resolving all three failure modes.
  - Decision: **retry archiso** (not Talos/Flatcar) — forge role is PXE bootstrap server (dnsmasq + matchbox + Tailscale), not a K8s node. Talos is the wrong tool. Existing `post-forge.sh` is ready; only the ISO build step was broken.
  - Synced fixed `install.sh` to `~/GitHub/dryad-gate/scripts/` snapshot.
- **Affects:** dryad-forge install (nvme0n1, MAC 7c:d3:0a:78:c4:a8), future dryad-gate rebuilds
- **Status:** READY FOR DEPLOYMENT — rebuild ISO and retry forge install

---

#### CHG-032 · dryad-gate VLAN connectivity restored — 2026-05-12

- **Node:** madhatter / FortiSwitch (dryad-sw1) / dryad-gate
- **What changed:**
  - Factory reset FortiSwitch (admin password lost, reset button held 15s). New password: `dryad2026`.
  - Configured FortiSwitch VLAN 10 on port4 (gate) and port1 (madhatter) via pyserial automation over USB console at 115200 baud.
  - Discovered madhatter's FortiSwitch-facing NIC is `enp2s0f1` (MAC `a0:46:9f:a6:af:95`), NOT `enp2s0f0`. Cluster static IP (`10.10.10.1/24`) was on wrong interface — ARP never resolved.
  - Fixed `/etc/systemd/network/10-cluster.network`: changed `Name=enp2s0f0` → `Name=enp2s0f1`.
  - Fixed `/etc/systemd/network/20-dryad-fabric.network`: changed to `DHCP=yes` on `enp2s0f0` (internet-facing).
  - Gate (`dryad-gate`) had `systemd-networkd` disabled — re-enabled and started manually via monitor. Static IP `10.10.10.21/24` applied correctly on `eno1`.
  - Result: madhatter ↔ gate ping 0.5ms, SSH working, caddy active, cloudflared reconnecting.
- **Affects:** dryad-gate reachability, madhatter cluster networking, FortiSwitch VLAN 10
- **Status:** DONE

---

<!-- New CHG entries go here, above the Archived Sessions section -->

## Archived Sessions

---

_No archived sessions yet. When this active session is closed (on next doc version bump), it will be archived here with a link to the updated docs._
