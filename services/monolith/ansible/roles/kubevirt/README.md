# kubevirt

Installs KubeVirt and CDI (Containerized Data Importer) operators on the k3s cluster.
Also creates the obelisk namespace.

## Status

**Archived — do not use.** KubeVirt was attempted for the Obelisk Windows 11 VM but
failed due to OVMF/virtio-blk incompatibility and CDI import issues with Microsoft's
session-authenticated ISO URLs. Obelisk now runs as a plain QEMU/KVM process.

See `docs/obelisk-runbook.md` for the current QEMU/KVM procedure.
