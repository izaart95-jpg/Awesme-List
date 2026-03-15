# Linux Kernel Documentation & Computer Architecture

## 1. What is the Kernel?

The **kernel** is the core component of an operating system that acts as a bridge between hardware and software. It has complete control over everything in the system.

### Primary Responsibilities:
- **Process Management**: Creating, scheduling, and terminating processes
- **Memory Management**: Allocating and deallocating memory space
- **Device Management**: Controlling hardware devices via drivers
- **File System Management**: Managing file operations and storage
- **Security & Access Control**: Enforcing permissions and isolation
- **System Calls**: Providing interface for applications to request services

---

## 2. Computer Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────┐
│                            USER SPACE                                    │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐   │
│  │  Application│  │  Application│  │  Application│  │    Shell    │   │
│  │   (Firefox) │  │   (VS Code) │  │  (Terminal) │  │   (Bash)    │   │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘  └────────────┘   │
│         │                │                │                │           │
│         └────────────────┴───────────────┴────────────────┘           │
│                                    │                                    │
│                          ┌────────▼────────┐                           │
│                          │  System Call    │                           │
│                          │    Interface    │                           │
│                          │   (glibc/libc)  │                           │
│                          └────────┬────────┘                           │
└───────────────────────────────────┼─────────────────────────────────────┘
                                    │ Context Switch
                                    ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                          KERNEL SPACE                                    │
│  ┌───────────────────────────────────────────────────────────────────┐ │
│  │                        SYSTEM CALL INTERFACE                       │ │
│  │  (open, read, write, close, fork, exec, mmap, socket, ioctl...)   │ │
│  └─────────────────────────────┬─────────────────────────────────────┘ │
│                                │                                        │
│         ┌──────────────────────┼──────────────────────┐                │
│         ▼                      ▼                      ▼                │
│  ┌─────────────┐       ┌─────────────┐       ┌─────────────┐          │
│  │   PROCESS   │       │   MEMORY    │       │   VIRTUAL   │          │
│  │ SCHEDULER   │       │  MANAGER    │       │ FILE SYSTEM │          │
│  │             │       │             │       │             │          │
│  │ • Scheduling│       │ • Allocation│       │ • ext4/xfs  │          │
│  │ • Context   │       │ • Paging    │       │ • btrfs     │          │
│  │   Switching │       │ • Swap      │       │ • VFS Layer │          │
│  └─────────────┘       └─────────────┘       └─────────────┘          │
│                                │                                        │
│         ┌──────────────────────┼──────────────────────┐                │
│         ▼                      ▼                      ▼                │
│  ┌─────────────┐       ┌─────────────┐       ┌─────────────┐          │
│  │   NETWORK   │       │   DEVICE    │       │  SECURITY   │          │
│  │   STACK     │       │  DRIVERS    │       │   MODULES   │          │
│  │             │       │             │       │             │          │
│  │ • TCP/IP    │       │ • Character │       │ • SELinux   │          │
│  │ • UDP       │       │ • Block     │       │ • AppArmor  │          │
│  │ • Sockets   │       │ • Network   │       │ • Capabilities│        │
│  └─────────────┘       └─────────────┘       └─────────────┘          │
│                                │                                        │
│                                ▼                                        │
│  ┌─────────────────────────────────────────────────────────────────┐  │
│  │                    HARDWARE ABSTRACTION LAYER                    │  │
│  │  (Architecture-specific code: x86_64, ARM, RISC-V, etc.)        │  │
│  └─────────────────────────────┬───────────────────────────────────┘  │
└────────────────────────────────┼───────────────────────────────────────┘
                                 │ Hardware Instructions
                                 ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                            HARDWARE                                      │
│  ┌──────────┐  ┌──────────┐  ┌──────────  ┌──────────┐  ┌──────────┐ │
│  │   CPU    │  │   RAM    │  │   GPU    │  │  Storage │  │ Network  │ │
│  │ (Cores/  │  │ (DDR4/   │  │ (Graphics│  │  (SSD/   │  │   Card   │ │
│  │  Cache)  │  │  DDR5)   │  │  Card)   │  │   HDD)   │  │ (NIC)    │ │
│  └──────────┘  └──────────  └──────────┘  └──────────┘  └──────────┘ │
│                                                                          │
│  ┌──────────────────────────────────────────────────────────────────┐  │
│  │                         BUS SYSTEM                                │  │
│  │         (PCIe, USB, SATA, I²C, SPI, Memory Bus)                  │  │
│  └──────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Kernel Architecture

