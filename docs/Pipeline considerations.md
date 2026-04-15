
**GitHub Actions** and **Terraform**,
building a "GitOps Lite" pipeline -  high-level approach for that transition from CloudFormation to Terraform in a local lab:

1. The State File "Gotcha"

Unlike CloudFormation, which stores state in the AWS backend, Terraform needs a place to keep its `.tfstate`. Since this is a local lab:

- **Local State:** Easiest for day one, but bad for GitHub Actions.
- **S3/GCS Backend:** You can actually use a free-tier S3 bucket to hold your homelab state so your GitHub Actions can reach it.
- **Alternative:** Use **Terraform Cloud** (free for small teams) to manage the state and locking without needing an external bucket.

2. Provider Mapping

In CloudFormation, you’re mostly dealing with `AWS::*` resources. For your lab, you’ll likely be using:

- **`telmate/proxmox`** or **`bpg/proxmox`** (if you go the hypervisor route).
- **`hashicorp/kubernetes`** and **`hashicorp/helm`** for the Navidrome layer.
- **`cloudflare/cloudflare`** if you eventually want to manage local DNS or Tunnels via code.

3. Hardening + GitHub Actions

Since you're hardening the box first (presumably via SSH/Ansible or manual scripts), you can use GitHub Actions to:

- **Lint your YAMLs:** Run `kube-linter` or `yamllint` before any push.
- **Terraform Plan/Apply:** Use a **self-hosted runner** (on your Ryzen box) to execute the Terraform applies. This avoids the need to expose your lab's K8s API to the public internet for the GitHub-hosted runners to reach.