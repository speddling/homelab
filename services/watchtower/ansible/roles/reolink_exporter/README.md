# reolink_exporter

Installs and configures `reolink_exporter` on Watchtower to collect metrics from
the Big Brother NVR and connected Reolink cameras.

## What it does

- Creates `reolink_exporter` system user
- Downloads and installs the exporter binary
- Configures systemd service with 9720 listening port
- Uses `vault_reolink_nvr_password` from Ansible Vault for NVR authentication
- Prometheus scrapes at `reolink_nvr` job (localhost:9720)

## Run

Via workflow: `Actions → Deploy Reolink Exporter → Run workflow`
