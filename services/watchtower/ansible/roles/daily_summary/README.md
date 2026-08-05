# daily_summary

Deploys the daily-summary Python script that posts a health digest to Slack
`#sentinel` at 8 AM and 8 PM America/New_York.

## What it does

- Renders `daily-summary.py.j2` template to `/usr/local/bin/daily-summary.py`
- Creates systemd service + timer (runs at 08:00 and 20:00 daily)
- Queries Prometheus for system health and formats a Slack message
- Pings a healthchecks.io check (`vault_healthchecks_daily_summary_url`) on each run

## Variables

`vault_healthchecks_daily_summary_url` in Ansible Vault — the healthchecks.io
ping URL for this script (separate from the Alertmanager watchdog URL).

## Run

Via workflow: `Actions → Deploy Watchtower → Run workflow` (monitoring)
