# System‑Drive Isolation for WSL: Professional Setup Guide

<details>
<summary>📑 Table of Contents</summary>

1. [Introduction](#introduction)  
2. [Pros & Cons Analysis](#pros--cons-analysis)  
3. [Prerequisites](#prerequisites)  
4. [Step‑by‑Step Setup Guide](#step-by-step-setup-guide)  
   1. [Folder Preparation](#folder-preparation)  
   2. [Enable Windows Features](#enable-windows-features)  
   3. [Install WSL & Ubuntu](#install-wsl--ubuntu)  
   4. [Export, Unregister & Import Distro](#export-unregister--import-distro)  
5. [Verification Steps](#verification-steps)  
6. [Advanced Configuration](#advanced-configuration)  
   1. [User Management](#user-management)  
   2. [Drive Mount Control](#drive-mount-control)  
   3. [Shell Prompt & Startup](#shell-prompt--startup)  
7. [GitHub Repository Implementation Guide](#github-repository-implementation-guide)  
8. [GitHub Pages Deployment](#github-pages-deployment)  
9. [Markdown & GitHub‑Ready Formatting](#markdown--github-ready-formatting)  
10. [Conclusion](#conclusion)  

</details>

---

## Introduction
This guide shows you how to install and configure **WSL 2** so that **all** Linux data resides on a separate drive (e.g. `E:\appData\WSL`) while your **C:** drive remains read‑only and uncluttered. You will also learn best practices for organizing a GitHub repository and publishing via GitHub Pages, complete with a professional Markdown template.

---

## Pros & Cons Analysis

### Advantages
- **System protection:** Isolating distro files on **E:** prevents accidental writes to **C:**, safeguarding Windows system files.  
- **Simplified backups:** Snapshot or copy `E:\appData\WSL` directly without digging through user profiles.  
- **Resource isolation:** Large data sets no longer bloat your system drive.  
- **Near‑native performance:** ext4 VHDX on a dedicated drive approaches native I/O speeds under WSL 2.  
- **Clean separation:** The ext4.vhdx is by default inaccessible to Windows tools, reducing cross‑contamination risk.

### Disadvantages
- **Setup complexity:** Multiple PowerShell and Linux commands increase the learning curve.  
- **Maintenance overhead:** Major WSL or distro updates may require re‑export/import or config tweaks.  
- **I/O caveats:** DrvFs mounts on non‑system drives can be slower under heavy load.  
- **Limited Windows scanning:** Antivirus or Windows tooling won’t inspect ext4.vhdx.  
- **Portability:** Custom drive mappings may not transfer seamlessly to other machines.

---

## Prerequisites
- **Windows Version:** Windows 10 1903+ or Windows 11  
- **Admin Access:** PowerShell **as Administrator**  
- **Drive Space:** ≥ 10 GB free on **E:** (or your chosen drive)  
- **Internet:** Required for kernel and distro downloads

---

## Step‑by‑Step Setup Guide

### Folder Preparation
Create the required directories on **E:**:
```powershell
New-Item -ItemType Directory `
  -Path "E:\appData\WSL\Ubuntu\Rootfs" `
  -Force
```

### Enable Windows Features
Enable WSL and the Virtual Machine Platform:
```powershell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux `
  /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform `
  /all /norestart
Restart-Computer -Force
```

### Install WSL & Ubuntu
Install the latest WSL components and Ubuntu:
```powershell
wsl --install
```

### Export, Unregister & Import Distro
Move your Ubuntu distro entirely to **E:**:
```powershell
wsl --shutdown
wsl --export Ubuntu E:\appData\WSL\Ubuntu.tar
wsl --unregister Ubuntu
wsl --import Ubuntu "E:\appData\WSL\Ubuntu\Rootfs" "E:\appData\WSL\Ubuntu.tar" --version 2
Remove-Item "E:\appData\WSL\Ubuntu\Ubuntu.tar" -Force
```

---

## Verification Steps
1. List installed distros and versions:
   ```powershell
   wsl -l -v
   ```
2. Confirm the BasePath registry entry:
   ```powershell
   (Get-ChildItem HKCU:\Software\Microsoft\Windows\CurrentVersion\Lxss |
     ForEach-Object { Get-ItemProperty $_.PSPath }) |
     Select DistributionName, BasePath
   ```
3. Verify **C:** is mounted read‑only:
   ```bash
   mount | grep ' on /mnt/c '
   ```

---

## Advanced Configuration

### User Management
Create a non‑root user with sudo privileges:
```bash
wsl -d Ubuntu -u root
adduser devuser
usermod -aG sudo devuser
```

### Drive Mount Control
Edit **`/etc/wsl.conf`** to disable automount and process `/etc/fstab`:
```ini
[boot]
systemd = true

[automount]
enabled    = false
mountFsTab = true

[interop]
enabled           = false
appendWindowsPath = false

[user]
default = devuser
```
Define a read‑only mount for **C:** in **`/etc/fstab`**:
```
C:  /mnt/c  drvfs  ro,defaults  0 0
```

### Shell Prompt & Startup
Ensure each shell session starts in the user’s home and uses a clean prompt. Add to `~/.bashrc` and `/root/.bashrc`:
```bash
cd ~
```

---
