# Dell-Latitude-Ubuntu-Migration
Wiping drive to install Ubuntu
# Dell Latitude 7420 — Windows Lockout Bypass & Ubuntu Installation

A comprehensive documentation of a successful operating system recovery on a Dell Latitude 7420 laptop. The machine was initially bricked by a Windows BitLocker encryption lockout and successfully recovered by performing a clean installation of Ubuntu Linux.

---

## Hardware Environment
* **Device:** Dell Latitude 7420 Laptop
* **Storage:** Internal NVMe SSD (Originally formatted with NTFS, Windows 11, and BitLocker Encryption)
* **Target OS:** Ubuntu Linux 24.04 / 26.04 LTS
* **Installation Media:** 8GB+ USB Flash Drive

---

## The Problem: Windows BitLocker Loop
The laptop was completely inaccessible due to a Windows BitLocker recovery screen demanding a 48-digit encryption key (`Recovery Key ID: 2DF428A4...`). 
* Attempting to reset the PC via the native Windows Recovery Environment (`Shift + Restart`) failed because the built-in Windows repair tools refused to modify or format the encrypted partition without the decryption key.
* Choosing **"Skip this drive"** repeatedly looped back to the same lock screen.

---

## The Solution: Bypassing Encryption via Live Linux USB
Because Windows BitLocker only protects data within the Windows ecosystem, booting the machine into an independent Linux live environment completely bypassed the software lock, allowing for a total storage drive wipe.

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

### Step 3: Troubleshooting the Hardware Interrupts (SupportAssist)
During the initial boot sequence, the Dell motherboard threw a firmware interrupt: 
> `WARNING: Battery is critically low.`

* **Root Cause:** The intensive processing required to configure the live environment drained the laptop battery below safe POST thresholds. 
* **Fix:** Left the AC charger connected, allowed the battery to gain a minimal safe charge, and selected **Continue** to pass the hardware diagnostic barrier.

### Step 4: The Clean Installation Wizard
Once inside the Ubuntu live environment, the installation wizard was configured with specific settings to ensure stability and stability across hardware components:

1. **Updates & Networks:** Selected **Normal Installation**. 
2. **Stability Tweak:** **Unchecked** *"Download updates while installing Ubuntu"*. This isolated the installation to local USB files and prevented the wizard from hanging or freezing due to background network drops.
3. **Hardware Compatibility:** **Checked** *"Install third-party software for graphics and Wi-Fi hardware"* to guarantee standard proprietary network card drivers were native upon first boot.
4. **Disk Partitioning:** Selected **"Erase disk and install Ubuntu"**. 

>  **Result:** This single action completely deleted the old file allocation tables, obliterated the encrypted BitLocker Windows partitions, and reformatted the internal NVMe storage drive to a clean **EXT4** file system.

5. **Finalizing User Space:** Configured the localized time zone, set up the primary user profile (`claudia`), and designated the machine network hostname (`claudia-laptop`).

---

## Post-Installation Log & Verification
Once the installer bar completed, the system unmounted the media files and prompted: 
> `Please remove the installation medium, then press ENTER:`

The USB drive was extracted, `Enter` was cycled, and the laptop executed a pristine reboot cycle directly into the native GRUB bootloader and the Ubuntu GNOME display manager.

### Post-Install Baseline Configuration
Immediately upon launching the terminal environment (`Ctrl + Alt + T`), the local package repositories were refreshed and updated to establish system stability:

```bash
# Refresh package indexes and upgrade native system binaries
sudo apt update && sudo apt upgrade -y

# Verify running kernel and system architecture
uname -a
