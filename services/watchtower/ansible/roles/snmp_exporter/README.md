# snmp_exporter

Installs and configures the snmp_exporter on Watchtower for collecting SNMP metrics
from network devices (ER605, SG2218P, EAP245s).

## What it does

- Downloads and installs snmp_exporter binary
- Writes `snmp.yml` from template (defines MIB-to-metric mappings for `if_mib`)
- Configures systemd service with 9116 listening port
- Prometheus scrapes targets via this exporter: ER605 (.10.1), SG2218P (.102),
  EAP Upstairs Hall (.100), EAP Downstairs Hall (.101)

## Variables

`vault_snmp_community` in Ansible Vault — the SNMPv2c community string.
Device IPs defined in `prometheus.yml.j2` under the snmp scrape job.

## Run

Via workflow: `Actions → Deploy Watchtower → Run workflow` (monitoring)
Manual: `ansible-playbook -i inventory.ini playbooks/monitoring.yml --vault-password-file ~/.vault_pass`
