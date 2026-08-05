# promtail

Installs and configures Promtail on Watchtower as a log shipper.

## What it does

- Creates `promtail` system user and adds it to the `systemd-journal` group
- Downloads and installs promtail binary
- Writes `/etc/promtail/config.yml` from `templates/config.yml.j2`
- Configures systemd service with 9080 listening port

## Sources

- Watchtower's systemd journal (all units)
- ER605 syslog listener on UDP/TCP 1514 (wired up, not yet configured on ER605)

## Run

Via workflow: `Actions → Deploy Watchtower → Run workflow` (monitoring)
