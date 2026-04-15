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
````
sudo apt update && sudo apt upgrade -y
# Find your 2TB drive (likely /dev/sdb)
lsblk
# Format and mount it to a dedicated folder for K8s storage
sudo mkfs.ext4 /dev/sdb
sudo mkdir /mnt/storage
sudo mount /dev/sdb /mnt/storage
````

## 5. Kubernetes Foundation
Since you have a single powerful node, I highly recommend installing **K3s**. It’s lightweight but fully compliant.

**To install K3s on your new server:**
```bash
curl -sfL https://get.k3s.io | sh -
# Check if it's running
sudo kubectl get nodes
```
## One Final "Ryzen" Tip
Early Ryzen chips (like the 1600x) occasionally had a "C-state" bug on Linux that caused idle freezes. If your tower randomly crashes when you aren't using it:
1.  Edit your GRUB config: `sudo nano /etc/default/grub`
2.  Add `processor.max_cstate=1` to the `GRUB_CMDLINE_LINUX_DEFAULT` line.
3.  Run `sudo update-grub` and reboot.




---------------------


The ASRock AB350 Pro4 is a classic workhorse for this kind of project, especially with a Ryzen 5 and 32GB of RAM. Since you are building a headless Ubuntu lab with Kubernetes, here are the hardware-specific "gotchas" and strategies for your storage layout based on that board's architecture: 

1. Storage Slot "Rules"

The AB350 Pro4 has a specific way it handles bandwidth that will dictate where you plug in your drives:

- **M2_1 (Ultra M.2):** This slot is directly connected to your Ryzen CPU. **Use this for your M.2 boot volume.** Note that using this slot **disables** the `PCIE4` slot (the second full-length PCIe slot).
- **M2_2:** This slot **only supports SATA M.2 drives**, not NVMe. If you ever buy an M.2 for this slot, it shares bandwidth with the `SATA3_3` port; using one disables the other.
- **SATA Ports:** You have **6 SATA3 ports**. Four are from the AMD chipset (SATA3_1~4) and two are from an ASMedia chip (SATA3_A1~A2). For the best performance with your 2TB and 512GB SSDs, use the **chipset ports (SATA3_1 or SATA3_2)** first. 

2. Headless Ryzen 5 Considerations

Since your Ryzen 5 likely does not have integrated graphics (IGPU), the motherboard's onboard HDMI/VGA ports will be **inactive**. 

- **Initial Setup:** You will need a temporary discrete GPU just to get the OS installed and SSH enabled.
- **Headless Boot:** Some versions of this board may refuse to boot without a GPU detected. If you run into "no-boot" issues once you pull the card for headless operation, you may need a **cheap "dummy" plug** or to check the BIOS for a "Halt on Error" setting to ignore the missing VGA. 

3. Recommended Storage Manifest Strategy

With your 512GB Crucial SSD now in the mix, your Kubernetes `PersistentVolume` (PV) strategy should look like this:

| Drive             | Purpose                  | K8s Volume Type                  |
| ----------------- | ------------------------ | -------------------------------- |
| **M.2 NVMe**      | OS / K8s System          | Local Boot                       |
| **512GB Crucial** | Navidrome DB / Art Cache | `ReadWriteOnce` PV (Fast I/O)    |
| **2TB SSD**       | 1.3TB Music Library      | `ReadOnlyMany` PV (Bulk Storage) |
