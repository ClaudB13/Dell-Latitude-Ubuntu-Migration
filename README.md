# Dell-Latitude-Ubuntu-Migration
Wiping drive to install Ubuntu
# Dell Latitude 7420 — Windows Lockout Bypass & Ubuntu Installation

A comprehensive documentation of a successful operating system recovery on an old, out-of-commission Dell Latitude 7420 laptop. The machine was initially bricked by a Windows BitLocker encryption lockout and successfully repurposed and given a second life by performing a clean installation of Ubuntu Linux.

---

## Hardware Environment
* **Device:** Dell Latitude 7420 Laptop (Legacy/Repurposed Hardware)
* **Storage:** Internal NVMe SSD (Originally formatted with NTFS, Windows 11, and BitLocker Encryption)
* **Target OS:** Ubuntu Linux 24.04 / 26.04 LTS
* **Installation Media:** 8GB+ USB Flash Drive

---

## The Problem: Windows BitLocker Loop on Legacy Hardware
This old laptop was completely inaccessible due to a Windows BitLocker recovery screen demanding a 48-digit encryption key (`Recovery Key ID: 2DF428A4...`). Because the machine had been out of active rotation, the original recovery key was entirely unavailable.
* Attempting to reset the PC via the native Windows Recovery Environment (`Shift + Restart`) failed because the built-in Windows repair tools refused to modify or format the encrypted partition without the decryption key.
* Choosing **"Skip this drive"** repeatedly looped back to the same lock screen.

---

## The Initial Installation Attempt

### Step 1: Preparing the Bootable Installation Media
*Using a secondary, functional computer:*
1. Downloaded the official desktop ISO image from **ubuntu.com**.
2. Connected a flash drive and used **Rufus** to write the ISO image to the USB.
3. Formatted the installation drive to be bootable via **UEFI** partition scheme.

### Step 2: Overriding the Native Boot Manager
1. Powered down the target Dell laptop completely and connected the AC adapter.
2. Inserted the prepared Ubuntu USB drive into an available port.
3. Powered on the machine and immediately tapped **`F12`** repeatedly to access the Dell **One-Time Boot Menu**.
4. Selected the **USB Storage Device** under the UEFI boot device section to force the hardware to ignore the internal drive's Windows Boot Manager.

### Step 3: Troubleshooting Hardware Interrupts & Low Battery (SupportAssist)
During the initial boot sequence, the old battery threw a firmware interrupt due to the heavy processing required to load the live system image:
> `WARNING: Battery is critically low.`

* **Fix:** Left the AC charger connected, allowed the battery to gain a minimal safe charge, and selected **Continue** to pass the hardware diagnostic barrier.

### Step 4: Running the Installer Wizard (First Attempt)
Once inside the Ubuntu live environment, the installation wizard was configured:
1. Selected **Normal Installation**. 
2. **Stability Tweak:** **Unchecked** *"Download updates while installing Ubuntu"* to isolate the installation to local USB files and prevent network freezes.
3. **Hardware Compatibility:** **Checked** *"Install third-party software for graphics and Wi-Fi hardware"*.
4. **Disk Partitioning:** Selected **"Erase disk and install Ubuntu"** and completed the installer prompts.

---

## The Roadblock: Trapped in the Windows Boot Loop
After completing the installation, removing the installation medium, and restarting, the laptop **booted straight back into the Windows login screen** instead of loading Ubuntu. 

The system appeared completely unchanged, and the internal drive remained locked.

### The Breakthrough: Switching from RAID to AHCI
To solve this, it was discovered that the laptop's storage controller architecture was blocking Linux from actually applying changes to the drive. Dell laptops default to **RAID On** (Intel Rapid Storage Technology) out of the box. While Windows can read this natively, the Linux kernel cannot interact with the NVMe SSD under a RAID configuration, rendering the previous installation attempt ineffective.

The storage controller had to be manually reconfigured:

1. Powered down the machine and tapped **`F2`** repeatedly during startup to enter the **Dell BIOS/System Setup**.
2. Navigated to the **Storage** configuration tab.
3. Located **SATA/Storage Operation** (which was set to *RAID On*).
4. Switched the selection to **AHCI/NVMe** mode.
5. Accepted the system firmware warning stating that switching modes might prevent the existing OS from booting (which was the goal).
6. Clicked **Apply Changes** and **Exit**.

---

## The Final Successful Installation
With the storage controller successfully switched to AHCI mode, the Linux kernel was finally able to natively detect, partition, and interact with the physical NVMe SSD hardware.

1. Booted back into the Ubuntu USB installer using the **`F12`** One-Time Boot Menu.
2. Ran through the installation wizard a second time using the exact same configuration parameters.
3. Selected **"Erase disk and install Ubuntu"**. 

> **Result:** With AHCI enabled, the installer successfully wiped out the old file allocation tables, completely destroyed the encrypted BitLocker Windows partitions, and reformatted the internal drive to a clean **EXT4** file system.

---

## Post-Installation Log & Verification
Once the installer bar completed, the system unmounted the media files and prompted to remove the installation medium and press `Enter`. 

The laptop executed a pristine reboot cycle directly into the native GRUB bootloader and the Ubuntu GNOME display manager.

### Post-Install Baseline Configuration
Immediately upon launching the terminal environment (`Ctrl + Alt + T`), the local package repositories were refreshed and updated to establish system stability and verify performance on the repurposed hardware:

```bash
# Refresh package indexes and upgrade native system binaries
sudo apt update && sudo apt upgrade -y

# Verify running kernel and system architecture
uname -a
