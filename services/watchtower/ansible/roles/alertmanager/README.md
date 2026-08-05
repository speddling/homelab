# alertmanager

Installs and configures Alertmanager on Watchtower for alert routing and notification
deduplication.

## What it does

- Creates `alertmanager` system user
- Downloads and installs alertmanager binary
- Writes `/etc/alertmanager/alertmanager.yml` from `templates/alertmanager.yml.j2`
- Writes `/etc/alertmanager/amtool.yml` for silences
- Configures systemd service with 9093 listening port

## Alert routing

All alerts route to Slack `#sentinel` channel via webhook URL stored in
`vault_slack_webhook_url` (Ansible Vault). The `DeadManSwitch` alert is routed
to a separate `watchdog` receiver that pings healthchecks.io every 5 minutes.

## Run

Via workflow: `Actions → Deploy Watchtower → Run workflow` (monitoring)
Manual: `ansible-playbook -i inventory.ini playbooks/monitoring.yml --vault-password-file ~/.vault_pass`
