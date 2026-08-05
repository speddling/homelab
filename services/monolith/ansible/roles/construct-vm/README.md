# construct-vm

Provisions the Construct development VM on Monolith using QEMU/KVM + cloud-init.

## What it does

- Installs qemu-kvm, qemu-utils, libvirt-daemon-system, libvirt-clients
- Creates an 80GB LVM logical volume from monolith's NVMe
- Downloads Debian 12 cloud image as the base disk
- Generates cloud-init user-data/meta-data and builds a cloud-init ISO
- Creates a `construct.service` systemd unit that manages the VM lifecycle
- SSH accessible via Tailscale (100.67.178.34) or port-forward `ssh -p 2222 speddling@monolith`

## Variables

Defined in `ansible/vars/main.yml`: `construct_vm_memory`, `construct_vm_cores`,
`construct_vm_ssh_port`, `construct_vm_tailscale_ip`.

## Run

Via workflow: `Actions → Bootstrap Construct VM → Run workflow`
Manual: `ansible-playbook -i inventory.ini playbooks/construct.yml --vault-password-file ~/.vault_pass`
