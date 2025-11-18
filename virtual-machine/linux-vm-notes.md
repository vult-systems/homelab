# Linux VM Notes

These notes document the foundational concepts and setup process for creating a Linux virtual machine environment for testing production pipeline tools (Ayon, Deadline, etc.).

---

## Package Managers

### What Are Package Managers?
Package managers are tools that automate the process of installing, updating, configuring, and removing software. They solve the problem of manually downloading installers, tracking dependencies, and managing updates.

### Common Package Managers

#### Windows - winget
```powershell
# Search for a package
winget search virtualbox

# Install a package
winget install Oracle.VirtualBox

# Update a package
winget upgrade Oracle.VirtualBox

# List installed packages
winget list
```

#### Linux (Ubuntu/Debian) - apt
```bash
# Update package list
sudo apt update

# Install a package
sudo apt install package-name

# Upgrade a package
sudo apt upgrade package-name

# Remove a package
sudo apt remove package-name
```

#### Node.js - npm
```bash
# Install a package locally
npm install package-name

# Install a package globally
npm install -g package-name

# Update packages
npm update

# Run scripts defined in package.json
npm run build
```

### Key Concepts
- **Dependencies**: Packages that other software needs to function
- **Version Locking**: Ensuring specific versions are installed (e.g., package-lock.json)
- **Global vs Local**: Some packages install system-wide, others per-project

---

## Environment Variables & PATH

### What is PATH?
PATH is an environment variable that contains a list of directories. When you type a command in the terminal, the system searches these directories in order to find the executable.

### How PATH Works
```
PATH = C:\Windows\System32;C:\Program Files\nodejs;C:\Users\YourName\bin
         ^                  ^                      ^
         Directory 1        Directory 2            Directory 3
```

The semicolon (`;`) separates each directory path.

### Viewing PATH
```powershell
# View entire PATH (one long string)
$env:PATH

# View PATH as a list (one directory per line)
$env:PATH -split ';'

# Search PATH for specific directories
$env:PATH -split ';' | Select-String "VirtualBox"
```

### Modifying PATH (PowerShell)

#### Adding to PATH (User Level)
```powershell
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\Oracle\VirtualBox", "User")
```

#### Removing from PATH
```powershell
# Remove all entries containing "VirtualBox"
$newPath = ($env:PATH -split ';' | Where-Object { $_ -notlike '*VirtualBox*' }) -join ';'
[Environment]::SetEnvironmentVariable('Path', $newPath, 'User')
```

#### Replacing a Typo in PATH
```powershell
# Fix a typo in PATH
[Environment]::SetEnvironmentVariable("Path", $env:Path.Replace("Oracl", "Oracle"), "User")
```

### Important Notes
- Changes to PATH require **closing and reopening the terminal** to take effect
- The terminal loads PATH at startup from the registry
- Modifying PATH in the current session doesn't update persistent PATH

### Using Executables Not in PATH
```powershell
# Full path with call operator (&)
& "C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" --version
```

---

## VMs vs Containers

### Virtual Machines (Virtualization)
- **What it is**: Simulates an entire computer (CPU, RAM, storage, network card)
- **Runs**: Complete operating system with its own kernel
- **Isolation**: Strong - each VM is completely independent
- **Size**: Heavy (gigabytes per VM)
- **Startup**: Slower (boots entire OS)
- **Use case**: Running different operating systems, strong isolation needs

**Example Tools**: VirtualBox, VMware, Hyper-V

### Containers (Containerization)
- **What it is**: Isolates applications and their dependencies
- **Runs**: Shares host OS kernel, packages just the application layer
- **Isolation**: Lighter - shares kernel with host
- **Size**: Light (megabytes per container)
- **Startup**: Fast (seconds)
- **Use case**: Deploying applications consistently across environments

**Example Tools**: Docker, Kubernetes (container orchestration)

### Our Setup: VMs + Docker
```
Windows PC (Host)
  └── VirtualBox (Virtualization)
      └── Ubuntu Linux VM (Guest OS)
          └── Docker (Containerization)
              └── Ayon Containers
                  ├── MongoDB (database)
                  ├── Ayon Server (web server)
                  └── Redis (caching)
```

**Why this layering?**
- VM gives us a full Linux environment to work in
- Docker provides portable, reproducible application deployment
- Mirrors production server architecture

---

## VirtualBox Setup

### Installation (Windows)

#### Using winget
```powershell
# Search for VirtualBox
winget search virtualbox

# Install VirtualBox
winget install Oracle.VirtualBox
```

#### Verification
```powershell
# After installation, close and reopen PowerShell, then:
VBoxManage --version
```

### VirtualBox Components
- **VirtualBox Application**: GUI for managing VMs
- **VBoxManage**: Command-line tool for VM management
- **Virtual Machine Manager**: Creates and configures VMs

---

## Resource Allocation

