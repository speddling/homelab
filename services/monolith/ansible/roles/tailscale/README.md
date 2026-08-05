# tailscale

Installs and configures Tailscale on Monolith (Ubuntu 24.04).

## What it does

- Adds Tailscale apt repository and installs the package
- Authenticates via `--auth-key` from `vault_tailscale_key` in Ansible Vault
- Enables IP forwarding and sets up exit routes
- Tags the node as `tag:monolith`

## Variables

`vault_tailscale_key` in `ansible/vars/vault.yml` — the Tailscale auth key
created in the Tailscale admin console.

## Run

Via workflow: `Actions → Deploy Monolith → Run workflow` (Tailscale role)
Manual: `ansible-playbook -i inventory.ini playbooks/tailscale.yml --vault-password-file ~/.vault_pass`
