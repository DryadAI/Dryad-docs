# Bring up a new dryad node (current procedure, 2026-05)

This is the procedure that actually worked on dryad-forge on 2026-05-14. It replaces the previous "custom offline ISO" plan, which is no longer the path of record.

## Assumptions

- You are on `madhatter` (10.0.4.43 on home LAN, 10.10.10.1 on cluster VLAN, also reachable via Tailscale `100.72.190.41`).
- The target node:
  - has UEFI firmware
  - has at least one NVMe disk we are happy to wipe
  - has a single primary NIC named `eno1`
  - has its MAC pre-registered in `install.sh`'s case statement (see "Adding a new node" at the end)
- The target node's ethernet is initially plugged into the **home LAN** (not the FortiSwitch / cluster VLAN). This is required so DHCP + internet work during install.
- You have a USB stick ≥ 4 GB on madhatter's `/dev/sdb` (verify with `lsblk` before any `dd`).

## Step 1 — Flash the stock Arch ISO

On madhatter:

```bash
curl -L -o /home/katalyst/iso/stock-archlinux.iso \
  https://geo.mirror.pkgbuild.com/iso/latest/archlinux-x86_64.iso

lsblk -o NAME,SIZE,RM,TRAN,MODEL | grep -v nvme

sudo dd if=/home/katalyst/iso/stock-archlinux.iso of=/dev/sdX bs=4M status=progress oflag=sync conv=fdatasync
sync && sudo eject /dev/sdX
```

Do **not** use the custom `/home/katalyst/iso/dryad-arch-*.iso`. It does not currently boot cleanly on Envy Slice hardware.

## Step 2 — Boot the target

1. Plug the USB into the target.
2. Plug ethernet into the home LAN (not the FortiSwitch).
3. Power on, hit F9 (HP) or whatever the target's boot-menu key is, pick the USB.
4. Stock archiso will land at `[root@archiso ~]#`.

## Step 3 — Prep the target for SSH

At the target's console:

```bash
passwd
systemctl start sshd
ip -4 addr show eno1 | awk '/inet / {print $2}'
```

Note the IP. Everything after this point is from madhatter — no more keyboard time on the target.

## Step 4 — Push madhatter's key and the install scripts

On madhatter:

```bash
TARGET_IP=10.0.4.101   # replace with whatever the target reported

ssh-copy-id -o StrictHostKeyChecking=accept-new root@${TARGET_IP}

rsync -rz \
  /home/katalyst/dryad-arch-iso/airootfs/root/install.sh \
  /home/katalyst/dryad-arch-iso/airootfs/root/post-forge.sh \
  /home/katalyst/dryad-arch-iso/airootfs/root/post-gate.sh \
  root@${TARGET_IP}:/root/

rsync -rz /home/katalyst/dryad-arch-iso/airootfs/root/gnupg-host/ \
  root@${TARGET_IP}:/tmp/gnupg-host/
ssh root@${TARGET_IP} 'ln -sfn /tmp/gnupg-host /root/gnupg-host'
```

If pubkey auth breaks after this with `Permission denied (publickey,password)`, run on the target:

```bash
chown root:root /root && chmod 750 /root
```

That recovers from the classic `rsync -a` ownership mistake.

## Step 5 — Run the online installer

Push `/tmp/online-install.sh` from madhatter to the target. The script must:

1. Mount the partitions created by `install.sh` steps 1–5 (or do its own parted run).
2. `pacman-key --init && pacman-key --populate archlinux`.
3. Run:

   ```bash
   pacstrap -K /mnt \
     base base-devel linux linux-firmware linux-headers \
     grub efibootmgr dosfstools networkmanager openssh \
     curl wget git tailscale docker docker-compose \
     e2fsprogs parted gptfdisk sudo vim htop tmux rsync \
     jq iproute2 iputils bind syslinux \
     dnsmasq inetutils
   ```

   `dnsmasq` and `inetutils` added based on forge/gate gaps discovered after CHG-036.

4. Do the same post-pacstrap configuration as `install.sh`'s step 7.
5. `sleep 10 && reboot`.

Kick it off from madhatter:

```bash
ssh root@${TARGET_IP} 'bash /root/online-install.sh' \
  2>&1 | tee /tmp/${HOSTNAME_TO_INSTALL}-install-online.log
```

Once the SSH session drops, the target is rebooting into the installed system at its **static** cluster IP — you can no longer reach it from the home LAN.

## Step 6 — Reach the installed node

After reboot the node is at `10.10.10.XX` on the cluster VLAN.

```bash
ssh katalyst@10.10.10.20    # forge
ssh katalyst@10.10.10.21    # gate
```

Pubkey login works because `install.sh` already deposited madhatter's `ssh-ed25519` key in `/home/katalyst/.ssh/authorized_keys`.

Madhatter must have NAT for the cluster VLAN if the new node needs internet (`iptables -t nat -A POSTROUTING -s 10.10.10.0/24 -o enp2s0f0 -j MASQUERADE`, persistent via `iptables.service` — see CHG-037).

Local console of the target: `root` / `katalyst` both have password `dryad2026`, `wheel` is NOPASSWD.

## Step 7 — Run the post-install script

On the target (over SSH):

```bash
sudo bash /root/post-forge.sh   # or post-gate.sh
```

**Known issue:** `post-forge.sh` currently logs `[post-forge] Configuring dnsmasq...` but does not actually install/configure the package. After running, verify with `systemctl is-active dnsmasq`. If `inactive`, install manually:

```bash
sudo pacman -S --noconfirm dnsmasq
# deploy /etc/dnsmasq.d/cluster-pxe.conf (DHCP 10.10.10.50-200, TFTP, PXE chain)
sudo systemctl enable --now dnsmasq
```

Tracked as a follow-up to patch the script in source.

## Step 8 — Mark the work in the changelog

Add a `CHG-NNN` entry to `~/dryad-systems/ACTIVE_CHANGELOG.md` describing the bring-up, including:

- Node hostname, MAC, static IP
- Date
- Anything that deviated from this runbook

## Adding a new node (MAC mapping)

`install.sh` detects which node it is via the MAC of `eno1`. To add a new node:

1. Boot the new node off the stock ISO once just to read its MAC: `cat /sys/class/net/eno1/address`.
2. Add a `case` arm to `install.sh`:

   ```sh
   case "$MAC" in
     9c:7b:ef:77:89:79) NODE="dryad-gate";  IP="10.10.10.21" ;;
     7c:d3:0a:78:c4:a8) NODE="dryad-forge"; IP="10.10.10.20" ;;
     XX:XX:XX:XX:XX:XX) NODE="dryad-NEW";   IP="10.10.10.NN" ;;
   esac
   ```

3. Commit and push the change before running install on the new node.

## Things this runbook deliberately does not do

- Build a custom ISO. See post-mortem.
- Pre-stage a full offline package bundle. The closure work has not been done and `pacstrap -U` will fail without it.
- Configure cluster role services at install time. Those live in `post-*.sh` and can be re-run.
- Touch FortiSwitch port config. That is a separate runbook (`docs/runbooks/fortiswitch-vlan.md`).
