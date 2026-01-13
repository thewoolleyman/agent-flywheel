# Hosting Options for Agent Flywheel

This document covers various hosting options for running agent-flywheel, from cloud VPS providers to local virtualization on macOS.

## Cloud VPS Providers

### Contabo VPS

Recommended cloud VPS provider for agent-flywheel setup:

- [Contabo VPS Plans](https://contabo.com/en-us/vps/)
- [Contabo Server Configuration](https://new.contabo.com/servers/vps)

**Recommended specs:**
- 64GB RAM minimum
- Ubuntu 24.04 LTS or 22.04 LTS
- ~$40-56/month

## Local Virtualization on macOS

For development or testing purposes, you can run agent-flywheel locally on macOS using virtualization.

## Lima VM (Linux Virtual Machines)

Lima launches Linux virtual machines with automatic file sharing and port forwarding, similar to WSL2 on Windows.

### What is Lima?

Lima (LInux MAchine) provides the simplest way to run Linux VMs on macOS with seamless file sharing and port forwarding. It uses Apple's native virtualization framework for optimal performance.

### Installation

**Using Homebrew (Recommended):**
```bash
brew install lima
```

**Using MacPorts:**
```bash
sudo port install lima
```

**Manual Installation:**
Download from [Lima releases](https://github.com/lima-vm/lima/releases) and extract under `/usr/local`.

### Quick Setup

**Create Ubuntu VM:**
```bash
limactl create --name=ubuntu24 template://ubuntu-lts
```

**Start and access the VM:**
```bash
limactl start ubuntu24
limactl shell ubuntu24
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

### Available Templates

Lima supports multiple Linux distributions:
- Ubuntu (LTS and latest)
- Alpine
- Debian
- Fedora
- Arch Linux
- And more

### Resource Configuration

**Configure VM resources during creation:**
```bash
limactl create --name=agent-flywheel --memory=64GiB --cpus=8 template://ubuntu-lts
```

**For agent-flywheel requirements:**
- Memory: 64GB (modify template or use custom YAML)
- CPUs: 8+ cores recommended
- Storage: 200GB+ recommended

### Running Agent Flywheel Setup

Once your Lima VM is running:

1. Access the VM:
   ```bash
   limactl shell ubuntu24
   ```

2. Run the agent-flywheel installer:
   ```bash
   curl --proto '=https' --proto-redir '=https' -fsSL 'https://raw.githubusercontent.com/Dicklesworthstone/agentic_coding_flywheel_setup/main/install.sh' | bash -s -- --mode vibe --yes
   ```

## UTM Virtual Machine

UTM provides full virtualization on macOS with a user-friendly interface, supporting both Intel and Apple Silicon Macs.

### What is UTM?

UTM is designed to give users the flexibility of QEMU without the steep learning curve. It can use Apple Virtualization framework for optimal performance on Apple Silicon.

### Installation

**Download UTM:**
- [UTM for Mac](https://mac.getutm.app/) (Free and open source)
- Also available on Mac App Store (identical to free version)

### Setup for Apple Silicon (M1/M2/M3) Macs

**1. Download Ubuntu ARM64 ISO:**
- [Ubuntu 24.04 ARM64](https://ubuntu.com/download/server/arm)
- [Ubuntu 22.04 ARM64](https://ubuntu.com/download/server/arm)

**2. Create New VM:**
1. Open UTM and click "+" to open VM creation wizard
2. Select "Virtualize"
3. Select "Linux"
4. Choose hardware configuration:
   - **Memory**: 64GB (65536 MB) for agent-flywheel
   - **CPU Cores**: 8+ cores recommended
   - **Storage**: 200GB+ recommended

**3. Operating System Setup:**
1. Click "Browse" and select the Ubuntu Server ISO
2. Complete the VM creation wizard
3. Start the VM and follow Ubuntu installation

**4. Install Desktop Environment (Optional):**
After Ubuntu Server installation:
```bash
sudo apt update
sudo apt install ubuntu-desktop
sudo reboot
```

### Setup for Intel Macs

**1. Download Ubuntu x86_64 ISO:**
- [Ubuntu 24.04 Desktop](https://ubuntu.com/download/desktop)
- [Ubuntu 22.04 Desktop](https://ubuntu.com/download/desktop)

**2. Follow similar VM creation steps** as Apple Silicon, but select x86_64 architecture.

### Post-Installation Configuration

**1. Install UTM Guest Tools (Recommended):**
```bash
sudo apt update
sudo apt install spice-vdagent spice-webdavd
```

**2. Configure SSH Access:**
```bash
sudo apt install openssh-server
sudo systemctl enable ssh
sudo systemctl start ssh
```

**3. Get VM IP address:**
```bash
ip addr show
```

**4. SSH from macOS host:**
```bash
ssh username@vm-ip-address
```

### Running Agent Flywheel Setup

Once your UTM VM is running Ubuntu:

1. SSH into the VM or use the UTM console
2. Run the agent-flywheel installer:
   ```bash
   curl --proto '=https' --proto-redir '=https' -fsSL 'https://raw.githubusercontent.com/Dicklesworthstone/agentic_coding_flywheel_setup/main/install.sh' | bash -s -- --mode vibe --yes
   ```

### Performance Tips

**For Apple Silicon Macs:**
- Enable "Use Apple Virtualization" in VM settings
- Allocate adequate RAM (64GB for full agent-flywheel experience)
- Use virtio drivers for best performance

**For Intel Macs:**
- Enable hardware acceleration
- Allocate sufficient CPU cores
- Use SSD storage for VM disk

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

## Cost Considerations

- **Lima/UTM**: Free software, uses local hardware resources
- **Cloud VPS**: ~$40-56/month for recommended specs
- **Total agent-flywheel cost**: Add ~$400-600/month for AI subscriptions (Claude Max, ChatGPT Pro, etc.)

For full cost breakdown and setup requirements, see the main [agent-flywheel.com](https://agent-flywheel.com) documentation.