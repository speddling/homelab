# LWA Infra -- Construct Runbook
> Last updated: 2026-07-28

**Host:** monolith (`192.168.30.10`)
**Guest:** Debian 12 (Bookworm)
**SSH (Tailscale):** `construct.yourtailnet.ts.net`
**SSH (port forward):** `monolith:2222` → `construct:22`
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
  -nographic \
  -serial mon:stdio \
  -daemonize \
  -pidfile /var/run/construct.pid
```

---

## Storage

All on NVMe boot drive LVM:

| Path | Size | Purpose |
|---|---|---|
| `/dev/vg-root/construct` | 80G LV | Mounted at `/vm/construct` |
| `/vm/construct/disk.img` | 80G qcow2 | Guest OS disk (grows dynamically) |
| `/vm/construct/debian-12-generic-amd64.qcow2` | ~500MB | Base cloud image (permanent artifact) |
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
- Installs and authenticates Tailscale
- Installs: git, gh, node, python, go, tmux, vim, curl, jq
- Installs Pi coding agent
- Installs wmux

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
Once the VM joins your tailnet (automatic via cloud-init):
```bash
ssh speddling@construct
```

### Via Port Forward (Fallback)
If Tailscale is down or not yet authenticated:
```bash
ssh -p 2222 speddling@monolith
```

Host port 2222 forwards to construct's port 22.

---

## Post-Boot Configuration

### Start wmux
```bash
ssh speddling@construct
wmux new dev    # or wmux attach dev if session exists
```

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
   sudo lvextend -L +20G /dev/vg-root/construct
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
   ssh -p 2222 speddling@monolith
   sudo tailscale status
   ```

2. Check auth key in vault is valid:
   ```bash
   ansible-vault view ansible/vars/vault.yml | grep tailscale_auth_key
   ```

3. Check Tailscale admin console for construct node

### Cloud-init didn't run
Check cloud-init logs inside construct:
```bash
ssh -p 2222 speddling@monolith
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
ssh speddling@construct

# SSH via port forward (fallback)
ssh -p 2222 speddling@monolith

# Start wmux session
wmux attach dev

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

- [ ] Automated backup cron job (daily qcow2 snapshot to hdd-c)
- [ ] Prometheus node_exporter inside construct
- [ ] UFW rules on monolith (SSH port 2222 restricted to LAN + Tailscale)
- [ ] AdGuard rewrite: `construct.littlewolfacres.com` → Tailscale IP
- [ ] Document wmux session management best practices
- [ ] Test disaster recovery (full rebuild from playbook)
- [ ] Evaluate SPICE for better console access vs nographic
