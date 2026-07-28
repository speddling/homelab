# Construct

Debian 12 development VM running on monolith via QEMU/KVM with cloud-init automation.

## Purpose

Persistent development environment accessible via Tailscale and wmux for seamless multi-location coding sessions. Named "construct" as a homage to both the Matrix construct loading program and the ops philosophy of building (constructing) infrastructure.

## Current Setup

- **QEMU/KVM** process on monolith managed by systemd
- **Disk image:** `/vm/construct/disk.img` (80G qcow2, dynamically allocated)
- **Cloud image:** `/vm/construct/debian-12-generic-amd64.qcow2` (base artifact)
- **Cloud-init ISO:** `/vm/construct/cloud-init.iso` (regenerated at provision time)
- **SSH:** `speddling@100.67.178.34` via Tailscale (`construct-1`) or `ssh -p 2222 speddling@monolith` via port forward
- **OS:** Debian 12 (Bookworm)

## What's Here

- `ansible/` — Ansible automation for VM provisioning
- `cloud-init/` — Cloud-init templates (user-data, meta-data)
- `scripts/` — Helper scripts for dev tool installation

## Deployment

See `docs/construct-runbook.md` for full operational details.

Initial provisioning:
```bash
# Run from your workstation
.github/workflows/bootstrap-construct.yml  # via GitHub Actions, or:

# Run manually
cd services/monolith/ansible
ansible-playbook -i inventory.ini playbooks/construct.yml \
  --vault-password-file ~/.vault_pass
```

## Access

```bash
# Via Tailscale (primary)
ssh speddling@100.67.178.34

# Via port forward (fallback)
ssh -p 2222 speddling@monolith

# Start wmux session
wmux attach dev
```

## Stack

Pre-installed via cloud-init:
- Tailscale (auto-authenticated)
- Git, GitHub CLI
- Node.js (LTS), Python 3, Go
- tmux, vim, curl, jq
- Pi coding agent
- wmux (your friend's multiplexer)

## Storage

Allocated from monolith's NVMe boot drive (`/dev/vg-root/construct`):
- 80GB LVM logical volume
- qcow2 format (grows dynamically as used)
- Fast Samsung PM9A1 NVMe backing

## Notes

- Unlike Obelisk (Windows), this VM uses virtio drivers for optimal Linux performance
- Headless (no VNC) — SSH only
- Cloud-init handles all initial provisioning (no manual install steps)
- Systemd service (`construct.service`) manages lifecycle
