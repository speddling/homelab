# tmobile_exporter

Installs and configures `tmobile_exporter` on Watchtower to collect metrics from
the T-Mobile FAST 5688W 5G gateway.

## What it does

- Creates `tmobile_exporter` system user
- Downloads and installs the exporter binary
- Configures systemd service with 9719 listening port
- Polls the gateway's local API for signal strength, band, uptime, traffic
- Prometheus scrapes at `tmobile` job (localhost:9719)

## Run

Via workflow: `Actions → Deploy T-Mobile Exporter → Run workflow`