### Monolithic Kernel Design (Linux)
Linux uses a **monolithic kernel** architecture where most OS services run in kernel space:

```
┌─────────────────────────────────────────┐
│           USER APPLICATIONS             │
└─────────────────┬───────────────────────┘
                  │ System Calls
┌─────────────────▼───────────────────────┐
│              KERNEL SPACE               │
│  ┌───────────────────────────────────┐ │
│  │      System Call Interface        │ │
│  └───────────────┬───────────────────┘ │
│  ┌───────────────▼───────────────────┐ │
│  │  Core Kernel Services:            │ │
│  │  • Process Scheduler              │ │
│  │  • Memory Manager                 │ │
│  │  • Virtual File System            │ │
│  │  • Network Stack                  │ │
│  │  • Device Drivers                 │ │
│  │  • Inter-Process Communication    │ │
│  └───────────────────────────────────┘ │
└─────────────────────────────────────────┘
```

### Kernel Space vs User Space

| Aspect | Kernel Space | User Space |
|--------|-------------|------------|
| **Privilege** | Ring 0 (highest) | Ring 3 (lowest) |
| **Memory Access** | Full system memory | Isolated virtual memory |
| **Hardware Access** | Direct | Via system calls |
| **Crash Impact** | System panic/crash | Process termination |
| **Performance** | Fast, no context switch | Slower, requires context switch |

---

## 4. Common Kernel Usage & Management

### Checking Kernel Information

```bash
# Current kernel version
uname -r
uname -a  # All information

# Kernel build information
cat /proc/version

# Loaded kernel modules
lsmod

# Kernel configuration (if enabled)
zcat /proc/config.gz | grep CONFIG_<OPTION>

# System uptime and load
uptime
cat /proc/uptime
```

### Kernel Modules Management

```bash
# List all loaded modules
lsmod

# Load a module
sudo modprobe <module_name>

# Remove a module
sudo modprobe -r <module_name>

# Check module dependencies
modinfo <module_name>

# List available modules for current kernel
find /lib/modules/$(uname -r) -name "*.ko"

# Blacklist a module (prevent loading)
echo "blacklist <module_name>" | sudo tee -a /etc/modprobe.d/blacklist.conf

# Auto-load module at boot
echo "<module_name>" | sudo tee -a /etc/modules-load.d/<module_name>.conf
```

### Common Kernel Modules

```bash
# Network drivers
lsmod | grep -E 'e1000|igb|ixgbe|iwlwifi|ath'

# Filesystem modules
lsmod | grep -E 'ext4|xfs|btrfs|vfat|ntfs'

# Virtualization
lsmod | grep -E 'kvm|kvm_intel|kvm_amd|vboxdrv'

# Graphics
lsmod | grep -E 'i915|nvidia|amdgpu|radeon'

# USB modules
lsmod | grep -E 'usb_storage|xhci_hcd|ehci_hcd'
```

---

## 5. System Calls

### What are System Calls?
System calls are the interface between user-space applications and the kernel.

### Common System Calls

| Category | System Calls |
|----------|-------------|
| **Process Control** | `fork()`, `exec()`, `exit()`, `wait()`, `getpid()` |
| **File Operations** | `open()`, `read()`, `write()`, `close()`, `lseek()` |
| **Directory Ops** | `mkdir()`, `rmdir()`, `chdir()`, `getcwd()` |
| **Device Ops** | `ioctl()`, `read()`, `write()` |
| **Information** | `getpid()`, `getuid()`, `time()`, `sysinfo()` |
| **Communication** | `pipe()`, `shmget()`, `mmap()`, `socket()` |
| **Protection** | `chmod()`, `chown()`, `umask()` |

### Tracing System Calls

```bash
# Trace system calls of a process
strace -p <PID>

# Trace specific system calls
strace -e trace=open,read,write <command>

# Count system calls
strace -c <command>

# View system call statistics for running process
cat /proc/<PID>/syscall
```

### Example: strace output
```bash
$ strace ls /tmp
execve("/bin/ls", ["ls", "/tmp"], 0x7ffd12345678) = 0
brk(NULL)                               = 0x555555777000
openat(AT_FDCWD, "/tmp", O_RDONLY|O_NONBLOCK|O_CLOEXEC|O_DIRECTORY) = 3
fstat(3, {st_mode=S_IFDIR|S_ISVTX|0777, st_size=4096, ...}) = 0
getdents64(3, /* 5 entries */, 32768)   = 160
write(1, "file1.txt\nfile2.log\n", 20)  = 20
close(3)                                = 0
```

