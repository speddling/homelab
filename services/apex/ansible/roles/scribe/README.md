# scribe

Deploys Scribe — Claude's git control plane — on **apex** (macOS).

## Status: pending migration

Scribe currently runs on apex (MacBook Air M4). It will migrate to the Construct
VM when the apex workstation is repurposed as a pure workstation without
long-running services.

## What it does

- Configures SSH authorized key for GitHub Actions deploy
- Creates a Python venv at `~/.venv/scribe` (uses Homebrew Python)
- Installs dependencies from `services/apex/scribe/requirements.txt`
- Deploys a macOS launchd plist as a background service
- Smoke-tests that the service is listening on port 8765

## Variables

`scribe_port` (8765), `scribe_python` (/opt/local/bin/python3.12),
`scribe_repo_roots` (paths Claude can access for git operations).

## Run

Via workflow: `Actions → Deploy Monolith → Run workflow` (Scribe playbook runs from apex runner)
