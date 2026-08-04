# LWA Infra -- Construct Runbook
> Last updated: 2026-08-04

**Host:** Monolith (`100.127.193.14`, Tailscale)
**Guest:** Debian 12 (Bookworm)
**SSH (Tailscale):** `speddling@100.95.178.6` (Tailscale name: `construct`)
**SSH (port forward):** `ssh -p 2222 speddling@100.127.193.14` → `construct:22`
**User:** `speddling` (sudo)

---

## Purpose

Persistent development environment for AI-assisted coding, accessible from any laptop via Tailscale. Built to run Pi coding agent and wmux for seamless session continuity across locations.

---

## Current State

Fully automated provisioning via Ansible + cloud-init. VM lifecycle managed by systemd service.

---

## How It's Running

QEMU/KVM process managed by systemd service `construct.service`:

```bash
sudo systemctl status construct
sudo systemctl start construct
sudo systemctl stop construct
```

**Underlying QEMU command** (from systemd unit):
```bash
/usr/bin/qemu-system-x86_64 \
  -enable-kvm \
  -m 16384 \
  -smp 8 \
  -machine q35 \
  -cpu host \
  -drive file=/vm/construct/disk.img,if=virtio,format=qcow2 \
  -drive file=/vm/construct/cloud-init.iso,media=cdrom,readonly=on \
  -netdev user,id=net0,hostfwd=tcp::2222-:22 \
  -device virtio-net-pci,netdev=net0 \
  -display none \
  -serial file:/tmp/construct-console.log \
  -daemonize \
  -pidfile /run/construct.pid
```

---

## Storage

All on NVMe boot drive LVM (`/dev/ubuntu-vg` on `/dev/nvme0n1p3`):

| Path | Size | Purpose |
|---|---|---|
| `/dev/ubuntu-vg/construct` | 80G LV | Mounted at `/vm/construct` |
| `/vm/construct/disk.img` | 3.4G qcow2 | Guest OS disk (grows dynamically) |
| `/vm/construct/debian-12-generic-amd64.qcow2` | 427M | Base cloud image (permanent artifact) |
| `/vm/construct/cloud-init.iso` | ~50KB | Cloud-init data (regenerated on provision) |

**Actual disk usage:** Check with `sudo qemu-img info /vm/construct/disk.img`

---

## Initial Provisioning

Fully automated via Ansible. Run once to create the VM:

```bash
# From your workstation
cd services/monolith/ansible
ansible-playbook -i inventory.ini playbooks/construct.yml \
  --vault-password-file ~/.vault_pass
```

**What the playbook does:**
1. Creates LVM logical volume (`/dev/vg-root/construct`)
2. Mounts it at `/vm/construct`
3. Downloads Debian 12 generic cloud image
4. Creates 80G qcow2 disk from cloud image
5. Generates cloud-init ISO with:
   - User `speddling` + SSH key
   - Hostname `construct`
   - Tailscale auth key (from vault)
   - Dev tool installation scripts
6. Creates systemd service
7. Starts the VM

**First boot** (via cloud-init):
- Creates user, installs SSH key
- Sets hostname
- Installs and authenticates Tailscale (`--accept-dns=false`, DNS managed manually)
- Installs: git, gh, node, python, go, tmux, vim, curl, jq
- Installs NVM + Node.js 22
- Installs Pi coding agent (under Node 22)
- Installs and builds wmux
- Creates wmux systemd user service (auto-started)
- Fixes DNS (resolv.conf → watchtower DNS, chattr +i)

---

## Cloud-Init Details

Cloud-init data is injected via ISO at `/vm/construct/cloud-init.iso`.

**user-data.yml** (rendered from Ansible template):
- SSH key from vault
- Package list
- Tailscale auth key
- Post-boot scripts (runcmd)

**meta-data.yml:**
- instance-id: `construct-vm`
- local-hostname: `construct`

**To regenerate cloud-init ISO** (if you need to reset the VM):
```bash
cd services/monolith/ansible
ansible-playbook -i inventory.ini playbooks/construct.yml \
  --tags cloud-init \
  --vault-password-file ~/.vault_pass
```