---

## 6. Kernel Parameters & Tuning

### Viewing and Modifying Parameters

```bash
# View all kernel parameters
sysctl -a

# View specific parameter
sysctl kernel.hostname
cat /proc/sys/kernel/hostname

# Temporarily change parameter
sudo sysctl -w vm.swappiness=10

# Permanently change parameter
echo "vm.swappiness=10" | sudo tee -a /etc/sysctl.conf

# Reload sysctl configuration
sudo sysctl -p

# Common tuning parameters location
/proc/sys/
├── kernel/      # Core kernel parameters
├── vm/          # Virtual memory settings
├── net/         # Network stack parameters
├── fs/          # Filesystem settings
└── dev/         # Device-specific settings
```

### Important Kernel Parameters

```bash
# Memory Management
vm.swappiness              # Swap usage tendency (0-100, default 60)
vm.dirty_ratio             # Max dirty pages before blocking writes (%)
vm.dirty_background_ratio  # Background writeback threshold (%)
vm.overcommit_memory       # Memory overcommit policy (0,1,2)

# Network Tuning
net.core.somaxconn         # Max listen queue backlog
net.ipv4.tcp_max_syn_backlog  # SYN backlog size
net.ipv4.ip_forward        # IP forwarding (0=off, 1=on)

# Security
kernel.randomize_va_space  # ASLR (0=off, 1=partial, 2=full)
kernel.dmesg_restrict      # Restrict dmesg access (0,1)
kernel.kptr_restrict       # Kernel pointer restriction (0,1,2)

# Performance
kernel.sched_min_granularity_ns  # Scheduler minimum granularity
vm.max_map_count           # Max memory map areas
```

### Example: High-Performance Web Server Tuning

```bash
# /etc/sysctl.d/99-webserver.conf
# Increase system file descriptor limit
fs.file-max = 2097152

# Increase system IP port limits
net.ipv4.ip_local_port_range = 1024 65535

# Increase TCP max buffer size
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216

# Increase Linux autotuning TCP buffer limits
net.ipv4.tcp_rmem = 4096 87380 16777216
net.ipv4.tcp_wmem = 4096 87380 16777216

# Enable TCP window scaling
net.ipv4.tcp_window_scaling = 1

# Increase connection queue
net.core.somaxconn = 65535
net.ipv4.tcp_max_syn_backlog = 8192

# Apply changes
sudo sysctl -p /etc/sysctl.d/99-webserver.conf
```

---

## 7. Kernel Compilation & Building

### Prerequisites

```bash
# Install build dependencies (Debian/Ubuntu)
sudo apt install build-essential libncurses-dev bison flex libssl-dev libelf-dev

# Install build dependencies (RHEL/Fedora)
sudo dnf install gcc make ncurses-devel bison flex openssl-devel elfutils-libelf-devel
```

### Building Kernel from Source

```bash
# 1. Download kernel source
cd /usr/src
sudo wget https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.x.tar.xz
sudo tar -xf linux-6.x.tar.xz
cd linux-6.x

# 2. Get current config (recommended)
cp /boot/config-$(uname -r) .config

# 3. Update config for new version
make olddefconfig

# OR: Interactive configuration
make menuconfig

# 4. Compile kernel (using all CPU cores)
make -j$(nproc)

# 5. Compile modules
make modules

# 6. Install modules
sudo make modules_install

# 7. Install kernel
sudo make install

# 8. Update bootloader (GRUB)
sudo update-grub  # Debian/Ubuntu
sudo grub2-mkconfig -o /boot/grub2/grub.cfg  # RHEL/Fedora

# 9. Reboot and select new kernel
sudo reboot
```

### Kernel Configuration Options

```bash
# View current configuration
zcat /proc/config.gz > current_config
grep "CONFIG_" current_config | head -20

# Common configuration options
CONFIG_SMP=y                    # Symmetric Multi-Processing
CONFIG_PREEMPT=y                # Preemptible kernel
CONFIG_HZ=1000                  # Timer frequency
CONFIG_MODULES=y                # Loadable module support
CONFIG_NET=y                    # Networking support
CONFIG_EXT4_FS=y                # EXT4 filesystem
CONFIG_USB_SUPPORT=y            # USB support
CONFIG_PCI=y                    # PCI support
```

---

## 8. Kernel Debugging & Troubleshooting

### Kernel Messages & Logs

