# zombatron_importer

Deploys the Zombatron Importer — a Slack bot that receives Minecraft world
import requests via Slack API and stages .mcworld files on Monolith for the
`import-minecraft-world.yml` workflow.

## Status: pending migration

Like Scribe, this runs on **apex** and is targeted for migration to Construct.

## What it does

- Creates a Python venv at `~/.venv/zombatron-importer` (uses Homebrew Python)
- Installs dependencies from `services/apex/zombatron-importer/requirements.txt`
- Deploys a macOS launchd plist as a background service
- Uses Slack Bolt API with Socket Mode (bot token + app token from Ansible Vault)

## Variables

`zombatron_importer_slack_channel` (C0B5C20R1SB), `zombatron_importer_monolith_host`,
`zombatron_importer_import_path` (/opt/minecraft/import/realm.mcworld).

## Run

Via workflow: `Actions → Deploy Zombatron Importer → Run workflow` (manual)