### Understanding VM Resources
When creating a VM, you allocate a portion of your host machine's resources:

- **CPU Cores**: Number of processor cores the VM can use
- **RAM**: Amount of memory dedicated to the VM
- **Storage**: Virtual disk size for the VM's filesystem
- **Network**: How the VM connects to networks

### Guidelines for Allocation

#### Example Host: Ryzen 9 3950X (16 cores), 64GB RAM

**For Testing/Development VM:**
- **CPU**: 4 cores (leaves 12 for host)
- **RAM**: 8GB (leaves 56GB for host)
- **Storage**: 50-100GB virtual disk
- **Network**: Bridged (VM gets its own IP address)

**For Production Server:**
- Depends on workload and whether server is dedicated
- Monitor actual usage and adjust as needed
- Can modify when VM is powered off

### Resource Allocation Considerations
- **Leave headroom for host**: Don't allocate 100% of resources
- **Workload matters**: Docker containers need memory; consider future growth
- **Adjustable**: VirtualBox allows changing RAM/CPU when VM is off
- **Storage**: Virtual disks can grow dynamically (if configured)

---

## Ubuntu Server Installation

### Downloading Ubuntu Server

#### Find Available Versions
```powershell
# List available ISOs for Ubuntu 24.04
Invoke-WebRequest -Uri "https://releases.ubuntu.com/24.04/" -UseBasicParsing | 
  Select-Object -ExpandProperty Links | 
  Where-Object { $_.href -like "*server*amd64.iso" }
```

#### Download ISO
```powershell
# Download Ubuntu 24.04.3 Server
Invoke-WebRequest -Uri "https://releases.ubuntu.com/24.04/ubuntu-24.04.3-live-server-amd64.iso" -OutFile "$env:USERPROFILE\Downloads\ubuntu-24.04-server.iso"
```

#### Verify Download
```powershell
# Check if file exists
Test-Path "$env:USERPROFILE\Downloads\ubuntu-24.04-server.iso"
```

### Ubuntu Version Selection

**LTS (Long Term Support) vs Regular Releases:**
- **LTS**: Every 2 years, 5 years of support, stable
  - Example: 24.04 LTS (supported until 2029)
- **Regular**: Every 6 months, 9 months of support, latest features
  - Example: 25.10 (supported until July 2026)

**For production servers**: Use LTS for stability and long-term support

### Next Steps (Covered in Future Sessions)
1. Create VM in VirtualBox using downloaded ISO
2. Configure VM settings (RAM, CPU, network)
3. Install Ubuntu Server
4. Set up SSH access
5. Install Docker
6. Deploy Ayon using docker-compose

---

## Common Patterns & Commands

### Terminal Navigation
```bash
# Print working directory
pwd

# List files/directories
ls
ls -la  # Detailed list including hidden files

# Change directory
cd /path/to/directory
cd ..   # Go up one directory
cd ~    # Go to home directory

# Create directory
mkdir new-folder

# Move/rename files
mv source.txt destination.txt
mv file.txt /path/to/new/location/
```

### File Operations
```bash
# View file contents
cat filename.txt
less filename.txt  # Paginated view

# Create empty file
touch newfile.txt

# Edit file (common editors)
nano filename.txt   # Beginner-friendly
vim filename.txt    # Advanced
```

### System Information
```bash
# Check Ubuntu version
lsb_release -a

# Check system resources
free -h          # RAM usage
df -h            # Disk usage
htop             # Interactive process viewer (if installed)
```

---

## Key Takeaways

1. **Package managers automate software management** across different platforms (winget, apt, npm)

2. **PATH tells the system where to find executables** - modify it to make commands globally available

3. **VMs provide full OS isolation**, containers provide application isolation - often used together

4. **Resource allocation balances VM needs with host requirements** - leave headroom for your workstation

5. **LTS releases provide stability** for production servers with long-term support

6. **The pattern is consistent**: Search → Install → Configure → Use

---

## Troubleshooting

### Command Not Found
**Problem**: Terminal says command doesn't exist
**Solution**: 
1. Check if installed: `Get-Command command-name` (PowerShell) or `which command-name` (Linux)
2. Check PATH: Is the executable's directory in PATH?
3. Use full path if not in PATH

### PowerShell Session Not Refreshing
**Problem**: Environment variable changes don't take effect
**Solution**: Close entire PowerShell window and open fresh session

### VM Performance Issues
**Problem**: VM is slow or unresponsive
**Solution**:
1. Check resource allocation - too little or too much?
2. Monitor host system resources
3. Adjust VM settings when powered off

---

## Reference Links

- [VirtualBox Documentation](https://www.virtualbox.org/wiki/Documentation)
- [Ubuntu Server Guide](https://ubuntu.com/server/docs)
- [Docker Documentation](https://docs.docker.com/)
- [Ayon Documentation](https://ayon.ynput.io/)