```bash
# View kernel ring buffer
dmesg

# View kernel messages in real-time
dmesg -w

# Filter by facility/level
dmesg --level=err,warn
dmesg --facility=kern

# Search for specific messages
dmesg | grep -i "usb"
dmesg | grep -i "error"

# Kernel logs via journalctl
journalctl -k
journalctl -k --since "1 hour ago"
journalctl -k -p err

# Persistent kernel logs
cat /var/log/kern.log  # Debian/Ubuntu
cat /var/log/messages  # RHEL/CentOS
```

### Common Kernel Issues

```bash
# Check for hardware errors
dmesg | grep -i "hardware error"
dmesg | grep -i "mce"  # Machine Check Exceptions

# Check for OOM (Out of Memory) killer
dmesg | grep -i "killed process"
grep -i "oom" /var/log/syslog

# Check for disk errors
dmesg | grep -i "I/O error"
dmesg | grep -i "ext4.*error"

# Check for kernel panics
cat /var/crash/*
journalctl -b -1 -p err  # Previous boot errors
```

### Kernel Crash Analysis

```bash
# Install crash analysis tools
sudo apt install crash kdump-tools  # Debian/Ubuntu
sudo dnf install crash kexec-tools  # RHEL/Fedora

# Configure kdump
sudo systemctl enable kdump
sudo systemctl start kdump

# Analyze crash dump
crash /usr/lib/debug/lib/modules/$(uname -r)/vmlinux /var/crash/*/vmcore

# View crash dump info
crash> sys
crash> ps
crash> bt  # Backtrace
```

### Performance Monitoring

```bash
# Install performance tools
sudo apt install linux-tools-generic linux-tools-$(uname -r)
sudo dnf install perf

# CPU profiling
perf top
perf record -g <command>
perf report

# Memory profiling
cat /proc/meminfo
free -h
vmstat 1

# I/O statistics
iostat -x 1
iotop

# Network statistics
netstat -s
ss -s
```

---

## 9. Kernel Security Features

### Built-in Security Mechanisms

```bash
# Check ASLR (Address Space Layout Randomization)
cat /proc/sys/kernel/randomize_va_space
# 0 = Disabled, 1 = Partial, 2 = Full

# Check kernel pointer restrictions
cat /proc/sys/kernel/kptr_restrict
# 0 = Visible, 1 = Hidden from unprivileged, 2 = Always hidden

# Check dmesg restrictions
cat /proc/sys/kernel/dmesg_restrict
# 0 = Visible, 1 = Restricted to CAP_SYS_ADMIN

# Enable secure boot verification
cat /sys/kernel/security/secureboot

# Check for lockdown mode
cat /sys/kernel/security/lockdown
```

### Kernel Hardening

```bash
# Enable lockdown mode (prevent root from modifying kernel)
# Add to kernel boot parameters:
lockdown=confidentiality

# Disable module loading (for maximum security)
echo 1 | sudo tee /proc/sys/kernel/modules_disabled

# Restrict ptrace (prevent process tracing)
echo "kernel.yama.ptrace_scope = 2" | sudo tee -a /etc/sysctl.d/60-kernel-hardening.conf

# Enable strict reverse path filtering
echo "net.ipv4.conf.all.rp_filter = 1" | sudo tee -a /etc/sysctl.d/60-kernel-hardening.conf

# Disable IP forwarding (if not needed)
echo "net.ipv4.ip_forward = 0" | sudo tee -a /etc/sysctl.d/60-kernel-hardening.conf
```

---

## 10. Kernel Versioning & Lifecycle

### Version Number Format

```
Major.Minor.Patch-Extra
Example: 6.5.11-200.fc38.x86_64

Where:
• 6      = Major version
• 5      = Minor version
• 11     = Patch level
• 200    = Distribution-specific build number
• fc38   = Fedora 38
• x86_64 = Architecture
```

### Kernel Release Cycle

```
┌─────────────────────────────────────────────────────────┐
│                    MAINLINE KERNEL                      │
│  • Released every 9-10 weeks                            │
│  • Latest features and drivers                          │
│  • Maintained by Linus Torvalds                         │
└────────────────────┬────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────┐
│                      STABLE KERNEL                      │
│  • Bug fixes and security patches                       │
│  • Maintained by Greg Kroah-Hartman                     │
│  • Multiple versions maintained simultaneously          │
└────────────────────┬────────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────────┐
│                    LONGTERM (LTS)                       │
│  • Maintained for 2-6+ years                            │
│  • Critical fixes only                                  │
│  • Used by distributions (RHEL, Ubuntu LTS, etc.)       │
│  • Current LTS: 5.10, 5.15, 6.1, 6.6                   │
└─────────────────────────────────────────────────────────┘
```

