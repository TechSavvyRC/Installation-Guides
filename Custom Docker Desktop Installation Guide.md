# Custom Docker Desktop Installation Guide

## 1. Introduction

[[Custom Docker Desktop installation involves specifying non-default locations for Docker Desktop's program files and data. This approach is beneficial for users who need to manage disk space across multiple drives, improve performance by placing data on faster storage, or adhere to specific organizational policies regarding software installation paths. By customizing the installation, users gain greater control over where Docker Desktop components reside on their system. This guide provides a comprehensive walkthrough for performing a custom installation of Docker Desktop on Windows. It addresses common challenges associated with default installations, such as limited space on the primary OS drive.]]

## 2. Pros & Cons Analysis

[[**Pros:**

* **Disk Space Management:** Allows users to install Docker Desktop and store its data on drives with more available space, preventing the primary OS drive from filling up.
* **Performance Optimization:** Placing Docker's virtual machine images and data on faster drives (e.g., SSDs) can lead to improved I/O performance for containers.
* **Organizational Compliance:** Enables adherence to specific IT policies that dictate where applications and their data should be stored within the system.
* **Simplified Backups:** Separating Docker data onto a dedicated drive can streamline backup processes for container-related files.
* **Clear Separation:** Keeps Docker-related files and data isolated from the primary operating system files, potentially aiding in system maintenance and troubleshooting.

**Cons:**

* **Complexity:** Custom installation is more involved than the default installation process, requiring careful execution of commands and understanding of the implications.
* **Potential Compatibility Issues:** Although rare, specifying non-standard paths could potentially lead to unforeseen compatibility issues with future Docker Desktop updates.
* **Increased Maintenance:** Users are responsible for ensuring the custom paths remain accessible and have adequate permissions after system changes or updates.
* **Troubleshooting Overhead:** Diagnosing issues related to custom paths might require more in-depth knowledge of Docker Desktop's architecture.
* **Limited Official Support for Edge Cases:** While custom installations are supported, troubleshooting highly specific issues related to unique custom configurations might have limited official support resources.]]

## 3. Prerequisites

[[**System Requirements:**

* **Windows 10 or 11 64-bit:** Ensure your operating system meets the minimum version requirements for Docker Desktop. *Reason: Docker Desktop relies on specific Windows features available in these versions.*
* **Virtualization Enabled:** Hyper-V or WSL 2 feature must be enabled in your BIOS and Windows Features. *Reason: Docker Desktop utilizes virtualization technologies for running containers.*
* **Sufficient Disk Space:** Ensure you have enough free space on the target drives for both the application installation and the container data. *Reason: Lack of disk space can lead to installation failures or performance issues.*
* **Minimum RAM:** Docker Desktop recommends a minimum amount of RAM (typically 4GB, but 8GB or more is recommended). *Reason: Running containers can be memory-intensive.*

**Administrative Permissions:**

* **Administrator Access:** You need administrator privileges on your Windows account to install and configure Docker Desktop. *Reason: The installation process requires system-level changes.*

**Pre-installation Preparations:**

* **Backup Important Data:** It's always recommended to back up any critical data before making significant system changes. *Reason: Although unlikely, installation issues could potentially lead to data loss.*
* **Uninstall Previous Docker Versions:** If you have older versions of Docker Desktop or Docker Engine installed, uninstall them cleanly. *Reason: Conflicts between different versions can cause installation problems.*

**Software Dependencies:**

* **WSL 2 (Recommended):** For optimal performance and resource utilization, it's highly recommended to have the Windows Subsystem for Linux 2 installed. *Reason: WSL 2 provides a lightweight and efficient environment for running Linux-based containers.*
* **Hyper-V (Alternative):** If WSL 2 is not an option, ensure Hyper-V is enabled. *Reason: Hyper-V is Microsoft's native hypervisor that Docker Desktop can utilize.*

**Knowledge Requirements:**

* **Basic Command Line Knowledge:** Familiarity with PowerShell or the Command Prompt will be helpful for executing the installation commands. *Reason: The custom installation process primarily involves command-line operations.*
* **Understanding of File Paths:** You need to understand how file paths work in Windows to specify the custom installation locations correctly. *Reason: Incorrect paths will lead to installation failures or data being stored in unintended locations.*]]

## 4. Installation Guide

This guide assumes you are using PowerShell for the installation process.

### 4.1 Prepare Custom Folders

First, create the directories where you want to install Docker Desktop and store its data. This example uses `C:\DevOps-Tools\Docker` for the application and `E:\appData\Docker\` for various data locations. Adjust these paths according to your needs.

<code>
New-Item -ItemType Directory -Path "C:\DevOps-Tools\Docker" -Force
New-Item -ItemType Directory -Path "E:\appData\Docker\wsl" -Force
New-Item -ItemType Directory -Path "E:\appData\Docker\hyperv" -Force
New-Item -ItemType Directory -Path "E:\appData\Docker\windows" -Force
</code>

* **Purpose:** These commands create the necessary folders for the Docker Desktop application and its associated data on your chosen drives.
* **Parameter/Argument Breakdown:**
    * `New-Item`: PowerShell cmdlet to create new items.
    * `-ItemType Directory`: Specifies that you want to create a directory (folder).
    * `-Path "..."`: Defines the full path where the new directory will be created.
    * `-Force`: Creates the directory even if parent directories do not exist.
* **Expected Output or Result:** Four new directories will be created at the specified paths if they don't already exist.

### 4.2 Define Download URL (Official Docker CDN)

Next, define a variable to store the download URL for the latest Docker Desktop installer.

<code>
$installerUrl = '[https://desktop.docker.com/win/main/amd64/Docker%20Desktop%20Installer.exe](https://desktop.docker.com/win/main/amd64/Docker%20Desktop%20Installer.exe)'
</code>

* **Purpose:** This line defines a variable containing the official download link for the Docker Desktop installer, ensuring you get the latest version.
* **Parameter/Argument Breakdown:**
    * `$installerUrl`: A variable name where the string value of the URL will be stored.
    * `'...'`: Encloses the string value of the Docker Desktop installer URL.
* **Expected Output or Result:** The variable `$installerUrl` will hold the URL of the Docker Desktop installer.

### 4.3 Prepare Your Downloads Folder

Ensure a local directory exists to temporarily store the installer.

<code>
$downloadDir   = 'C:\Users\zanyb\Downloads'
New-Item -Path $downloadDir -ItemType Directory -Force
</code>

* **Purpose:** This sets a variable for your Downloads directory and ensures the directory exists.
* **Parameter/Argument Breakdown:**
    * `$downloadDir`: A variable holding the path to your Downloads folder.
    * `New-Item -Path ... -ItemType Directory -Force`: Creates the Downloads directory if it doesn't exist.
* **Expected Output or Result:** The `$downloadDir` variable is set, and the Downloads directory exists.

### 4.4 Download the Installer into Downloads

Download the Docker Desktop installer using the URL defined earlier.

<code>
$installerPath = Join-Path $downloadDir 'DockerDesktopInstaller.exe'
Invoke-WebRequest `
    -Uri $installerUrl `
    -OutFile $installerPath `
    -UseBasicParsing
</code>

* **Purpose:** This command downloads the Docker Desktop installer from the official Docker website and saves it to your Downloads folder.
* **Parameter/Argument Breakdown:**
    * `$installerPath`: A variable combining the download directory and the installer filename.
    * `Invoke-WebRequest`: PowerShell cmdlet to send HTTP and HTTPS requests.
    * `-Uri $installerUrl`: Specifies the URL to download from.
    * `-OutFile $installerPath`: Defines the local path where the downloaded file will be saved.
    * `-UseBasicParsing`: Forces `Invoke-WebRequest` to use the basic parser for the response content, which can be useful in certain network environments.
* **Expected Output or Result:** The `DockerDesktopInstaller.exe` file will be downloaded and saved in your Downloads folder.

### 4.5 Build Argument List for Docker Desktop Installer

Construct the arguments to be passed to the Docker Desktop installer for a custom installation.

<code>
$installArgs = @(
    'install',                                          # Begin installation command
    '--quiet',                                          # Suppress output
    '--accept-license',                                 # Auto-accept subscription terms
    '--installation-dir=C:\DevOps-Tools\Docker',         # Program files path
    '--wsl-default-data-root=E:\appData\Docker\wsl',     # WSL2 distro disk path
    '--hyper-v-default-data-root=E:\appData\Docker\hyperv', # Hyper-V VM disk path
    '--windows-containers-default-data-root=E:\appData\Docker\windows', # Windows-containers path
    '--always-run-service'                              # Start Docker service automatically
)
</code>

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
* **Expected Output or Result:** The `$installArgs` variable will contain an array of installation parameters.

### 4.6 Execute Installer from Downloads

Run the downloaded installer with the custom arguments.

<code>
Start-Process `
    -FilePath $installerPath `
    -ArgumentList $installArgs `
    -Wait
</code>

* **Purpose:** This command executes the Docker Desktop installer with the arguments defined in the `$installArgs` variable, performing the custom installation.
* **Parameter/Argument Breakdown:**
    * `Start-Process`: PowerShell cmdlet to start a new process.
    * `-FilePath $installerPath`: Specifies the path to the executable file to run (the Docker Desktop installer).
    * `-ArgumentList $installArgs`: Provides the array of arguments to be passed to the installer.
    * `-Wait`: Makes the PowerShell script wait until the installer process finishes before proceeding.
* **Expected Output or Result:** Docker Desktop will be installed in the `C:\DevOps-Tools\Docker` directory, and its data will be stored in the `E:\appData\Docker\` subdirectories. The installation will proceed silently without a GUI.

### 4.7 Update System Path

Add the custom binary directory to your system's PATH environment variable so you can run `docker` and `docker-compose` commands from any command prompt.

<code>
setx PATH "$($Env:PATH);C:\DevOps-Tools\Docker" /M
</code>

* **Purpose:** This command modifies the system-wide PATH environment variable to include the directory where the Docker executables (`docker.exe`, `docker-compose.exe`, etc.) are installed. This allows you to run these commands without specifying their full path.
* **Parameter/Argument Breakdown:**
    * `setx`: Command-line utility to set environment variables.
    * `PATH`: The name of the environment variable being modified.
    * `"$($Env:PATH);C:\DevOps-Tools\Docker"`: The new value for the PATH variable. It appends the custom Docker installation directory to the existing PATH. `$Env:PATH` retrieves the current value of the PATH variable.
    * `/M`: Specifies that the environment variable should be set in the system environment (requires administrator privileges).
* **Expected Output or Result:** The system PATH environment variable will be updated to include `C:\DevOps-Tools\Docker`. You might need to restart your command prompt or PowerShell session for the changes to take effect.

Verify the Docker CLI is accessible from the custom installation location:

<code>
$where = (Get-Command docker).Source
Write-Host "Docker CLI installed at: $where"
</code>

* **Purpose:** This command checks the location from where the `docker` command is being executed, confirming it's pointing to your custom installation directory.
* **Parameter/Argument Breakdown:**
    * `(Get-Command docker).Source`: Retrieves the full path of the `docker` executable.
    * `$where`: A variable storing the path of the `docker` executable.
    * `Write-Host "..."`: Displays the value of the `$where` variable in the PowerShell console.
* **Expected Output or Result:** The output should display `Docker CLI installed at: C:\DevOps-Tools\Docker\docker.exe` (or your chosen custom installation path).

### 4.8 Restart Machine

A system restart is often required for all Docker Desktop components and environment variable changes to take effect properly.

<code>
Restart-Computer -Force
</code>

* **Purpose:** This command initiates a forced restart of your computer.
* **Parameter/Argument Breakdown:**
    * `Restart-Computer`: PowerShell cmdlet to restart the local computer.
    * `-Force`: Restarts the computer without prompting the user to save unsaved work. Use with caution.
* **Expected Output or Result:** Your computer will restart.

## 5. Verification Steps

After restarting your machine, follow these steps to verify that Docker Desktop has been installed correctly with the custom paths.

### 5.1 Check Docker Version

Open a new PowerShell or Command Prompt window and run the following command:

<code>
docker --version
</code>

* **Purpose:** This command checks if the `docker` command-line interface (CLI) is installed and accessible, and it displays the installed Docker version.
* **Parameter/Argument Breakdown:**
    * `docker`: The Docker CLI executable.
    * `--version`: An argument that instructs the Docker CLI to output its version information.
* **Expected Output or Result:** You should see output similar to `Docker version <version>, build <commit>`.

### 5.2 Check Disk Image Location

Verify that the WSL 2 disk image (if you chose WSL 2 integration) is located in your custom data root directory.

<code>
Get-ChildItem -Path "E:\appData\Docker\wsl"
</code>

* **Purpose:** This command lists the files and folders within your custom WSL 2 data root directory.
* **Parameter/Argument Breakdown:**
    * `Get-ChildItem`: PowerShell cmdlet to get the items (files and folders) in a specified location.
    * `-Path "E:\appData\Docker\wsl"`: Specifies the path to your custom WSL 2 data root directory.
* **Expected Output or Result:** You should see files related to the WSL 2 Docker distribution, such as a `.vhdx` file (virtual hard disk). The exact filename might vary.

## 6. Exceptions

[[**1. `C:\Program Files\Docker\cli-plugins`:**

* **Purpose:** This directory stores Docker CLI plugins, which extend the functionality of the `docker` command. These plugins are often installed separately and integrate with the core Docker CLI.
* **Why Specific Path:** This is a standard location where Docker expects to find CLI plugins to automatically discover and load them. Deviating from this path would require manual configuration and might break plugin functionality.
* **Typical Files:** Executable files (`.exe` on Windows) and potentially supporting libraries for various Docker plugins (e.g., for cloud integrations, build tools).
* **Data Accumulation:** Typically small, growing only when new CLI plugins are installed.
* **Modification/Relocation:** It is generally **not recommended** to manually modify or relocate this directory, as it can lead to Docker CLI malfunctions.
* **Official Documentation:** Refer to the official Docker documentation on "Extend Docker with plugins" for more details.

**2. `C:\ProgramData\DockerDesktop`:**

* **Purpose:** This directory holds application-wide data and configuration settings for Docker Desktop itself. This includes settings related to the Docker Engine, networking, and other core functionalities.
* **Why Specific Path:** `C:\ProgramData` is a standard Windows location for application data that is shared across all users on the system. Docker Desktop relies on this consistent location for its global configuration.
* **Typical Files:** Configuration files (often in `.json` or `.ini` format), logs, and potentially some internal state databases.
* **Data Accumulation:** Can grow over time depending on Docker Desktop usage and logging levels.
* **Modification/Relocation:** **Modifying or relocating this directory is strongly discouraged** as it can lead to Docker Desktop instability or failure to start.
* **Official Documentation:** While not a specific page, information related to Docker Desktop's core operation and settings implicitly refers to this directory.

**3. `C:\Users\[username]\.docker`:**

* **Purpose:** This user-specific directory stores configuration and credentials related to the Docker CLI for the individual user. This includes authentication tokens for Docker Hub and
