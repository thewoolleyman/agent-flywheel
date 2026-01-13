# Hosting Options for Agent Flywheel

This document covers various hosting options for running agent-flywheel, from cloud VPS providers to local virtualization on macOS.

## Cloud VPS Providers

### Contabo VPS

Recommended cloud VPS provider for agent-flywheel setup:

- [Contabo VPS Plans](https://contabo.com/en-us/vps/)
- [Contabo Server Configuration](https://new.contabo.com/servers/vps)

**Recommended specs:**
- 64GB RAM minimum
- Ubuntu 24.04 LTS

## Local Virtualization on macOS

For development or testing purposes, you can run agent-flywheel locally on macOS using virtualization.

## Lima VM (Linux Virtual Machines)

Lima launches Linux virtual machines with automatic file sharing and port forwarding, similar to WSL2 on Windows.

### What is Lima?

Lima (LInux MAchine) provides the simplest way to run Linux VMs on macOS with seamless file sharing and port forwarding. It uses Apple's native virtualization framework for optimal performance.

### Installation

```bash
brew install lima
```

### Quick Setup

**Create Ubuntu VM with recommended resources:**
```bash
limactl create --name=ubuntu24 --memory=32 --cpus=8 template:ubuntu-lts
```

**Start and access the VM:**
```bash
limactl start ubuntu24
limactl shell ubuntu24
```

**Stop and restart the VM:**
```bash
limactl stop ubuntu24      # Stop the VM (preserves state)
limactl start ubuntu24     # Start it again
limactl list               # Check VM status
limactl delete ubuntu24    # Remove VM completely (destructive)
```

### Key Features

- **Blazing Fast Setup** - Go from zero to Linux in one or two commands
- **No GUI Overhead** - All terminal-based, clean and efficient
- **Lightweight and Native** - Uses Apple's native virtualization framework
- **No ISO Files** - Uses templates for quick provisioning
- **Automatic File Sharing** - Mounts your home directory inside the VM
- **Port Forwarding** - Seamless network access

### File System Access

Lima mounts your entire home directory inside the VM for seamless access. Write permissions are available under `/tmp/lima` by default.


### Resource Configuration

**Configure VM resources during creation:**
```bash
limactl create --name=ubuntu24 --memory=32 --cpus=8 template:ubuntu-lts
```

The `--memory` flag takes a number in GiB (e.g., `32` for 32GB).

**For agent-flywheel requirements:**
- Memory: 32GB minimum for local development, 64GB for full experience
- CPUs: 8+ cores recommended
- Storage: 100GB default (expandable)

### Running Agent Flywheel Setup

Lima VMs use your macOS username by default, but ACFS expects an `ubuntu` user. The Ubuntu auto-upgrade also doesn't work in Lima VMs, so we use `--skip-ubuntu-upgrade`.

**1. Create the ubuntu user:**
```bash
limactl shell ubuntu24 sudo bash -c '
useradd -m -s /bin/bash ubuntu
echo "ubuntu ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/ubuntu
chown -R ubuntu:ubuntu /home/ubuntu
'
```

**2. Configure Lima to use ubuntu as default user:**
```bash
limactl stop ubuntu24
```

Edit `~/.lima/ubuntu24/lima.yaml` and add after the first line:
```yaml
user:
  name: ubuntu
```

Then restart:
```bash
limactl start ubuntu24
```

**3. Run the installer:**
```bash
limactl shell ubuntu24 -- bash -c 'curl -fsSL "https://raw.githubusercontent.com/Dicklesworthstone/agentic_coding_flywheel_setup/main/install.sh" | bash -s -- --mode vibe --yes --skip-ubuntu-upgrade'
```

Installation takes approximately 13 minutes.

**4. Access the VM and verify:**
```bash
limactl shell ubuntu24
acfs doctor
onboard
```

## UTM Virtual Machine

UTM provides full virtualization on macOS with a user-friendly interface for Apple Silicon Macs.

### What is UTM?

UTM is designed to give users the flexibility of QEMU without the steep learning curve. It can use Apple Virtualization framework for optimal performance on Apple Silicon.

### Installation

**Option 1: Homebrew (recommended):**
```bash
brew install --cask utm
```

**Option 2: Direct download:**
- [UTM for Mac](https://mac.getutm.app/) (Free and open source)
- Also available on Mac App Store

### Setup

**1. Download Ubuntu ARM64 ISO:**
```bash
# Direct download link:
wget -O ~/Downloads/ubuntu-24.04.3-live-server-arm64.iso \
  "https://cdimage.ubuntu.com/releases/24.04/release/ubuntu-24.04.3-live-server-arm64.iso"
```
Or download from: [Ubuntu 24.04 ARM64](https://ubuntu.com/download/server/arm)

**2. Create New VM:**

1. Open UTM and click **"+"** to create a new VM
2. Select **"Virtualize"** (not Emulate - uses Apple Silicon natively)
3. Select **"Linux"**
4. On the Linux configuration screen:
   - Leave **"Use Apple Virtualization"** unchecked (QEMU is recommended)
   - Select **"Boot from ISO image"**
   - Click **"Browse..."** and select the Ubuntu ISO from Downloads
   - Click **"Continue"**
5. On the Hardware screen:
   - **Memory**: 32768 MiB (32GB) - drag slider or type value
   - **CPU Cores**: 8
   - **Enable display output**: checked
   - **OpenGL acceleration**: leave unchecked (driver issues)
   - Click **"Continue"**
6. On the Storage screen:
   - Set disk size to **100 GB** or more
   - Click **"Continue"**
7. On the Shared Directory screen:
   - Optionally configure a shared folder (can skip)
   - Click **"Continue"**
8. On the Summary screen:
   - Give the VM a name (e.g., "Ubuntu-ACFS")
   - Click **"Save"**

**3. Install Ubuntu:**

1. Select your new VM and click the **Play** button to start
2. Follow the Ubuntu Server installer:
   - Select language and keyboard
   - Choose "Ubuntu Server" (not minimized)
   - Configure network (DHCP is fine)
   - Skip proxy and mirror configuration
   - Use entire disk for storage
   - Create user **"ubuntu"** (to match ACFS expectations)
   - Enable OpenSSH server when prompted
   - Skip optional snaps
3. Complete installation and reboot
4. When prompted, press Enter to remove the installation medium


### Post-Installation Configuration

**1. Get VM IP address** (from UTM console):
```bash
ip addr show enp0s1 | grep inet
```

**2. SSH from macOS host:**
```bash
ssh ubuntu@<vm-ip-address>
```

**3. Install guest tools (optional, for clipboard sharing):**
```bash
sudo apt update
sudo apt install -y spice-vdagent spice-webdavd
```

### Running Agent Flywheel Setup

Once your UTM VM is running and you can SSH in:

```bash
# SSH into the VM
ssh ubuntu@<vm-ip-address>

# Run the installer (as ubuntu user)
curl -fsSL "https://raw.githubusercontent.com/Dicklesworthstone/agentic_coding_flywheel_setup/main/install.sh" | bash -s -- --mode vibe --yes
```

Installation takes approximately 15-25 minutes. After completion:
```bash
acfs doctor
onboard
```

### Performance Tips

- Leave "Use Apple Virtualization" unchecked (QEMU is more stable for Linux)
- Allocate 32GB+ RAM for comfortable ACFS usage
- UTM uses virtio drivers by default for best performance

## Comparison: Lima vs UTM

| Feature | Lima | UTM |
|---------|------|-----|
| **Ease of Use** | Command-line, minimal setup | GUI-based, visual setup |
| **Performance** | Native virtualization, lightweight | Full virtualization with GUI |
| **File Sharing** | Automatic home directory mount | Manual setup required |
| **Network Access** | Automatic port forwarding | Manual configuration |
| **Resource Usage** | Lower overhead | Higher overhead with GUI |
| **Use Case** | Development, CLI workflows | Desktop environments, GUI apps |

## Recommendations

**Choose Lima if:**
- You prefer command-line workflows
- Want minimal overhead
- Need seamless file sharing
- Primarily use terminal/SSH access

**Choose UTM if:**
- You prefer GUI setup and management
- Need full desktop environment
- Want more control over VM configuration
- Need to run GUI applications

**Choose Cloud VPS if:**
- You need consistent 24/7 availability
- Want to share access with team members
- Require more resources than local machine provides
- Need production-like environment

For full setup requirements, see the main [agent-flywheel.com](https://agent-flywheel.com) documentation.