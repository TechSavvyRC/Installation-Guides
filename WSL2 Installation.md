# Table of Contents

1.  [Introduction](#1-introduction)
2.  [Pros & Cons Analysis](#2-pros--cons-analysis)
3.  [Prerequisites](#3-prerequisites)
4.  [Installation Guide](#4-installation-guide)
    1.  [Prepare Custom Folders](#41-prepare-custom-folders)
    2.  [Enable Windows Features](#42-enable-windows-features)
    3.  [Install WSL & Default Distro (Ubuntu)](#4-3-install-wsl--default-distro-ubuntu)
    4.  [Export the Installed Distro](#44-export-the-installed-distro)
    5.  [Unregister the Original Distro](#45-unregister-the-original-distro)
    6.  [Import to Custom Location on E](#46-import-to-custom-location-on-e)
5.  [Verification Steps](#5-verification-steps)
    1.  [List installed distros and versions](#51-list-installed-distros-and-versions)
    2.  [Confirm the BasePath registry entry](#52-confirm-the-basepath-registry-entry)
    3.  [Virtual Hard Disk File](#53-virtual-hard-disk-file)
6.  [Post‑Installation Configuration](#6-post‑installation-configuration)
    1.  [User Management](#61-user-management)
    2.  [Shell Prompt & Startup](#62-shell-prompt-startup)
    3.  [Drive Mount Control](#63-drive-mount-control)
    4.  [Mount C Drive Read-only](#64-mount-c-drive-read-only)
7.  [Final Verification](#7-final-verification)
    1.  [Shut Down WSL](#71-shut-down-wsl)
    2.  [Launch WSL](#72-launch-wsl)
    3.  [Verify Home Directory](#73-verify-home-directory)
    4.  [Test Read-Only Mount](#74-test-read-only-mount)
<p>&nbsp;</p>

---
## 1. Introduction

Windows Subsystem for Linux (WSL) enables developers and system administrators to run a full Linux environment directly within Windows — without the need for dual-booting or using a VM. WSL 2, the latest version, utilizes a lightweight virtual machine with a real Linux kernel, offering significant performance and compatibility improvements over WSL 1.

By default, WSL stores Linux distributions and their associated data on the system drive (typically `C:`). While this is convenient for quick installations, it can lead to several problems in professional or performance-sensitive environments:

* **System clutter:** The `C:` drive becomes populated with large, growing files (`ext4.vhdx`) and configuration data.
* **Security risks:** Mixing critical OS files with mutable Linux root files can expose the system to unintended modification or corruption.
* **Backup complexity:** Backing up Linux environments becomes more complex when they're nested under system user directories.

This guide walks you through setting up WSL 2 in a way that **`isolates the Linux root filesystem on a separate drive`**, such as `E:\`. This method enables better performance, security, maintainability, and portability for your development environment. All Linux-related files — including the ext4 virtual hard disk, root filesystem, and configuration artifacts — are stored in a dedicated path outside of your Windows system volume, keeping things clean, secure, and easy to manage.

You’ll also benefit from:

* Full control over where Linux files are stored
* Easier snapshot, backup, and replication of entire environments
* Clean separation of concerns between your Linux environment and Windows

By the end of this guide, you will have:

* Installed WSL 2 with Ubuntu (or a distro of your choice)
* Exported and re-imported your Linux distro to a location like `E:\appData\WSL\Ubuntu`
* Configured a well-organized directory structure for long-term use and system protection

This setup is ideal for power users, developers, or anyone who wants a robust and professional-grade WSL environment.
<p>&nbsp;</p>

---
## 2. Pros & Cons Analysis

### 2.1 Advantages
* **System protection:** Isolating distro files on **`E:`** prevents accidental writes to **`C:`**, safeguarding Windows system files.
* **Simplified backups:** Snapshot or copy `E:\appData\WSL` directly without digging through user profiles.
* **Resource isolation:** Large data sets no longer bloat your system drive.
* **Near‑native performance:** ext4 VHDX on a dedicated drive approaches native I/O speeds under WSL 2.
* **Clean separation:** The ext4.vhdx is by default inaccessible to Windows tools, reducing cross‑contamination risk.
---
### 2.2 Disadvantages
* **Setup complexity:** Multiple PowerShell and Linux commands increase the learning curve.
* **Maintenance overhead:** Major WSL or distro updates may require re‑export/import or config tweaks.
* **I/O caveats:** DrvFs mounts on non‑system drives can be slower under heavy load.
* **Limited Windows scanning:** Antivirus or Windows tooling won’t inspect ext4.vhdx.
* **Portability:** Custom drive mappings may not transfer seamlessly to other machines.
<p>&nbsp;</p>

---
## 3. Prerequisites
* **Windows Version:** Windows 10 1903+ or Windows 11
* **Admin Access:** PowerShell **`as Administrator`**
* **Drive Space:** ≥ 10 GB free on **`E:`** (or your chosen drive)
* **Internet:** Required for kernel and distro downloads
<p>&nbsp;</p>

---

> ⚠️ **Important:** All commands in this guide must be executed in **`PowerShell running as Administrator`** ⚠️

---

## 4. Installation Guide

### 4.1 Prepare Custom Folders
Open PowerShell as an administrator and execute the following command to create the necessary directory structure on your desired non-system drive (replace **`E:`** with your preferred drive letter if needed).
```powershell
New-Item -ItemType Directory -Path "E:\appData\WSL\Ubuntu\Rootfs" -Force
```
* **Purpose of the command:** These commands create a nested directory structure where the WSL distribution files will reside.
* **Breakdown of parameters and arguments:**
    * **`New-Item:`** PowerShell cmdlet to create new items (like directories).
    * **`-ItemType Directory:`** Specifies that a directory should be created.
    * **`-Path "E:\\appData\\...":`** Defines the full path where the new directory will be created.
    * **`-Force:`** If the directory already exists, it will be overwritten without prompting. Use with caution if you have existing data in these paths.
* **Expected output:** The successful creation of the specified directory structure on the **`E:`** drive.
---
### 4.2 Enable Windows Features
Still in an administrator PowerShell session, run the following commands to enable the WSL and Virtual Machine Platform features.
```powershell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart

# Then reboot your machine.
Restart-Computer -Force
```
* **Purpose of the command:** These commands use the Deployment Image Servicing and Management tool (dism.exe) to enable the necessary Windows features for WSL.
* **Breakdown of parameters and arguments:**
    * **`dism.exe /online:`** Targets the currently running operating system.
    * **`/enable-feature:`** Specifies that a feature should be enabled.
    * **`/featurename:`** Microsoft-Windows-Subsystem-Linux and /featurename:VirtualMachinePlatform: The names of the features to enable.
    * **`/all:`** Enables all parent and dependent features.
    * **`/norestart:`** Prevents an automatic reboot after the command completes.
* **Expected output:** ***`The operation completed successfully`*** for both commands. You will then need to manually reboot your computer.
---
### 4.3 Install WSL & Default Distro(Ubuntu)
After rebooting, open a new PowerShell window and run the following command to install WSL with the default Ubuntu distribution.
```powershell
wsl --install
```
* **Purpose of the command:** This command downloads and installs the WSL core components and the default Linux distribution (currently Ubuntu).
* **Breakdown of parameters and arguments:**
    * **`wsl:`** The command-line tool for managing WSL.
    * **`--install:`** Installs WSL and the default distribution.
* **Expected output:** Output similar to the following:

    ```powershell
    Downloading: Windows Subsystem for Linux 2.4.13
    Installing: Windows Subsystem for Linux 2.4.13
    Windows Subsystem for Linux 2.4.13 has been installed.
    The operation completed successfully.
    Downloading: Ubuntu
    Installing: Ubuntu
    Distribution successfully installed. It can be launched via 'wsl.exe -d Ubuntu'
    ```
---
### 4.4 Export the Installed Distro
Before unregistering the default installation, shut down all running WSL instances and export the Ubuntu distribution to a TAR archive on your **`E:`** drive.
```powershell
wsl --shutdown
wsl --export Ubuntu E:\appData\WSL\Ubuntu.tar
Test-Path "E:\appData\WSL\Ubuntu.tar"
```
* **Purpose of the command:**
    * **`wsl --shutdown:`** Terminates all running WSL distributions.
    * **`wsl --export Ubuntu E:\\appData\\WSL\\Ubuntu.tar:`** Exports the Ubuntu distribution named ***`Ubuntu`*** to a TAR archive at the specified path.
    * **`Test-Path:`** Verifies if the exported TAR file was created successfully.
* **Breakdown of parameters and arguments:**
    * **`--shutdown:`** No specific arguments.
    * **`--export <DistributionName> <ExportFilePath>:`** Specifies the distribution to export and the path for the resulting TAR file.
    * **`Test-Path <Path>:`** Checks if a file or directory exists at the given path.
* **Expected output:** If successful, ***`Test-Path`*** will return **`True`**.
---
### 4.5 Unregister the Original Distro
Now, unregister the default Ubuntu installation. This removes the registration but leaves the exported TAR file intact.
```powershell
wsl --unregister Ubuntu
```
* **Purpose of the command:** Removes the registration of the specified WSL distribution.
* **Breakdown of parameters and arguments:**
    * **`--unregister <DistributionName>:`** Specifies the distribution to unregister.
* **Expected output:** ***`Unregistering.`*** followed by ***`The operation completed successfully.`***
---
### 4.6 Import to Custom Location on E
Import the exported TAR archive to the custom location you created on the E: drive. This command registers a new WSL instance pointing to the specified directory.
```powershell
wsl --import Ubuntu "E:\appData\WSL\Ubuntu\Rootfs" "E:\appData\WSL\Ubuntu.tar" --version 2
Remove-Item "E:\appData\WSL\Ubuntu\Ubuntu.tar" -Force
```
* **Purpose of the command:**
    * **`wsl --import Ubuntu "E:\\appData\\WSL\\Ubuntu\\Rootfs" "E:\\appData\\WSL\\Ubuntu.tar" --version 2:`** Imports the distribution from the TAR file to the specified directory, registering it with the name "Ubuntu" and using WSL version
    * **`Remove-Item -Path "E:\\appData\\WSL\\Ubuntu.tar" -Force:`** Deletes the temporary TAR archive.
* **Breakdown of parameters and arguments:**
    * **`--import <NewDistributionName> <InstallLocation> <TarFilePath>:`** Specifies the name for the new distribution, the directory where the file system will be stored, and the path to the TAR archive.
    * **`--version <1 or 2>:`** Specifies the WSL version to use for this distribution. WSL 2 is highly recommended for performance.
    * **`Remove-Item:`** PowerShell cmdlet to delete files or directories.
    * **`-Path:`** Specifies the path to the item to be removed.
    * **`-Force:`** Deletes the item without prompting.
* **Expected output:** Successful registration of the ***`Ubuntu`*** distribution at the new location.
<p>&nbsp;</p>

---
## 5. Verification Steps
After completing the installation and import process, it's crucial to verify that WSL is correctly set up and functioning as expected in the new location.

### 5.1 List installed distros and versions
To confirm the installation, this command lists all installed WSL distributions and their versions.
```powershell
wsl -l -v
```
* **Purpose of the command:** Displays installed WSL distributions.
* **Breakdown of parameters and arguments:**
    * **`wsl:`** The WSL command-line tool.
    * **`-l:`** Lists distributions.
    * **`-v:`** Displays version information.
* **Expected output:** A list of WSL distributions, including ***`Ubuntu`***, along with its version.
---
### 5.2 Confirm the BasePath registry entry
This command retrieves the installation path from the Windows Registry to ensure it points to the new location.
```powershell
(Get-ChildItem HKCU:\Software\Microsoft\Windows\CurrentVersion\Lxss |
  ForEach-Object { Get-ItemProperty $_.PSPath }) |
  Select-Object DistributionName, BasePath
```
* **Purpose of the command:** Retrieves the installation path of the WSL distribution.
* **Breakdown of parameters and arguments:**
    * **`Get-ChildItem HKCU:\\Software\\Microsoft\\Windows\\CurrentVersion\\Lxss:`** Gets all sub-items (distributions) under the Lxss registry key.
    * **`ForEach-Object { Get-ItemProperty $\_.PSPath }:`** For each distribution, gets its properties, including the path.
    * **`Select-Object DistributionName, BasePath:`** Selects and displays the distribution name and its base path.
* **Expected output:** A table showing the distribution name ***`Ubuntu`*** and its corresponding BasePath.
---
### 5.3 Virtual Hard Disk File
This command checks for the existence of the virtual hard disk file in the new location, confirming that the distribution's files are in the correct place.
```powershell
Test-Path "E:\appData\WSL\Ubuntu\Rootfs\ext4.vhdx"
```
* **Purpose of the command:** Verifies the existence of the virtual hard disk.
* **Breakdown of parameters and arguments:**
    * **`Test-Path:`** PowerShell cmdlet to check if a file or directory exists.
    * **`"E:\\appData\\WSL\\Ubuntu\\Rootfs\\ext4.vhdx":`** The path to the virtual hard disk file.
* **Expected output:`** ***`True***" if the file exists, ***`False`*** otherwise.
<p>&nbsp;</p>

---
## 6. Post‑Installation Configuration
After verifying the installation, we'll proceed to configure the Ubuntu environment for enhanced security, control, and usability.

### 6.1 User Management
To perform initial setup, this command logs into the Ubuntu distribution as the root user. This command logs into the Ubuntu distribution as the root user to allow for initial configuration.
```powershell
wsl -d Ubuntu -u root
adduser devuser
usermod -aG sudo devuser
```
* **Purpose of the command:** Logs into the Ubuntu distribution as the root user. Creates a regular user account with administrative privileges.
* **Breakdown of parameters and arguments:**
    * **`wsl:`** The WSL command-line tool.
    * **`-d Ubuntu:`** Specifies the distribution to log into ***`Ubuntu`***.
    * **`-u root:`** Specifies the user to log in as ***`root`***.
    * **`adduser devuser:`** Creates a new user named ***`devuser`***. This is an interactive command that will prompt for a password and other details.
    * **`usermod -aG sudo devuser:`** Modifies the user devuser to add it to the ***`sudo`*** group.
    * **`usermod:`** Modifies a user account.
    * **`-aG sudo:`** Adds the user to the ***`sudo`*** group.
* **Expected output:** A root user shell within the Ubuntu distribution. A new user account named ***`devuser`*** that can execute commands with sudo.
---
### 6.2 Shell Prompt & Startup
Ensure each shell session starts in the user’s home and uses a clean prompt. These commands modify the ***`.bashrc`*** files to ensure the terminal starts in the user's home directory.
```bash
cd ~
```
* **Purpose of the command:** Configures the terminal to start in the user's home directory.
* **Breakdown of parameters and arguments:**
    * **`echo "cd ~" >> /root/.bashrc:`** Adds the command ***`cd ~`*** to the end of the ***`/root/.bashrc`*** file.
    * **`echo "cd ~" >> /home/devuser/.bashrc:`** Adds the command ***`cd ~`*** to the end of the ***`/home/devuser/.bashrc`*** file.
* **Expected output:** The terminal will start in the home directory for the respective user.
---
### 6.3 Drive Mount Control
This command edits the wsl.conf file to apply specific WSL settings, including enabling systemd, disabling automatic drive mounting, and setting the default user.
```bash
#edit '/etc/wsl.conf' file and add below content in it.
vi /etc/wsl.conf
[boot]
systemd = true

[automount]
enabled    = false    # turn off all automatic Windows-drive mounts
mountFsTab = true     # disable processing of /etc/fstab

[interop]
enabled            = false    # disable launching Win binaries
appendWindowsPath  = false    # don't pollute $PATH with Windows paths

[user]
default = devuser         # set your non-root default user
```
* **Purpose of the command:** Configures WSL settings.
* **Breakdown of parameters and arguments:**
    * **`vi /etc/wsl.conf:`** Opens the wsl.conf file in the vi editor.
    * **The following lines specify various WSL configurations:**
        * **`[boot] systemd = true:`** Enables systemd.
        * **`[automount] enabled = false:`** Disables automatic mounting of Windows drives.
        * **`[automount] mountFsTab = true:`** Enables using ***`/etc/fstab`***.
        * **`[interop] enabled = false:`** Disables launching Windows binaries from WSL.
        * **`[interop] appendWindowsPath = false:`** Prevents Windows paths from being added to the WSL environment's PATH variable.
        * **`[user] default = devuser:`** Sets the default user to ***`devuser`***.
---
### 6.4 Mount C Drive Read-only
To enhance security and prevent accidental modification of the Windows system drive, this command mounts it in read-only mode. If you only want to mount certain drives or mount them read‑only, disable automount globally, then in ***`/etc/fstab`*** add entries as mentioned below.
```bash
vi /etc/fstab
C:  /mnt/c  drvfs  defaults,ro  0  0
```
* **Purpose of the command:** Mounts the ***`C:`*** drive read-only.
* **Breakdown of parameters and arguments:**
    * **`vi /etc/fstab:`** Opens the ***`/etc/fstab`*** file in the ***`vi`*** editor.
    * **`C: /mnt/c drvfs defaults,ro 0 0:`** Entry to mount the ***`C:`*** drive.
        * **`C::`** The drive letter.
        * **`/mnt/c:`** The mount point.
        * **`drvfs:`** The file system type.
        * **`defaults,ro:`** Mount options, including read-only.
* **Expected output:** The ***``C:``*** drive mounted as ***`read-only`*** in WSL.
<p>&nbsp;</p>

---

## 7. Final Verification
Finally, after configuring the Ubuntu environment, we need to restart WSL and verify that all the changes have been applied correctly.

### 7.1 Shut Down WSL
After making the configuration changes, this command shuts down all running WSL instances to ensure the new settings are applied upon restart.
```powershell
#From PowerShell (Admin or normal), run:
wsl --shutdown
```
* **Purpose of the command:** Shuts down all running WSL instances.
* **Breakdown of parameters and arguments:**
    * **`wsl:`** The WSL command-line tool.
    * **`--shutdown:`** Shuts down WSL.
* **Expected output:`** All WSL instances are terminated.
---
### 7.2 Launch WSL
This command launches the Ubuntu distribution, which should now log in as the default user, as configured in ***`/etc/wsl.conf``***.
```powershell
#Launch normally. It must login as `devuser` by default as mentioned under `/etc/wsl.conf` file
wsl -d Ubuntu
```
* **Purpose of the command:** Launches the Ubuntu distribution.
* **Breakdown of parameters and arguments:**
    * **wsl:** The WSL command-line tool.
    * **-d Ubuntu:** Specifies the distribution to launch.
* **Expected output:** The Ubuntu distribution starts, and you are logged in as ***`devuser``***.
---
### 7.3 Verify Home Directory
This command ensures that the terminal starts in the user's home directory
```bash
#Force Start in Home Directory
#Add at the top of both ~/.bashrc (for devuser) and /root/.bashrc:
# always jump to home on shell start
vi /home/devuser/.bashrc
cd ~

#Reload with:
source /home/devuser/.bashrc
```
* **Purpose of the command:** Configures the terminal to start in the user's home directory.
* **Breakdown of parameters and arguments:**
    * **`echo "cd ~" >> /home/devuser/.bashrc:`** Adds the command ***`cd ~`*** to the end of the ***`/home/devuser/.bashrc`*** file.
    * **`source /home/devuser/.bashrc:`** Reloads the ***`/home/devuser/.bashrc`*** file.
* **Expected output:** The terminal will start in the home directory for the respective user.
---
### 7.4 Test Read-Only Mount
Finally, this command attempts to create a file on the ***`C:`*** drive to verify that the read-only mount configuration was successful.
```bash
#Testing Read‑Only Enforcement
touch /mnt/c/Windows/System32/test_wsl.txt
```
* **Purpose of the command:** Tests the read-only mount of the ***`C:`*** drive.
* **Breakdown of parameters and arguments:**
    * **`touch /mnt/c/Windows/System32/test_wsl.txt:`** Attempts to create a file at the specified path.
* **Expected output:** An error message indicating a ***`Read-only file system``***.

---
