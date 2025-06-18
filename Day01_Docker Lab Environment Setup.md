# Docker Lab Environment Setup: Comprehensive Guide

This guide covers setting up a complete Docker lab environment, from virtual machine installation and configuration to Docker Engine installation on multiple Linux VMs.

---

## **Part 1: Initial Virtual Machine Setup (VMware Workstation)**

This part details the initial steps for setting up your virtual machines using VMware Workstation.

### **I. Accessing Necessary Files (Only works in academy computer | setup files are in my kakaotalk)**

* **Location:** All required files are located on the **D drive**.
* **Shared Folder Access:**
    * Open the "Run" window (press **Windows Key + R**).
    * Type `\\192.168.11.30` and press Enter.
    * **ID:** `cloud1`
    * **Password:** `cloud1234!`
    * Navigate to the `Kim Jeong-woo` folder, then enter the `Docker` folder.
* **Copy Files:** Copy the entire `Docker` folder to your personal D drive (e.g., `D:\<Your_Name>\ Docker`).
    * *Note:* For those using lecture hall PCs, these files are often pre-downloaded. For personal notebooks or those watching the recorded video, copying this folder is essential.

### **II. Launching & Configuring VMware Workstation 16**

* **Launch VMware Workstation 16 Pro.** (If the icon isn't on your desktop, search "VM" in the Windows search bar).
    * *Tip:* VMware Workstation 16 Pro is currently free and can be downloaded from `broadcom.com` (the company that acquired VMware).
* **Initial Cleanup (if applicable):**
    * If you see any existing virtual machines (e.g., `Client1`, `Client2`, `Server1`) in the left panel, **delete them**.
        * Right-click the VM and select "Remove" (or select the VM and click the 'X' button).
* **Create New Virtual Machine:**
    * Click **File** $\rightarrow$ **New Virtual Machine...**
    * **Choose a Custom (advanced)** configuration $\rightarrow$ Click **Next**.
    * **Installer Disc Image:** Select "I will install the operating system later" $\rightarrow$ Click **Next**.
    * **Guest Operating System:** Select **Linux**.
    * **Version:** Select **CentOS 8 64-bit**.
        * *(Visual: Dropdown menu showing various Linux distributions, with CentOS 8 64-bit highlighted)*
    * Click **Next**.
* **Name the Virtual Machine:**
    * **Virtual machine name:** `docker1`
    * **Location:** Click "Browse" and set the path to your personal Docker folder: `D:\<Your_Name>\Docker\VM Image\docker1`
        * *(Visual: Screenshot showing the "Name the Virtual Machine" dialog with "docker1" entered and the custom path)*
    * Click **Next**.
* **Processors:**
    * **Number of processors:** `1`
    * **Number of cores per processor:** `4` (This results in a total of 4 virtual CPUs).
        * *(Visual: Screenshot of the "Processors" settings, highlighting "Number of cores per processor" set to 4)*
    * Click **Next**.
* **Memory:** Set the memory to `3072 MB` (3GB).
        * *(Visual: Screenshot of the "Memory for the Virtual Machine" slider, set to 3072 MB)*
    * Click **Next**.
* **Network Adapter (Crucial for connectivity!):**
    * Select **Use network address translation (NAT)**.
        * *Note:* NAT is recommended for this lab to easily provide internet access to your VMs.
    * Click **Next**.
* **I/O Controller Types:** Keep the default option (LSI Logic SAS) $\rightarrow$ Click **Next**.
* **Disk Type:** Keep the default option (SCSI) $\rightarrow$ Click **Next**.
* **Specify Disk Capacity:**
    * **Maximum disk size (GB):** `100`
    * Select **Split virtual disk into multiple files**.
        * *(Visual: Screenshot of the "Specify Disk Capacity" dialog, with 100 GB and "Split virtual disk into multiple files" selected)*
    * Click **Next**.
* **Specify Disk File:** Keep the default option $\rightarrow$ Click **Next**.
* **Review Settings:**
    * Review the summary of your VM settings.
    * Click **Finish**.

---

## **Part 2: Installing CentOS Stream 9 on `docker1`**

This part guides you through the operating system installation process for your first virtual machine.

### **I. Start the Virtual Machine and Begin Installation**

* **Start the VM:**
    * In VMware Workstation, select `docker1` from the left panel.
    * Click **Power on this virtual machine** (the green play button).
* **Resolve Black Screen Issue (if applicable):**
    * If the VM screen remains black, it usually means the ISO image was not properly inserted.
    * Go to **VM** $\rightarrow$ **Settings** $\rightarrow$ **CD/DVD (IDE)**.
    * Ensure "Use ISO image file" is selected and points to the correct ISO file: `D:\<Your_Name>\Docker\01_CentOS-Stream-9-latest-x86_64-dvd1.iso`.
    * Click **OK** and restart the VM.

### **II. CentOS Installation Steps**

1.  **Language Selection:**
    * Select **Korean** (or your preferred language).
    * Click **Continue**.

2.  **Installation Summary Screen:**
    * You will see a summary of installation options. Red exclamation marks indicate required settings.
    * **Host name & Network:**
        * Click "Ethernet (ens33)" to toggle it **ON**.
        * Click "Configure..." $\rightarrow$ Go to the **IPv4 Settings** tab.
        * **Method:** Select **Manual**.
        * Click **Add**.
        * **Address:** `192.168.2.10`
        * **Netmask:** `24`
        * **Gateway:** `192.168.2.254`
        * **DNS servers:** `168.126.63.1` (This is a common Korean DNS server, e.g., KT DNS).
        * **Search domains:** `example.com`
        * Click **Save** $\rightarrow$ Click **Done**.
        * **Host name:** Change `localhost.localdomain` to `docker1.example.com` $\rightarrow$ Click **Apply**.
    * **Time & Date:**
        * Click "Time & Date" $\rightarrow$ Select **Asia/Seoul** (or your local timezone).
        * Click **Done**.
    * **Root Account (root password):**
        * Click "Root Account (root password)".
        * **Root Password:** `centos`
        * **Confirm:** `centos`
        * **Crucial:** Check the box for **"Allow root password for SSH login"**. (Do **NOT** check "Lock the root account").
        * Click **Done**.
    * **User Creation:**
        * For this lab, you can **skip creating a regular user**. We will primarily use the `root` account for simplicity.
        * Click **Done** without adding any users.
    * **Installation Destination:**
        * Click "Installation Destination".
        * Select the **100GB disk** you configured.
        * Click **Done**.
    * **Software Selection:**
        * Choose **Server with GUI** (this provides a graphical desktop environment).
        * Click **Done**.

3.  **Begin Installation:**
    * Click **Begin Installation**.
    * The installation process will now run. This typically takes 10-15 minutes depending on your system's performance.

4.  **Reboot System:**
    * Once the installation completes, click **Reboot System**.

### **III. Post-Installation Verification (First Boot)**

1.  **Welcome Screen:**
    * After rebooting, you'll see a welcome screen. Click **Next** through any privacy, online accounts, or initial setup screens.
2.  **Start Using CentOS Stream:**
    * Click this button on the final welcome screen.
3.  **Root Login:**
    * At the login screen, click "Not listed?".
    * Type `root` and press Enter.
    * Enter the password `centos` and press Enter.

4.  **Initial Connectivity Checks (in Terminal):**
    * Open **Terminal** (usually a black icon on the bottom bar or search for "Terminal").
    * Check IP address: `ip address`
        * Verify that `ens33` shows `192.168.2.10/24`.
    * Check Gateway/Routing: `ip route`
        * Verify `default via 192.168.2.254`.
    * Check DNS server: `cat /etc/resolv.conf`
        * Verify `nameserver 168.126.63.1`.
    * Ping Google DNS: `ping -c 5 168.126.63.1`
        * You should see successful replies.
    * NSLookup Google: `nslookup www.google.com`
        * This should successfully resolve `www.google.com` to its IP addresses.
    * Access Browser: Open Firefox (the browser icon on the bottom bar).
        * Type `www.google.com` or `www.centos.org` to confirm active internet connectivity.

---

## **Part 3: Finalizing `docker1` Configuration with Setup Script**

This part automates many common post-installation configurations to streamline your Docker environment setup.

### **I. Download and Run Setup Scripts**

* **Download Setup Scripts:**
    * Open Terminal (if it's closed).
    * Download the `ping.sh` and `setup.sh` scripts from GitHub:
        * `wget -q https://raw.githubusercontent.com/ncs10322/docker/main/ping.sh`
        * `wget -q https://raw.githubusercontent.com/ncs10322/docker/main/setup.sh`
    * Verify files are downloaded: `ls` (You should see `ping.sh` and `setup.sh` listed).

* **Run Network Test Script:**
    * `sh ping.sh`
    * This script will automatically run a series of network tests (including pinging Google DNS and accessing `google.com`). All tests should pass.

* **Run Main Setup Script:**
    * `sh setup.sh`
    * This script automates several important configurations:
        * **SELinux:** Changes SELinux mode to `permissive`. (By default, it's 'enforcing', which can sometimes interfere with Docker).
            * *Verification after reboot:* `sestatus` (Current mode should be `permissive`).
        * **FirewallD:** Disables the FirewallD service.
            * *Verification after reboot:* `systemctl status firewalld` (Should show `inactive (dead)`).
        * **GNOME Tweaks & Extensions:** Installs `gnome-tweaks` and `gnome-extensions` packages for desktop customization.
        * **Memory & Screen Settings:** Disables sleep/hibernate and screen saver to prevent interruptions.
        * **`.vimrc` file:** Creates a `.vimrc` file with basic Vim editor configurations (e.g., line numbers, dark background, syntax highlighting).
            * *Verification:* `cat ~/.vimrc`
        * **`.bashrc` aliases & PS1:** Adds useful aliases (e.g., `rm -i`, `cp -i`, `mv -i`, `ls --color=auto --time-style=long-iso`) and customizes the shell prompt (`PS1`).
            * *Verification:* `cat ~/.bashrc`
        * **`/etc/hosts`:** Modifies the `/etc/hosts` file to include entries for both `docker1` and `docker2`.
        * **EPEL & Git:** Installs the EPEL repository and Git.
    * **Important:** The system will automatically **reboot** after `setup.sh` completes.

---

## **Part 4: Creating `docker2` (Cloning `docker1`)**

This part details how to efficiently create a second virtual machine by cloning the first, and then how to properly configure its network and hostname.

### **I. Prepare `docker1` for Cloning**

* **Power Off `docker1`:**
    * In VMware Workstation, right-click `docker1` in the left panel.
    * Select **Power** $\rightarrow$ **Power Off**.
        * *(Alternative from inside the VM Terminal):* Type `init 0` (or `poweroff`) and press Enter. It's crucial to power off the VM completely before cloning.

### **II. Clone `docker1` to `docker2`**

* **Start the Cloning Wizard:**
    * In VMware Workstation, right-click `docker1` in the left panel.
    * Select **Manage** $\rightarrow$ **Clone...**
    * Click **Next** on the "Welcome to the Clone Virtual Machine Wizard" screen.
* **Select Clone Type:**
    * Choose **Create a full clone**.
        * *Explanation:* A **full clone** creates a completely independent copy, which is ideal for separate lab environments like this Docker setup. A linked clone is smaller but depends on the original VM.
    * Click **Next**.
* **Name & Location for `docker2`:**
    * **Virtual machine name:** `docker2`
    * **Location:** Click "Browse" and set the path to `D:\<Your_Name>\Docker\VM Image\docker2`.
        * *(Visual: Screenshot showing "Virtual machine name: docker2" and the custom location in the wizard)*
    * Click **Finish**.
* **Wait for Cloning:**
    * The cloning process will begin. This may take several minutes depending on your system's performance.

### **III. Configure `docker2` After Cloning**

1.  **Power On `docker2`.**
2.  **Login as `root`** (password: `centos`).
3.  **Change Hostname (Essential for unique identification):**
    * `hostnamectl set-hostname docker2.example.com`
    * Type `exit` and press Enter (this logs you out and then logs back in, applying the hostname change to the terminal prompt).
    * Login again as `root`.
    * *Verification:* Your terminal prompt should now show `root@docker2 ~]#`.
        * *(Visual: Terminal showing command `hostnamectl set-hostname docker2.example.com` and the new prompt)*
4.  **Change IP Address (Crucial for network uniqueness):**
    * Since `docker1` has the IP address `192.168.2.10`, `docker2` needs a different, unique IP within the same network.
    * Type `nmtui` and press Enter (this opens a text-based user interface for network configuration).
    * Select "Edit a connection" $\rightarrow$ Highlight `ens33` $\rightarrow$ Press Enter (to **Edit**).
    * Navigate to the "IPv4 CONFIGURATION" section $\rightarrow$ Select **Manual**.
    * Go to "Addresses" $\rightarrow$ Highlight `192.168.2.10/24` $\rightarrow$ Press **Delete**.
    * Click **Add**.
    * **Address:** `192.168.2.20/24` (This is the new IP for `docker2`).
    * **Gateway:** `192.168.2.254` (Keep the same gateway as `docker1`).
    * **DNS servers:** `168.126.63.1` (Keep the same DNS server).
    * **Search domains:** `example.com` (Keep the same search domain).
    * *(Visual: Screenshot of the IPv4 settings in `nmtui` with the IP changed to `192.168.2.20`)*
    * Navigate to "OK" $\rightarrow$ Select "Activate a connection" $\rightarrow$ Select `ens33` (Press Enter to Deactivate, then press Enter again to Activate) $\rightarrow$ Select "Back" $\rightarrow$ Select "Quit".
    * *Verification:* Type `ip address` and press Enter. Confirm that `ens33` now shows `192.168.2.20`.
    * *Test connectivity to `docker1` (from `docker2`):* `ping -c 5 192.168.2.10` (This should successfully ping `docker1`).
5.  **Update `/etc/hosts` (for proper name resolution between VMs):**
    * `gedit /etc/hosts &` (This opens a graphical text editor in the background).
    * Add the following line (below the `docker1.example.com` entry):
        `192.168.2.20 docker2.example.com docker2`
    * Save the file and close `gedit`.
    * *Verification:* `cat /etc/hosts`
    * *Test name resolution from `docker2`:* `ping docker1.example.com` (This should now correctly resolve and ping `192.168.2.10`).

---

## **Part 5: Installing Docker Engine**

This final part guides you through the process of installing the Docker Engine on your newly prepared Linux virtual machines.

### **I. Retrieve Docker Installation Steps**

* **Source:** Always refer to the official Docker documentation for the most up-to-date and reliable installation instructions.
    * Search "Docker install CentOS" on Google.
    * *Direct Link:* [https://docs.docker.com/engine/install/centos/](https://docs.docker.com/engine/install/centos/)
    * *(Visual: Screenshot of the Docker documentation page for CentOS installation)*

### **II. Docker Engine Installation Steps (for both `docker1` and `docker2`)**

**Perform these steps on `docker1` first, then repeat for `docker2`.**

1.  **Add Docker Repository:**
    * This step adds the official Docker repository to your system's package manager.
    * `sudo yum -y install yum-utils` (Install `yum-utils` if it's not already on your system. This package provides the `yum-config-manager` utility).
        * *Note:* While CentOS Stream 9 primarily uses `dnf` as its package manager, `yum` commands are often aliased to `dnf` for backward compatibility.
    * `sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo`
    * *Verification:* `yum repolist` (You should see `docker-ce-stable` listed in the output).

2.  **Download Convenience Script (Optional but Recommended):**
    * This script streamlines the Docker Engine installation process by handling dependencies and configurations.
    * `curl -fsSL https://get.docker.com -o get-docker.sh`
        * `curl`: A command-line tool for transferring data with URL syntax.
        * `-fsSL`: Flags for silent, fail-fast, and redirect following.
        * `https://get.docker.com`: The URL for the Docker installation script.
        * `-o get-docker.sh`: Saves the downloaded script as `get-docker.sh`.
    * *Verification:* `ls` (Confirm that `get-docker.sh` is now present in your current directory).
    * *Preview what the script will do (optional but good practice):* `sh get-docker.sh --dry-run`

3.  **Run Docker Installation Script:**
    * Execute the downloaded script to install Docker Engine.
    * `sh get-docker.sh`
    * This script will install Docker Engine, Containerd (the container runtime), and Docker Compose (a tool for defining and running multi-container Docker applications).

4.  **Start and Enable Docker Service:**
    * After installation, you need to start the Docker service and configure it to launch automatically at boot.
    * `sudo systemctl enable --now docker`
        * `systemctl enable`: Configures the Docker service to start automatically when the system boots up.
        * `--now`: Immediately starts the Docker service after enabling it.
    * *Verification:* `sudo systemctl status docker`
        * You should see output indicating `Active: active (running)` and `enabled`.
        * *(Visual: Screenshot showing `systemctl status docker` output with `active (running)` and `enabled` highlighted)*

5.  **Test Docker Installation (Hello World):**
    * Run a simple test container to verify Docker is working correctly.
    * `docker container run hello-world`
    * This command performs several actions:
        * Downloads the `hello-world` image from Docker Hub (if not already present locally).
        * Creates a new container based on that image.
        * Runs the container.
        * The container executes a small program that prints a "Hello from Docker!" message and then exits.
        * *(Visual: Terminal output showing "Hello from Docker!" message)*

6.  **Test Docker Installation (Nginx - for `docker1`):**
    * This test demonstrates running a more functional container (a web server) and accessing it from your host machine.
    * **On `docker1` only:** `docker container run -p 80:80 nginx`
        * `docker container run`: Command to create and run a new container.
        * `-p 80:80`: Maps port 80 of your `docker1` VM (host) to port 80 inside the Nginx container.
        * `nginx`: The name of the Docker image to use (Nginx web server).
    * **Access from Windows:** Open your web browser on your Windows host machine.
        * Navigate to `http://192.168.2.10/`. You should see the default Nginx welcome page.
    * *Note:* The `docker container run -p 80:80 nginx` command will keep the Nginx container running in the foreground of your Terminal. To continue working in the VM, open a **new Terminal window** (e.g., Ctrl+Shift+T).

### **III. Final Lab Wrap-up**

* **Repeat Docker Engine Installation on `docker2`:**
    * Go to your `docker2` VM.
    * Perform all steps from **Part 5, steps 1 through 5** (installing Docker Engine, starting/enabling, and testing `hello-world`).
    * You don't need to run Nginx on `docker2` for now, but you can if you wish.

* **Power Off Both VMs:**
    * Once you have completed all steps and verified Docker is running on both, it is **crucial to power off both VMs** to save their current state.
    * **In each VM's Terminal:** `init 0` (or `poweroff`) and press Enter.
    * *Alternatively, in VMware Workstation:* Right-click the VM in the left panel $\rightarrow$ **Power** $\rightarrow$ **Power Off**.