Then restart the VM:
```bash
sudo systemctl restart construct
```

---

## Network Access

### Via Tailscale (Primary)
```bash
ssh speddling@100.95.178.6
```

**Recommended `~/.ssh/config` entry:**
```text
Host construct
  HostName construct.tailea7d70.ts.net
  User speddling
```

Then you can use:
```bash
ssh construct
```

> **Note:** After the latest rebuild (2026-08-04), the old Tailscale node
> (`construct` at IP `100.67.178.34`) was deleted and the rebuilt VM was
> renamed to `construct` (IP `100.95.178.6`). Tailscale assigns a new machine
> identity on disk rebuild, so the node key changes each time.


### Via Port Forward (Fallback)
If Tailscale is down or not yet authenticated:
```bash
ssh -p 2222 speddling@100.127.193.14
```

Host port 2222 forwards to construct's port 22.

---

## Post-Boot Configuration

### Start wmux
```bash
ssh speddling@construct
wmux new dev    # or wmux attach dev if session exists
```

wmux is also running as a systemd user service (`wmux.service`) on construct,
accessible via browser over Tailscale:

```text
http://construct.tailea7d70.ts.net:3478/?token=QatyoM5m45aVPvai6pNNk0UWSHQl487c
```

> **Tip:** You can also use `http://100.95.178.6:3478/...` (Tailscale IP) or
> `http://construct.littlewolfacres.com:3478/...` if you configure a DNS record.
> See [DNS Names](#dns-names) below.

- **No SSH tunnel needed** — construct is directly reachable at its Tailscale IP
- **Token** is stored in `~/.wmux/token` inside the VM
- **Machine**: "construct" is configured as `kind: local` (spawns a bash PTY
  on construct itself)
- **Auth mode**: `shared-or-login` (shared token URL by default)
- **Management**:
  ```bash
  systemctl --user status wmux.service
  systemctl --user restart wmux.service
  journalctl --user -u wmux.service -f
  ```

> **Dev mode**: The service runs `npm run dev` (tsx hot-reload). The pre-built
> `dist/` is available via `npm start` for production mode if performance
> becomes an issue.

### wmux 403 Forbidden (wrong Host header)

**Symptom**: Accessing wmux via a custom domain (`construct.littlewolfacres.com`)
returns `403 Forbidden` with `{"error":"forbidden_host"}`.

**Cause**: wmux validates the `Host` header against an allowlist. By default,
only the bound IP, `localhost`, and `*.ts.net` names are accepted. Custom
domains must be added via `WMUX_ALLOWED_HOSTS`.

**Fix**: Add `construct.littlewolfacres.com` to `WMUX_ALLOWED_HOSTS` in the
wmux service file:
```bash
# Edit the service file
nano /home/speddling/.config/systemd/user/wmux.service
# Add: Environment=WMUX_ALLOWED_HOSTS=construct.littlewolfacres.com

# Restart the service
systemctl --user daemon-reload
systemctl --user restart wmux.service
```

This is already handled by the cloud-init template (variable:
`construct_wmux_allowed_hosts`).

### DNS Names

wmux is reachable via several DNS names, all resolving to the same Tailscale IP
(`100.95.178.6`):

| URL | How it works | Setup |
|-----|-------------|-------|
| `construct.tailea7d70.ts.net:3478` | Tailscale MagicDNS (automatic) | None — auto-generated |
| `100.95.178.6:3478` | Tailscale direct IP | None |
| `construct.littlewolfacres.com:3478` | Your custom domain | Add A record → `100.95.178.6` |

**For `construct.littlewolfacres.com`:**
- Add an **A record** (gray-cloud / DNS-only) on Cloudflare pointing to
  `100.95.178.6` (Tailscale IP). This is only reachable from inside the
  tailnet — the `100.x.x.x` IP is CGNAT and not publicly routable.
- Optionally add the same A record on your local DNS (watchtower at
  `192.168.30.11`) so `littlewolfacres.com` resolves correctly for tailnet
  devices via the Tailscale Split DNS route.
- **Do not** proxy (orange-cloud) the Cloudflare record — Cloudflare cannot
  reach Tailscale IPs.

The cloud-init automatically detects the Tailscale DNS name and sets it as
`WMUX_PUBLIC_URL` in the service file. This survives IP changes across rebuilds.

### Portless Access (port 80 → 3478 redirect)

wmux listens on port 3478, but for convenience a portless URL (`http://construct.littlewolfacres.com/`)
is available via an iptables redirect installed by cloud-init:

```bash
# Redirects port 80 → 3478 on the Tailscale interface
iptables -t nat -C PREROUTING -i tailscale0 -p tcp --dport 80 -j REDIRECT --to-port 3478
```

This rule is idempotent (skips if already present) and persists across reboots via
`iptables-persistent` (installed as an apt package in cloud-init). It only applies
to traffic arriving via the Tailscale interface — localhost traffic is unaffected.

**Browser URL**: `http://construct.littlewolfacres.com/?token=QatyoM5m45aVPvai6pNNk0UWSHQl487c`

### Install Additional Tools
```bash
# Inside construct
sudo apt update
sudo apt install <package>
```

### Update Pi
```bash
npm update -g @earendil-works/pi-coding-agent
```

---

## Maintenance

### Expand Disk (if needed)
1. Grow the LV on monolith:
   ```bash
   sudo lvextend -L +20G /dev/ubuntu-vg/construct
   ```

2. Grow the qcow2 image:
   ```bash
   sudo qemu-img resize /vm/construct/disk.img +20G
   ```

3. Inside construct, grow the filesystem:
   ```bash
   sudo growpart /dev/vda 1
   sudo resize2fs /dev/vda1
   ```

### Backup
```bash
# On monolith
sudo qemu-img convert -O qcow2 -c /vm/construct/disk.img \
  /mnt/hdd-c/backups/construct-$(date +%Y%m%d).qcow2
```

### Rebuild from Scratch
```bash
# On monolith
sudo systemctl stop construct
sudo rm -rf /vm/construct/*

# From your workstation
cd services/monolith/ansible
ansible-playbook -i inventory.ini playbooks/construct.yml \
  --vault-password-file ~/.vault_pass
```

---

## Troubleshooting

### VM won't boot
Check systemd service logs:
```bash
sudo journalctl -u construct -f
```

### Can't SSH via Tailscale
1. Check Tailscale status inside the VM (via port forward):
   ```bash
   ssh -p 2222 speddling@100.127.193.14
   sudo tailscale status
   ```

2. Check auth key in vault is valid:
   ```bash
   ansible-vault view ansible/vars/vault.yml | grep tailscale_auth_key
   ```

3. Check Tailscale admin console for construct node

### DNS Resolution Failure (Tailscale upstream DNS missing)

**Symptom**: Pi reports "Connection error" / "Retry failed after 3 attempts"
when trying to reach model provider APIs (api.github.com, api.kilo.ai, etc.).
DNS lookups return empty.

**Root cause**: The Tailscale tailnet has no upstream DNS resolvers configured
in the admin console. Tailscale's managed stub resolver (100.100.100.100) can
only resolve internal `.ts.net` domains, not public internet domains.

**Fix applied** (in cloud-init):
- `--accept-dns=false` passed to `tailscale up` (prevents Tailscale from
  managing `/etc/resolv.conf`)
- `/etc/resolv.conf` written with the local DNS server (watchtower at
  `192.168.30.11`)
- `chattr +i /etc/resolv.conf` prevents overwrites

**Permanent fix** (to do once in [Tailscale admin console](https://login.tailscale.com/admin/dns)):
1. Go to **DNS → Nameservers**
2. Add `192.168.30.11` (or `1.1.1.1`, `8.8.8.8`) as upstream DNS servers
3. After applying, revert the cloud-init local fix (remove `--accept-dns=false`
   and the manual resolv.conf writes) — Tailscale will manage DNS properly
   tailnet-wide

### Node.js Version Mismatch (crypto.hash / regex v flag)

**Symptom**: `SyntaxError: Invalid regular expression flags` when running pi,
or `crypto.hash is not a function` when running wmux.

**Cause**: Debian 12 ships Node.js 18. Both pi-coding-agent and wmux require
Node.js 22+ (they use `crypto.hash()` and regex `v` flag).

**Fix**: Node.js 22 is installed via [nvm](https://github.com/nvm-sh/nvm)
and set as the default. The wmux systemd service sets `PATH` to use
`~/.nvm/versions/node/v22.x/bin`. nvm init is sourced in both `.bashrc`
and `.profile` so non-interactive shells (SSH commands, systemd) also pick up
Node 22.

> **Note:** Debian 12's system npm sets `npm_config_prefix=/usr/local`, which
> breaks nvm ("nvm is not compatible with npm_config_prefix"). The cloud-init
> template unsets this in the nvm init snippets written to `.bashrc`/`.profile`.

**For automation**: The cloud-init installs nvm, installs Node 22, and writes
nvm init to `.bashrc`/`.profile`. If rebuilding, verify with:
```bash
bash -lc 'node --version'   # should be v22.x
```

### wmux Service Crash Loop (durable endpoint parent directory must be owner-only)

**Symptom**: wmux service crashes on startup with
`durable endpoint parent directory must be owner-only` and restarts in a loop
(restart counter climbs rapidly).

**Root cause**: wmux's `DurableEndpointStore` checks that the `~/.wmux/`
directory has mode `0700` (owner-only). If the directory was pre-created by
another process with `0755` permissions (the default under umask 022), wmux
refuses to start. The `ensureSecureParent()` check creates the directory if
it doesn't exist, but does **not** chmod it if it already exists with wrong
permissions.

**Fix** (on construct VM):
```bash
chmod 700 ~/.wmux
systemctl --user restart wmux.service
```

**Permanent fix**: The cloud-init template (`cloud-init-user-data.j2`) now
includes `mkdir -p + chmod 700 ~/.wmux` in the `setup-wmux-service.sh` script
before starting the service. This ensures correct permissions even if the
directory was pre-created by an earlier process.

### Cloud-init didn't run
Check cloud-init logs inside construct:
```bash
ssh -p 2222 speddling@100.127.193.14
sudo cloud-init status
sudo cat /var/log/cloud-init.log
sudo cat /var/log/cloud-init-output.log
```

### High CPU usage
Check QEMU process:
```bash
ps aux | grep qemu | grep construct
top -p $(cat /var/run/construct.pid)
```

---

## Connect

```bash
# SSH via Tailscale (primary)
ssh speddling@construct.tailea7d70.ts.net
# or, with ~/.ssh config entry above:
ssh construct

# SSH via port forward (fallback)
ssh -p 2222 speddling@100.127.193.14

# Start wmux session
wmux attach dev

# Access wmux in browser
#   http://construct.tailea7d70.ts.net:3478/?token=$(cat ~/.wmux/token)

# Check VM status
sudo systemctl status construct

# Restart VM
sudo systemctl restart construct

# Stop VM
sudo systemctl stop construct

# View console output (if running in foreground for debugging)
sudo journalctl -u construct -f
```

---

## TODO

- [ ] Configure Tailscale upstream DNS in admin console (192.168.30.11) — eliminates need for local resolv.conf fix
- [ ] Add `construct.littlewolfacres.com` A record → `100.95.178.6` on Cloudflare (DNS-only/gray-cloud) and watchtower local DNS
- [ ] Evaluate switching wmux from dev mode (`npm run dev`) to production mode (`npm start`) for better performance
- [ ] Automated backup cron job (daily qcow2 snapshot to hdd-c)
- [ ] Prometheus node_exporter inside construct
- [ ] UFW rules on Monolith (SSH port 2222 restricted to LAN + Tailscale)
- [ ] Test disaster recovery (full rebuild from playbook)
- [ ] Evaluate serial console access (`/tmp/construct-console.log`) for better debugging vs -nographic
