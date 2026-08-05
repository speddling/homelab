# nut

Installs NUT (Network UPS Tools) client for monitoring the CyberPower CP1000PFCLCD UPS.

## Status: planned

The UPS hardware is purchased. This role is written and ready for deployment.
When run, it will:
- Install `nut-client` package
- Configure upsmon to monitor the UPS on localhost
- Set up the `upsmon` systemd service
- Configure NUT password from `vault_nut_monitor_password` in Ansible Vault

## Pending

- Physical UPS needs to be connected to Watchtower
- Prometheus scrape config for the nut_exporter target needs to be added
- Alert rules for `UPSOnBattery` / `UPSLowBattery` need to be written

## Run

Via workflow: `Actions → Deploy Watchtower → Run workflow` (monitoring)
