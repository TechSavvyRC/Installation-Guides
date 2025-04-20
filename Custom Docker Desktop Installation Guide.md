# Table of Contents

1.  [Introduction](#1-introduction)
2.  [Pros & Cons Analysis](#2-pros--cons-analysis)
3.  [Prerequisites](#3-prerequisites)
4.  [Installation Guide](#4-installation-guide)
    1.  [Prepare Custom Folders](#41-prepare-custom-folders)
    2.  [Define Download URL](#42-define-download-url)
    3.  [Prepare Your Downloads Folder](#43-prepare-your-downloads-folder)
    4.  [Download the Installer into Downloads](44-download-the-installer-into-downloads)
    5.  [Build Argument List for Docker Desktop Installer](45-build-argument-list-for-docker-desktop-installer)
    6.  [Execute Installer from Downloads](#46-execute-installer-from-downloads)
	7.  [Update System Path](#47-update-system-path)
	8.  [Restart Machine](#48-restart-machine)
5.  [Verification Steps](#5-verification-steps)
    1.  [Check Docker Version](#51-check-docker-version)
    2.  [Check Disk Image Location](#52-check-disk-image-location)
6.  [Exceptions](#6-exceptions)
    1. [`C:\Program Files\Docker\cli-plugins`](#61-cprogram-filesdockercli-plugins)
    2. [`C:\ProgramData\DockerDesktop`](#62-cprogramdatadockerdesktop)
    3. [`C:\Users\[username]\.docker`](#63-cusersusername.docker)
    4. [`C:\Users\[username]\AppData\Local\Docker`](#64-cusersusernameappdatalocaldocker)
7.  [Optional Configuration](#7-optional-configuration)
    1.  [Remove the Installer to Save Space](#71-remove-the-installer-to-save-space)
    2.  [Ensure Docker Service Starts on Boot](#72-ensure-docker-service-starts-on-boot)
<p>&nbsp;</p>

---
## 1. Introduction

Custom Docker Desktop installation involves specifying non-default locations for Docker Desktop's program files and data. This approach is beneficial for users who need to manage disk space across multiple drives, improve performance by placing data on faster storage, or adhere to specific organizational policies regarding software installation paths. By customizing the installation, users gain greater control over where Docker Desktop components reside on their system. This guide provides a comprehensive walkthrough for performing a custom installation of Docker Desktop on Windows. It addresses common challenges associated with default installations, such as limited space on the primary OS drive.
<p>&nbsp;</p>

---
## 2. Pros & Cons Analysis

### 2.1 Advantages
* **Disk Space Management:** Allows users to install Docker Desktop and store its data on drives with more available space, preventing the primary OS drive from filling up.
* **Performance Optimization:** Placing Docker's virtual machine images and data on faster drives (e.g., SSDs) can lead to improved I/O performance for containers.
* **Organizational Compliance:** Enables adherence to specific IT policies that dictate where applications and their data should be stored within the system.
* **Simplified Backups:** Separating Docker data onto a dedicated drive can streamline backup processes for container-related files.
* **Clear Separation:** Keeps Docker-related files and data isolated from the primary operating system files, potentially aiding in system maintenance and troubleshooting.
---
### 2.2 Disadvantages
* **Complexity:** Custom installation is more involved than the default installation process, requiring careful execution of commands and understanding of the implications.
* **Potential Compatibility Issues:** Although rare, specifying non-standard paths could potentially lead to unforeseen compatibility issues with future Docker Desktop updates.
* **Increased Maintenance:** Users are responsible for ensuring the custom paths remain accessible and have adequate permissions after system changes or updates.
* **Troubleshooting Overhead:** Diagnosing issues related to custom paths might require more in-depth knowledge of Docker Desktop's architecture.
* **Limited Official Support for Edge Cases:** While custom installations are supported, troubleshooting highly specific issues related to unique custom configurations might have limited official support resources.
<p>&nbsp;</p>

---
## 3. Prerequisites

### 3.1 System Requirements:**
* **Windows 10 or 11 64-bit:** Ensure your operating system meets the minimum version requirements for Docker Desktop. *Reason: Docker Desktop relies on specific Windows features available in these versions.*
* **Virtualization Enabled:** Hyper-V or WSL 2 feature must be enabled in your BIOS and Windows Features. *Reason: Docker Desktop utilizes virtualization technologies for running containers.*
* **Sufficient Disk Space:** Ensure you have enough free space on the target drives for both the application installation and the container data. *Reason: Lack of disk space can lead to installation failures or performance issues.*
* **Minimum RAM:** Docker Desktop recommends a minimum amount of RAM (typically 4GB, but 8GB or more is recommended). *Reason: Running containers can be memory-intensive.*
---
### 3.2 Administrative Permissions:**
* **Administrator Access:** You need administrator privileges on your Windows account to install and configure Docker Desktop. *Reason: The installation process requires system-level changes.*
---
### 3.3 Pre-installation Preparations:**
* **Backup Important Data:** It's always recommended to back up any critical data before making significant system changes. *Reason: Although unlikely, installation issues could potentially lead to data loss.*
* **Uninstall Previous Docker Versions:** If you have older versions of Docker Desktop or Docker Engine installed, uninstall them cleanly. *Reason: Conflicts between different versions can cause installation problems.*
---
### 3.4 Software Dependencies:**
* **WSL 2 (Recommended):** For optimal performance and resource utilization, it's highly recommended to have the Windows Subsystem for Linux 2 installed. *Reason: WSL 2 provides a lightweight and efficient environment for running Linux-based containers.*
* **Hyper-V (Alternative):** If WSL 2 is not an option, ensure Hyper-V is enabled. *Reason: Hyper-V is Microsoft's native hypervisor that Docker Desktop can utilize.*
---
### 3.5 Knowledge Requirements:**
* **Basic Command Line Knowledge:** Familiarity with PowerShell or the Command Prompt will be helpful for executing the installation commands. *Reason: The custom installation process primarily involves command-line operations.*
* **Understanding of File Paths:** You need to understand how file paths work in Windows to specify the custom installation locations correctly. *Reason: Incorrect paths will lead to installation failures or data being stored in unintended locations.*
<p>&nbsp;</p>

---

> ⚠️ **Important:** All commands in this guide must be executed in **`PowerShell running as Administrator`** ⚠️

---

## 4. Installation Guide

### 4.1 Prepare Custom Folders
First, create the directories where you want to install Docker Desktop and store its data. This example uses `C:\DevOps-Tools\Docker` for the application and `E:\appData\Docker\` for various data locations. Adjust these paths according to your needs.
```powershell
New-Item -ItemType Directory -Path "C:\DevOps-Tools\Docker" -Force
New-Item -ItemType Directory -Path "E:\appData\Docker\wsl" -Force
New-Item -ItemType Directory -Path "E:\appData\Docker\hyperv" -Force
New-Item -ItemType Directory -Path "E:\appData\Docker\windows" -Force
```
* **Purpose:** These commands create the necessary folders for the Docker Desktop application and its associated data on your chosen drives.
* **Parameter/Argument Breakdown:**
    * `New-Item`: PowerShell cmdlet to create new items.
    * `-ItemType Directory`: Specifies that you want to create a directory (folder).
    * `-Path "..."`: Defines the full path where the new directory will be created.
    * `-Force`: Creates the directory even if parent directories do not exist.
* **Expected Output:** Four new directories will be created at the specified paths if they don't already exist.
---
### 4.2 Define Download URL
Next, define a variable to store the download URL for the latest Docker Desktop installer.
```powershell
$installerUrl = '[https://desktop.docker.com/win/main/amd64/Docker%20Desktop%20Installer.exe](https://desktop.docker.com/win/main/amd64/Docker%20Desktop%20Installer.exe)'
```
* **Purpose:** This line defines a variable containing the official download link for the Docker Desktop installer, ensuring you get the latest version.
* **Parameter/Argument Breakdown:**
    * `$installerUrl`: A variable name where the string value of the URL will be stored.
    * `'...'`: Encloses the string value of the Docker Desktop installer URL.
* **Expected Output:** The variable `$installerUrl` will hold the URL of the Docker Desktop installer.
---
### 4.3 Prepare Your Downloads Folder
Ensure a local directory exists to temporarily store the installer.
```powershell
$downloadDir   = 'C:\Users\zanyb\Downloads'
New-Item -Path $downloadDir -ItemType Directory -Force
```
* **Purpose:** This sets a variable for your Downloads directory and ensures the directory exists.
* **Parameter/Argument Breakdown:**
    * `$downloadDir`: A variable holding the path to your Downloads folder.
    * `New-Item -Path ... -ItemType Directory -Force`: Creates the Downloads directory if it doesn't exist.
* **Expected Output:** The `$downloadDir` variable is set, and the Downloads directory exists.
---
### 4.4 Download the Installer into Downloads
Download the Docker Desktop installer using the URL defined earlier.
```powershell
$installerPath = Join-Path $downloadDir 'DockerDesktopInstaller.exe'
Invoke-WebRequest `
    -Uri $installerUrl `
    -OutFile $installerPath `
    -UseBasicParsing
```
* **Purpose:** This command downloads the Docker Desktop installer from the official Docker website and saves it to your Downloads folder.
* **Parameter/Argument Breakdown:**
    * `$installerPath`: A variable combining the download directory and the installer filename.
    * `Invoke-WebRequest`: PowerShell cmdlet to send HTTP and HTTPS requests.
    * `-Uri $installerUrl`: Specifies the URL to download from.
    * `-OutFile $installerPath`: Defines the local path where the downloaded file will be saved.
    * `-UseBasicParsing`: Forces `Invoke-WebRequest` to use the basic parser for the response content, which can be useful in certain network environments.
* **Expected Output:** The `DockerDesktopInstaller.exe` file will be downloaded and saved in your Downloads folder.
---
### 4.5 Build Argument List for Docker Desktop Installer
Construct the arguments to be passed to the Docker Desktop installer for a custom installation.
```powershell
$installArgs = @(
    'install',
    '--quiet',
    '--accept-license',
    '--installation-dir=C:\DevOps-Tools\Docker',
    '--wsl-default-data-root=E:\appData\Docker\wsl',
    '--hyper-v-default-data-root=E:\appData\Docker\hyperv',
    '--windows-containers-default-data-root=E:\appData\Docker\windows',
    '--always-run-service'
)
```
* **Purpose:** This creates an array of arguments that will instruct the Docker Desktop installer to perform a silent, custom installation with specified paths.
* **Parameter/Argument Breakdown:**
    * `$installArgs`: A variable holding an array (`@()`) of strings, where each string is an argument for the installer.
    * `'install'`: The main command to initiate the installation.
    * `'--quiet'`: Runs the installation without displaying a graphical user interface.
    * `'--accept-license'`: Automatically accepts the Docker Desktop license agreement.
    * `'--installation-dir=...'`: Specifies the custom installation directory for the Docker Desktop application files.
    * `'--wsl-default-data-root=...'`: Sets the custom location for the virtual hard disk used by the WSL 2 integration.
    * `'--hyper-v-default-data-root=...'`: Sets the custom location for the virtual hard disk used by the Hyper-V integration (if enabled).
    * `'--windows-containers-default-data-root=...'`: Sets the custom location for the data associated with Windows containers (if enabled).
    * `'--always-run-service'`: Ensures the Docker service starts automatically after installation.
* **Expected Output:** The `$installArgs` variable will contain an array of installation parameters.
---
### 4.6 Execute Installer from Downloads
Run the downloaded installer with the custom arguments.
```powershell
Start-Process `
    -FilePath $installerPath `
    -ArgumentList $installArgs `
    -Wait
```
* **Purpose:** This command executes the Docker Desktop installer with the arguments defined in the `$installArgs` variable, performing the custom installation.
* **Parameter/Argument Breakdown:**
    * `Start-Process`: PowerShell cmdlet to start a new process.
    * `-FilePath $installerPath`: Specifies the path to the executable file to run (the Docker Desktop installer).
    * `-ArgumentList $installArgs`: Provides the array of arguments to be passed to the installer.
    * `-Wait`: Makes the PowerShell script wait until the installer process finishes before proceeding.
* **Expected Output:** Docker Desktop will be installed in the `C:\DevOps-Tools\Docker` directory, and its data will be stored in the `E:\appData\Docker\` subdirectories. The installation will proceed silently without a GUI.
---
### 4.7 Update System Path
Add the custom binary directory to your system's PATH environment variable so you can run `docker` and `docker-compose` commands from any command prompt.
```powershell
setx PATH "$($Env:PATH);C:\DevOps-Tools\Docker" /M
```
* **Purpose:** This command modifies the system-wide PATH environment variable to include the directory where the Docker executables (`docker.exe`, `docker-compose.exe`, etc.) are installed. This allows you to run these commands without specifying their full path.
* **Parameter/Argument Breakdown:**
    * `setx`: Command-line utility to set environment variables.
    * `PATH`: The name of the environment variable being modified.
    * `"$($Env:PATH);C:\DevOps-Tools\Docker"`: The new value for the PATH variable. It appends the custom Docker installation directory to the existing PATH. `$Env:PATH` retrieves the current value of the PATH variable.
    * `/M`: Specifies that the environment variable should be set in the system environment (requires administrator privileges).
* **Expected Output:** The system PATH environment variable will be updated to include `C:\DevOps-Tools\Docker`. You might need to restart your command prompt or PowerShell session for the changes to take effect.

Verify the Docker CLI is accessible from the custom installation location:
```powershell
$where = (Get-Command docker).Source
Write-Host "Docker CLI installed at: $where"
```

* **Purpose:** This command checks the location from where the `docker` command is being executed, confirming it's pointing to your custom installation directory.
* **Parameter/Argument Breakdown:**
    * `(Get-Command docker).Source`: Retrieves the full path of the `docker` executable.
    * `$where`: A variable storing the path of the `docker` executable.
    * `Write-Host "..."`: Displays the value of the `$where` variable in the PowerShell console.
* **Expected Output:** The output should display `Docker CLI installed at: C:\DevOps-Tools\Docker\docker.exe` (or your chosen custom installation path).
---
### 4.8 Restart Machine
A system restart is often required for all Docker Desktop components and environment variable changes to take effect properly.
```powershell
Restart-Computer -Force
```
* **Purpose:** This command initiates a forced restart of your computer.
* **Parameter/Argument Breakdown:**
    * `Restart-Computer`: PowerShell cmdlet to restart the local computer.
    * `-Force`: Restarts the computer without prompting the user to save unsaved work. Use with caution.
* **Expected Output:** Your computer will restart.
<p>&nbsp;</p>

---
## 5. Verification Steps
After restarting your machine, follow these steps to verify that Docker Desktop has been installed correctly with the custom paths.

### 5.1 Check Docker Version
Open a new PowerShell or Command Prompt window and run the following command:
```powershell
docker --version
```
* **Purpose:** This command checks if the `docker` command-line interface (CLI) is installed and accessible, and it displays the installed Docker version.
* **Parameter/Argument Breakdown:**
    * `docker`: The Docker CLI executable.
    * `--version`: An argument that instructs the Docker CLI to output its version information.
* **Expected Output:** You should see output similar to `Docker version <version>, build <commit>`.
---
### 5.2 Check Disk Image Location
Verify that the WSL 2 disk image (if you chose WSL 2 integration) is located in your custom data root directory.
```powershell
Get-ChildItem -Path "E:\appData\Docker\wsl"
```
* **Purpose:** This command lists the files and folders within your custom WSL 2 data root directory.
* **Parameter/Argument Breakdown:**
    * `Get-ChildItem`: PowerShell cmdlet to get the items (files and folders) in a specified location.
    * `-Path "E:\appData\Docker\wsl"`: Specifies the path to your custom WSL 2 data root directory.
* **Expected Output:** You should see files related to the WSL 2 Docker distribution, such as a `.vhdx` file (virtual hard disk). The exact filename might vary.
<p>&nbsp;</p>

---
## 6. Exceptions

### 6.1 `C:\Program Files\Docker\cli-plugins`
* **Purpose:** This directory stores Docker CLI plugins, which extend the functionality of the `docker` command. These plugins are often installed separately and integrate with the core Docker CLI.
* **Why Specific Path:** This is a standard location where Docker expects to find CLI plugins to automatically discover and load them. Deviating from this path would require manual configuration and might break plugin functionality.
* **Typical Files:** Executable files (`.exe` on Windows) and potentially supporting libraries for various Docker plugins (e.g., for cloud integrations, build tools).
* **Data Accumulation:** Typically small, growing only when new CLI plugins are installed.
* **Modification/Relocation:** It is generally **not recommended** to manually modify or relocate this directory, as it can lead to Docker CLI malfunctions.
---
### 6.2 `C:\ProgramData\DockerDesktop`
* **Purpose:** This directory holds application-wide data and configuration settings for Docker Desktop itself. This includes settings related to the Docker Engine, networking, and other core functionalities.
* **Why Specific Path:** `C:\ProgramData` is a standard Windows location for application data that is shared across all users on the system. Docker Desktop relies on this consistent location for its global configuration.
* **Typical Files:** Configuration files (often in `.json` or `.ini` format), logs, and potentially some internal state databases.
* **Data Accumulation:** Can grow over time depending on Docker Desktop usage and logging levels.
* **Modification/Relocation:** **Modifying or relocating this directory is strongly discouraged** as it can lead to Docker Desktop instability or failure to start.
---
### 6.3 `C:\Users\[username]\.docker`
* **Purpose:** This user-specific directory stores configuration and credentials related to the Docker CLI for the individual user. This includes authentication tokens for Docker Hub and other registries, CLI configuration files, and potentially build cache metadata.
* **Why Specific Path:** The `\.docker` directory (or `~/.docker` on Linux/macOS) is a convention for storing user-specific application configuration files. Docker CLI tools look for configuration in this standard location.
* **Typical Files:** `config.json` (for registry authentication), `cli.json` (for CLI settings), and potentially subdirectories for build cache or other user-specific data.
* **Data Accumulation:** Can vary depending on how frequently you interact with Docker registries and build images. Authentication tokens are relatively small, but build cache can grow.
* **Modification/Relocation:** You can modify configuration files within this directory to customize your Docker CLI behavior. However, relocating the entire `\.docker` directory is generally **not recommended** and might require complex symlinking or environment variable adjustments, which could lead to issues.
---
### 6.4 `C:\Users\[username]\AppData\Local\Docker`
* **Purpose:** This directory stores local application data for Docker Desktop, which is specific to the current user and the local machine. This might include temporary files, downloaded resources, and application-specific caches.
* **Why Specific Path:** **AppData\Local** is a standard Windows location for non-roaming, user-specific application data. Data stored here is not typically synchronized across different machines if you use roaming profiles.
* **Typical Files:** Logs, temporary installation files, downloaded components, and potentially application-specific databases or caches.
* **Data Accumulation:** Can vary depending on Docker Desktop usage and the frequency of updates or installations. Logs can grow over time.
* **Modification/Relocation:** While you might be able to delete temporary files within this directory, modifying or relocating the entire `C:\Users\[username]\AppData\Local\Docker` directory is **not recommended** and could lead to application errors.
---
**When to Use Default Installation Instead:**
Consider using the default installation if:
- You are new to Docker and prefer a simpler setup process.
- You have ample space on your primary OS drive.
- You do not have specific organizational policies requiring custom installation paths.
- You prefer to rely on the standard, well-tested configuration.
- You anticipate less need for advanced customization or disk space management for Docker's data.
<p>&nbsp;</p>

---
## 7. Optional Configuration

### 7.1 Remove the Installer to Save Space
Once the installation is complete, you can remove the downloaded installer file to free up disk space in your Downloads folder.
```powershell
Remove-Item -Path $installerPath -Force
```
* **Purpose:** This command deletes the Docker Desktop installer file that was downloaded earlier.
* **Parameter/Argument Breakdown:**
    * `Remove-Item`: PowerShell `cmdlet` to delete files and directories.
    * `-Path $installerPath`: Specifies the path to the installer file (which is stored in the $installerPath variable).
    * `-Force`: Deletes the file without prompting for confirmation. Use with caution.
* **Expected Output:** The `Docker Desktop Installer.exe` file will be deleted from your Downloads folder.
---
### 7.2 Ensure Docker Service Starts on Boot
While the --always-run-service argument was used during installation, you can explicitly verify or set the Docker service to start automatically when your system boots.
```powershell
Set-Service -Name com.docker.service -StartupType Automatic
```
* **Purpose:** This command configures the Docker Desktop service to start automatically whenever you start your computer.
* **Parameter/Argument Breakdown:**
    * `Set-Service`: PowerShell `cmdlet` to manage Windows services.
    * `-Name com.docker.service`: Specifies the name of the Docker Desktop service.
    * `-StartupType Automatic`: Sets the startup type of the service to `Automatic`, meaning it will start during system boot.
* **Expected Output:** The startup type of the com.docker.service will be set to Automatic. You can verify this using the Services management console (services.msc).

---
