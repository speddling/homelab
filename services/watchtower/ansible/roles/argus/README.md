# argus

Installs and configures the Argus MCP server on Watchtower — provides Claude
with read-only access to Alertmanager and Prometheus configs, systemd state,
journald logs, and monitoring HTTP APIs.

## What it does

- Creates `argus` system user (`{{ argus_user }}`)
- Downloads and installs the Argus MCP server binary
- Writes systemd service file from template
- Configures listening port from `port_argus_mcp` (default 9800)
- Prometheus scrape config includes Argus service (if applicable)

## Variables

`argus_user`, `argus_dest_dir`, `argus_port` defined in `ansible/vars/main.yml`.

## Run

Via workflow: `Actions → Deploy Watchtower → Run workflow` (monitoring)
