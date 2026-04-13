When you wake up, here is your "no-nonsense" guide to getting that Ryzen tower transformed into a headless K8s node.

Since the **Ryzen 5 1600x does not have integrated graphics**, you will need to plug in a GPU (any old card) just for the initial 15-minute installation. Once SSH is enabled, you can pull the card out (if the motherboard BIOS allows "headless boot").

---

## 1. Prepare the Installation Media
1.  **Download:** Get the [Ubuntu Server 24.04 LTS ISO](https://ubuntu.com/download/server).
2.  **Flash:** Use **Rufus** or **BalenaEtcher** to flash the ISO to a USB drive.

## 2. BIOS Settings (Crucial for Ryzen/K8s)
Boot into your Asrock AB350 Pro4 BIOS (usually `F2` or `Del`):
* **OC Tweaker/CPU Configuration:** Enable **SVM Mode** (this is AMD’s virtualization). Kubernetes needs this for certain container runtimes and isolation.
* **Advanced/Storage:** Ensure SATA mode is **AHCI**.
* **Boot:** Set the **M.2 drive** as the primary boot device.
* **CSM:** If your GPU is old, you might need CSM enabled; if it's modern, leave it **Disabled (UEFI only)**.

---

## 3. The Installation Process
Plug in your ethernet cable (headless labs are 100% better on wire than Wi-Fi).

1.  **Boot from USB:** Select the M.2 drive as the destination for the OS.
2.  **Storage Layout (Important):** * **M.2 (512GB):** Mount as `/` (root). This is where your OS and the heavy Kubernetes container layers will live.
    * **HDD (2TB):** Do **not** mount this during the initial setup wizard. We will mount this later as a "Persistent Volume" or bulk storage for your cluster.
3.  **Networking:** Ensure it picks up an IP via DHCP.
4.  **Profile:** Set your username/password.
5.  **SSH Setup:** **Check the box that says "Install OpenSSH Server."** This is the most important step for a headless build.
6.  **Featured Snaps:** Leave these blank for now; you’ll want to install Kubernetes tools manually to learn the process.

---

## 4. Post-Install Headless Setup
Once the install finishes and the system reboots, you can unplug the monitor and keyboard. From your main computer, log in via terminal:
`ssh username@your-tower-ip`

### Prepare the Drives for K8s
Update the system and format that 2TB drive for storage:
```bash
sudo apt update && sudo apt upgrade -y
# Find your 2TB drive (likely /dev/sdb)
lsblk
# Format and mount it to a dedicated folder for K8s storage
sudo mkfs.ext4 /dev/sdb
sudo mkdir /mnt/storage
sudo mount /dev/sdb /mnt/storage

---

## 5. Kubernetes Foundation
Since you have a single powerful node, I highly recommend installing **K3s**. It’s lightweight but fully compliant.

**To install K3s on your new server:**
```bash
curl -sfL [https://get.k3s.io](https://get.k3s.io) | sh -
# Check if it's running
sudo kubectl get nodes
```
## One Final "Ryzen" Tip
Early Ryzen chips (like the 1600x) occasionally had a "C-state" bug on Linux that caused idle freezes. If your tower randomly crashes when you aren't using it:
1.  Edit your GRUB config: `sudo nano /etc/default/grub`
2.  Add `processor.max_cstate=1` to the `GRUB_CMDLINE_LINUX_DEFAULT` line.
3.  Run `sudo update-grub` and reboot.