# loki

Installs and configures Loki on Watchtower for log aggregation in Prometheus/Grafana.

## What it does

- Creates `loki` system user
- Downloads and installs loki binary (monolithic mode)
- Writes `/etc/loki/loki-config.yaml` from template
- Configures systemd service with 3100 listening port
- 60-day retention, filesystem storage under `/var/lib/loki`

## Run

Via workflow: `Actions → Deploy Watchtower → Run workflow` (monitoring)