### Checking Kernel Support Status

```bash
# Current kernel version
uname -r

# Check if running latest stable
curl -s https://www.kernel.org | grep -A1 "latest stable"

# View kernel end-of-life dates
# Visit: https://endoflife.date/linux

# Check distribution kernel support
# RHEL: https://access.redhat.com/articles/rhel-limits
# Ubuntu: https://ubuntu.com/about/release-cycle
```

---

## 11. Quick Reference Commands

```bash
# ============ INFORMATION ============
uname -a                    # All kernel info
cat /proc/version           # Kernel version string
hostnamectl                 # System and kernel info
lsmod                       # List loaded modules
modinfo <module>            # Module information

# ============ MODULES ============
modprobe <module>           # Load module
modprobe -r <module>        # Remove module
lsmod | grep <pattern>      # Search modules
depmod -a                   # Update module dependencies

# ============ PARAMETERS ============
sysctl -a                   # All parameters
sysctl <param>              # Get parameter
sysctl -w <param>=<value>   # Set parameter (temp)
sysctl -p                   # Reload config

# ============ LOGS & DEBUG ============
dmesg                       # Kernel messages
dmesg -w                    # Real-time messages
journalctl -k               # Kernel logs (systemd)
strace <command>            # Trace syscalls
perf top                    # Performance monitoring

# ============ MEMORY & CPU ============
cat /proc/meminfo           # Memory info
cat /proc/cpuinfo           # CPU info
free -h                     # Human-readable memory
vmstat 1                    # Virtual memory stats

# ============ HARDWARE ============
lspci                       # PCI devices
lsusb                       # USB devices
lscpu                       # CPU architecture
lsblk                       # Block devices
```

---

## 12. Advanced Topics

### eBPF (Extended Berkeley Packet Filter)

```bash
# Modern kernel tracing and monitoring
# Install tools
sudo apt install bpfcc-tools linux-tools-$(uname -r)

# List available BPF tools
ls /usr/share/bcc/tools/

# Examples
sudo /usr/share/bcc/tools/execsnoop    # Trace exec() calls
sudo /usr/share/bcc/tools/opensnoop    # Trace file opens
sudo /usr/share/bcc/tools/tcpconnect   # Trace TCP connections
```

### Namespaces (Container Foundation)

```bash
# View namespaces
ls -la /proc/self/ns/

# Namespace types:
# • pid    - Process isolation
# • net    - Network isolation
# • mnt    - Mount points
# • uts    - Hostname isolation
# • ipc    - IPC isolation
# • user   - User/group isolation

# Create namespace
unshare --pid --fork --mount-proc /bin/bash
```

### Control Groups (cgroups)

```bash
# Resource limiting and accounting
# View cgroups
cat /proc/<pid>/cgroup

# Modern cgroups v2
mount | grep cgroup2
ls /sys/fs/cgroup/

# Limit memory for a process
echo 512M > /sys/fs/cgroup/memory.limit_in_bytes
echo <pid> > /sys/fs/cgroup/cgroup.procs
```

---

## 13. Best Practices

✅ **Security**
- Keep kernel updated with security patches
- Enable automatic security updates
- Use signed kernel modules
- Enable SELinux/AppArmor
- Disable unnecessary kernel modules

✅ **Performance**
- Tune kernel parameters for workload
- Use appropriate I/O scheduler
- Monitor system resources regularly
- Use performance profiling tools

✅ **Stability**
- Test new kernels in staging first
- Keep previous kernel for rollback
- Monitor kernel logs for errors
- Use LTS kernels for production

✅ **Maintenance**
- Document custom kernel parameters
- Backup kernel configurations
- Remove old unused kernels
- Track kernel version changes

---

## 14. Resources

### Official Documentation
- [Kernel.org](https://www.kernel.org/)
- [Kernel Documentation](https://www.kernel.org/doc/html/latest/)
- [Kernel Newbies](https://kernelnewbies.org/)

### Books
- "Linux Kernel Development" by Robert Love
- "Understanding the Linux Kernel" by Bovet & Cesati
- "Linux Device Drivers" by Corbet, Rubini, Hartman

### Tools
- `perf` - Performance analysis
- `bpftrace` - Tracing with eBPF
- `crash` - Kernel crash analysis
- `kgdb` - Kernel debugger

---

> 💡 **Remember**: The kernel is the heart of your system. Always test changes in a non-production environment first, and maintain backups before making modifications!
