# Operations Deployment

> Reference: [Kodekloud public notes](https://notes.kodekloud.com/docs/Linux-Foundation-Certified-System-Administrator-LFCS/Introduction/Course-Introduction)

## Boot, Reboot, and Shutdown

**Basic Commands:**
```bash
# Reboot
systemctl reboot
sudo systemctl reboot --force       # Force reboot (use with caution)

# Shutdown
sudo systemctl poweroff
sudo systemctl poweroff --force     # Force shutdown (use with caution)

# Scheduled shutdown/reboot
sudo shutdown 02:00                 # Shutdown at 2 AM
sudo shutdown +15                   # Shutdown in 15 minutes
sudo shutdown -r +15                # Reboot in 15 minutes
sudo shutdown -r +1 'Scheduled restart for kernel upgrade'  # With message
```

**Important Notes:**
- Using `--force` twice acts like pressing the physical reset button (data loss risk)
- Wall messages notify all logged-in users about scheduled maintenance

## Boot Targets (Operating Modes)

**Check and Set Default Target:**
```bash
systemctl get-default                              # Check current default
sudo systemctl set-default multi-user.target       # Text-based boot
sudo systemctl set-default graphical.target        # GUI boot
sudo systemctl isolate graphical.target            # Switch to GUI immediately
```

**Common Targets:**
| Target              | Description                                    | Use Case                           |
|---------------------|------------------------------------------------|------------------------------------|
| graphical.target    | Full graphical desktop environment             | Desktop usage                      |
| multi-user.target   | Text-based with network services               | Servers, low-resource systems      |
| rescue.target       | Essential services with root shell             | Administrative tasks               |
| emergency.target    | Minimal services, root FS read-only            | Critical troubleshooting           |

**Warning:** Ensure root password is set before using rescue/emergency targets

## Bash Scripting for System Maintenance

**Script Structure:**
```bash
#!/bin/bash
# Script description (always start with shebang and comments)

# Example: Log system info
date >> /tmp/script.log
cat /proc/version >> /tmp/script.log

# Make executable
chmod +x script.sh
./script.sh
```

**Conditional Statements:**
```bash
#!/bin/bash
# Check if file exists before archiving
if [ -f /tmp/archive.tar.gz ]; then
    mv /tmp/archive.tar.gz /tmp/archive.tar.gz.OLD
fi
tar -czf /tmp/archive.tar.gz /etc/apt/

# Using grep exit status
if grep -q '5' /etc/default/grub; then
    echo 'Grub has a timeout of 5 seconds.'
else
    echo 'Grub DOES NOT have a timeout of 5 seconds.'
fi
```

**Exit Status:** 0 = success, non-zero = error/condition not met

## systemd Service Management

**Essential Commands:**
```bash
systemctl status service-name          # Check service status and logs
systemctl start service-name           # Start service
systemctl stop service-name            # Stop service
systemctl restart service-name         # Restart service
systemctl reload service-name          # Reload config without stopping
systemctl enable service-name          # Enable at boot
systemctl disable service-name         # Disable at boot
systemctl is-active service-name       # Quick status check
systemctl is-enabled service-name      # Check if enabled at boot
systemctl list-units --type=service    # List all services
systemctl daemon-reload                # Reload systemd after unit file changes
```

**Unit File Locations:**
- `/lib/systemd/system/` - System default unit files
- `/etc/systemd/system/` - Custom/modified unit files (takes precedence)

## Creating systemd Services

**Basic Service Unit File** (`/etc/systemd/system/myservice.service`):
```ini
[Unit]
Description=My Custom Service
After=network.target                    # Start after network is up

[Service]
Type=simple                             # Service type (simple, forking, oneshot)
ExecStart=/usr/bin/my-application       # Command to start
ExecStop=/usr/bin/kill-my-application   # Command to stop (optional)
Restart=on-failure                      # Restart policy
RestartSec=5                            # Wait before restart
User=myuser                             # Run as specific user
WorkingDirectory=/opt/myapp             # Working directory

[Install]
WantedBy=multi-user.target              # Target to enable service in
```

**After creating/modifying:**
```bash
sudo systemctl daemon-reload
sudo systemctl enable myservice.service
sudo systemctl start myservice.service
```

## Process Management

**Viewing Processes:**
```bash
ps aux                    # All processes
ps -ef                    # Full format listing
top                       # Interactive process viewer
htop                      # Enhanced interactive viewer (if installed)
pstree                    # Process tree
pgrep process-name        # Find PID by name
pidof process-name        # Get PID of running process
```

**Managing Processes:**
```bash
kill PID                  # Terminate process (SIGTERM)
kill -9 PID               # Force kill (SIGKILL)
kill -15 PID              # Graceful termination (SIGTERM)
killall process-name      # Kill all processes by name
pkill process-name        # Kill processes by name pattern
nice -n 10 command        # Start with lower priority
renice -n 5 -p PID        # Change priority of running process
```

**Background/Foreground:**
```bash
command &                 # Run in background
jobs                      # List background jobs
fg %1                     # Bring job 1 to foreground
bg %1                     # Resume job 1 in background
Ctrl+Z                    # Suspend current process
```

## System Logs

**journalctl (systemd logs):**
```bash
journalctl                                    # All logs
journalctl -u service-name                    # Logs for specific service
journalctl -f                                 # Follow logs (like tail -f)
journalctl -p err                             # Only errors
journalctl -b                                 # Logs since last boot
journalctl --since "2024-01-01"               # Logs since date
journalctl --since "1 hour ago"               # Recent logs
journalctl -n 50                              # Last 50 lines
journalctl --disk-usage                       # Check log disk usage
journalctl --vacuum-size=100M                 # Clean logs to 100MB
```

**Traditional Log Files:**
```bash
/var/log/syslog          # System logs (Debian/Ubuntu)
/var/log/messages        # System logs (RHEL/CentOS)
/var/log/auth.log        # Authentication logs
/var/log/boot.log        # Boot logs
/var/log/dmesg           # Kernel ring buffer
/var/log/kern.log        # Kernel logs
```

**Viewing Logs:**
```bash
tail -f /var/log/syslog               # Follow logs in real-time
less /var/log/syslog                  # View with pagination
grep "error" /var/log/syslog          # Search for errors
dmesg | grep -i error                 # Kernel errors
```

## Task Scheduling

**Cron:**
```bash
crontab -e               # Edit user crontab
crontab -l               # List user crontab
crontab -r               # Remove user crontab
sudo crontab -e -u user  # Edit crontab for specific user

# Crontab format: MIN HOUR DOM MON DOW COMMAND
# * * * * * command
# | | | | |
# | | | | +-- Day of week (0-7, both 0 and 7 are Sunday)
# | | | +---- Month (1-12)
# | | +------ Day of month (1-31)
# | +-------- Hour (0-23)
# +---------- Minute (0-59)

# Examples:
0 2 * * * /path/to/backup.sh          # Daily at 2 AM
*/15 * * * * /path/to/check.sh        # Every 15 minutes
0 0 * * 0 /path/to/weekly.sh          # Weekly on Sunday at midnight
0 9 1 * * /path/to/monthly.sh         # Monthly on 1st at 9 AM
```

**System-wide cron directories:**
```bash
/etc/cron.daily/         # Daily jobs
/etc/cron.weekly/        # Weekly jobs
/etc/cron.monthly/       # Monthly jobs
/etc/cron.hourly/        # Hourly jobs
```

**at (one-time scheduled tasks):**
```bash
at 2:30 PM                           # Schedule task
at> /path/to/script.sh
at> Ctrl+D                           # Finish input

at now + 30 minutes                  # Schedule relative time
atq                                  # List scheduled at jobs
atrm job_number                      # Remove at job
```

**systemd Timers (modern alternative to cron):**

Timer file (`/etc/systemd/system/mybackup.timer`):
```ini
[Unit]
Description=Backup Timer

[Timer]
OnCalendar=daily                     # Or: *-*-* 02:00:00 (2 AM daily)
Persistent=true                      # Run missed jobs on next boot
RandomizedDelaySec=300               # Random delay up to 5 minutes

[Install]
WantedBy=timers.target
```

Service file (`/etc/systemd/system/mybackup.service`):
```ini
[Unit]
Description=Backup Service

[Service]
Type=oneshot
ExecStart=/usr/local/bin/backup.sh
```

**Manage timers:**
```bash
sudo systemctl daemon-reload
sudo systemctl enable mybackup.timer
sudo systemctl start mybackup.timer
systemctl list-timers               # List all timers
```

## Package Management

**APT (Debian/Ubuntu):**
```bash
apt update                          # Update package lists
apt upgrade                         # Upgrade packages
apt install package-name            # Install package
apt remove package-name             # Remove package
apt purge package-name              # Remove package and config
apt autoremove                      # Remove unused dependencies
apt search keyword                  # Search packages
apt show package-name               # Show package info
apt list --installed                # List installed packages
```

**YUM/DNF (RHEL/CentOS/Fedora):**
```bash
yum update                          # Update all packages
yum install package-name            # Install package
yum remove package-name             # Remove package
yum search keyword                  # Search packages
yum info package-name               # Package information
yum list installed                  # List installed packages
dnf update                          # DNF (newer than yum)
```

## Repository Configuration

**APT repositories** (`/etc/apt/sources.list` or `/etc/apt/sources.list.d/`):
```bash
# Add repository
sudo add-apt-repository ppa:user/ppa-name
sudo apt update

# Manual entry in sources.list
deb http://repo-url distribution component
```

**YUM repositories** (`/etc/yum.repos.d/`):
```bash
# Create repo file: /etc/yum.repos.d/myrepo.repo
[myrepo]
name=My Repository
baseurl=http://repo-url
enabled=1
gpgcheck=1
gpgkey=http://repo-url/RPM-GPG-KEY
```

## Compiling from Source

**Basic Steps:**
```bash
# 1. Install build tools
sudo apt install build-essential    # Debian/Ubuntu
sudo yum groupinstall "Development Tools"  # RHEL/CentOS

# 2. Extract source
tar -xzf package.tar.gz
cd package

# 3. Configure, compile, install
./configure --prefix=/usr/local     # Configure with options
make                                # Compile
make test                           # Optional: run tests
sudo make install                   # Install
```

**Common configure options:**
```bash
./configure --help                  # View all options
./configure --prefix=/opt/software  # Custom install location
./configure --enable-feature        # Enable specific feature
```

## System Resource Verification

**System Performance:**
```bash
uptime                              # System uptime and load average
free -h                             # Memory usage (human-readable)
df -h                               # Disk usage
du -sh /path                        # Directory size
iostat                              # CPU and I/O statistics
vmstat 1 5                          # Virtual memory stats (1s interval, 5 times)
lsof                                # List open files
lsof -i :80                         # Process using port 80
netstat -tulpn                      # Network connections
ss -tulpn                           # Socket statistics (modern netstat)
```

**System Information:**
```bash
uname -a                            # Kernel and system info
hostnamectl                         # Hostname and OS info
lscpu                               # CPU information
lsblk                               # Block devices
lspci                               # PCI devices
lsusb                               # USB devices
dmidecode                           # Hardware info from BIOS
```

## Kernel Runtime Parameters

**Temporary (non-persistent):**
```bash
sysctl parameter=value              # Set parameter
sysctl -a                           # List all parameters
sysctl kernel.hostname=newname      # Example: change hostname

# Examples:
sysctl net.ipv4.ip_forward=1        # Enable IP forwarding
echo 1 > /proc/sys/net/ipv4/ip_forward  # Alternative method
```

**Persistent:**
```bash
# Edit /etc/sysctl.conf or create file in /etc/sysctl.d/
sudo vi /etc/sysctl.d/99-custom.conf

# Add parameters:
net.ipv4.ip_forward = 1
vm.swappiness = 10
fs.file-max = 65536

# Apply changes:
sudo sysctl -p /etc/sysctl.d/99-custom.conf  # Specific file
sudo sysctl --system                          # All config files
```

**Common Parameters:**
```bash
net.ipv4.ip_forward              # IP forwarding
net.ipv6.conf.all.forwarding     # IPv6 forwarding
vm.swappiness                    # Swap usage tendency (0-100)
fs.file-max                      # Max open files system-wide
kernel.panic                     # Panic behavior
```

## SELinux

**Check SELinux Status:**
```bash
getenforce                          # Current mode (Enforcing/Permissive/Disabled)
sestatus                            # Detailed status
```

**SELinux Modes:**
```bash
setenforce 0                        # Set to Permissive (temporary)
setenforce 1                        # Set to Enforcing (temporary)

# Permanent: Edit /etc/selinux/config
SELINUX=enforcing|permissive|disabled
```

**File Contexts:**
```bash
ls -Z /path/to/file                 # View file context
ps -eZ                              # View process contexts
id -Z                               # View user context

# Set file context
chcon -t httpd_sys_content_t /var/www/html/file
restorecon -Rv /var/www/html/       # Restore default contexts

# Make persistent
semanage fcontext -a -t httpd_sys_content_t "/webfiles(/.*)?"
restorecon -Rv /webfiles/
```

**SELinux Booleans:**
```bash
getsebool -a                        # List all booleans
getsebool httpd_can_network_connect # Check specific boolean
setsebool httpd_can_network_connect on  # Temporary
setsebool -P httpd_can_network_connect on  # Persistent (-P)
```

**Troubleshooting SELinux:**
```bash
ausearch -m avc -ts recent          # Recent AVC denials
audit2why < /var/log/audit/audit.log  # Analyze denials
audit2allow -a                      # Generate allow rules
sealert -a /var/log/audit/audit.log # User-friendly analysis
```

## Container Management

**Podman (rootless, daemonless):**
```bash
podman run -d --name myapp -p 8080:80 nginx    # Run container
podman ps                                       # List running containers
podman ps -a                                    # List all containers
podman images                                   # List images
podman stop myapp                               # Stop container
podman start myapp                              # Start container
podman rm myapp                                 # Remove container
podman rmi image-name                           # Remove image
podman logs myapp                               # View logs
podman exec -it myapp /bin/bash                 # Execute command
```

**Docker (requires daemon):**
```bash
docker run -d --name myapp -p 8080:80 nginx    # Run container
docker ps                                       # List running containers
docker images                                   # List images
docker stop myapp                               # Stop container
docker start myapp                              # Start container
docker rm myapp                                 # Remove container
docker rmi image-name                           # Remove image
docker logs myapp                               # View logs
docker exec -it myapp /bin/bash                 # Execute command
systemctl status docker                         # Check Docker service
```

**Container Networking:**
```bash
podman network ls                               # List networks
podman network create mynet                     # Create network
podman run --network mynet nginx                # Run with specific network
```

## Virtual Machines (libvirt)

**virsh Commands:**
```bash
virsh list                          # List running VMs
virsh list --all                    # List all VMs
virsh start vm-name                 # Start VM
virsh shutdown vm-name              # Graceful shutdown
virsh destroy vm-name               # Force stop
virsh reboot vm-name                # Reboot VM
virsh autostart vm-name             # Enable autostart
virsh autostart --disable vm-name   # Disable autostart
virsh console vm-name               # Connect to console
virsh dominfo vm-name               # VM information
```

**Create VM:**
```bash
virt-install \
  --name myvm \
  --ram 2048 \
  --disk path=/var/lib/libvirt/images/myvm.qcow2,size=20 \
  --vcpus 2 \
  --os-variant ubuntu20.04 \
  --network bridge=virbr0 \
  --graphics none \
  --console pty,target_type=serial \
  --cdrom /path/to/ubuntu.iso
```

**Manage VMs:**
```bash
virsh edit vm-name                  # Edit VM configuration
virsh dumpxml vm-name               # View VM XML config
virsh snapshot-create-as vm-name snapshot1  # Create snapshot
virsh snapshot-list vm-name         # List snapshots
virsh snapshot-revert vm-name snapshot1     # Revert to snapshot
```

**Networking:**
```bash
virsh net-list                      # List networks
virsh net-start default             # Start network
virsh net-autostart default         # Enable network autostart
```
