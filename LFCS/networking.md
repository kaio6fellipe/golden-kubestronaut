# Networking

> Reference: [Kodekloud public notes](https://notes.kodekloud.com/docs/Linux-Foundation-Certified-System-Administrator-LFCS/Introduction/Course-Introduction)

## Table of Contents

- [Networking](#networking)
  - [Table of Contents](#table-of-contents)
  - [IPv4 and IPv6 Networking](#ipv4-and-ipv6-networking)
    - [Viewing Network Configuration](#viewing-network-configuration)
    - [Activating and Managing Interfaces](#activating-and-managing-interfaces)
    - [Manually Adding IP Addresses (Temporary)](#manually-adding-ip-addresses-temporary)
    - [Permanent Network Configuration with Netplan](#permanent-network-configuration-with-netplan)
    - [Adding Routes](#adding-routes)
  - [Hostname Resolution](#hostname-resolution)
    - [Managing Hostnames](#managing-hostnames)
    - [DNS Configuration](#dns-configuration)
    - [Local Hostname Resolution](#local-hostname-resolution)
  - [Network Services](#network-services)
    - [Managing Network Services](#managing-network-services)
    - [Network Diagnostics](#network-diagnostics)
  - [Bridge and Bonding Devices](#bridge-and-bonding-devices)
    - [Network Bridging](#network-bridging)
    - [Network Bonding (Link Aggregation)](#network-bonding-link-aggregation)
  - [Packet Filtering and Firewall](#packet-filtering-and-firewall)
    - [UFW (Uncomplicated Firewall) - Ubuntu/Debian](#ufw-uncomplicated-firewall---ubuntudebian)
    - [iptables - Low-Level Firewall](#iptables---low-level-firewall)
    - [firewalld - RHEL/CentOS](#firewalld---rhelcentos)
  - [Port Redirection and NAT](#port-redirection-and-nat)
    - [Enable IP Forwarding](#enable-ip-forwarding)
    - [Port Forwarding with iptables](#port-forwarding-with-iptables)
    - [Network Address Translation (NAT)](#network-address-translation-nat)
  - [Reverse Proxies and Load Balancers](#reverse-proxies-and-load-balancers)
    - [Nginx Reverse Proxy](#nginx-reverse-proxy)
    - [Nginx Load Balancer](#nginx-load-balancer)
    - [HAProxy Load Balancer](#haproxy-load-balancer)
  - [System Time Synchronization](#system-time-synchronization)
    - [Viewing Time Information](#viewing-time-information)
    - [Setting Time and Timezone](#setting-time-and-timezone)
    - [NTP Time Synchronization](#ntp-time-synchronization)
    - [Chrony (Modern NTP Client)](#chrony-modern-ntp-client)
    - [NTP (Traditional NTP Daemon)](#ntp-traditional-ntp-daemon)
  - [SSH Configuration](#ssh-configuration)
    - [SSH Service Management](#ssh-service-management)
    - [SSH Server Configuration](#ssh-server-configuration)
    - [SSH Key-Based Authentication](#ssh-key-based-authentication)
    - [SSH Client Usage](#ssh-client-usage)
    - [SSH Client Configuration](#ssh-client-configuration)
    - [SSH File Transfers](#ssh-file-transfers)
    - [SSH Security Best Practices](#ssh-security-best-practices)
  - [Additional Networking Commands](#additional-networking-commands)
  - [Netplan Reference Examples](#netplan-reference-examples)

---

## IPv4 and IPv6 Networking

### Viewing Network Configuration

```bash
# Display all network interfaces and their IP addresses
ip link
```

```bash
# Display IP addresses assigned to interfaces (detailed)
ip addr
# Or shorthand:
ip a
```

```bash
# Display IP addresses with color highlighting
ip -c addr
```

```bash
# View specific interface
ip addr show enp0s3
```

```bash
# View routing table
ip route
# Or shorthand:
ip r
```

### Activating and Managing Interfaces

```bash
# Bring interface up
sudo ip link set enp0s8 up
```

```bash
# Bring interface down
sudo ip link set enp0s8 down
```

### Manually Adding IP Addresses (Temporary)

```bash
# Add IPv4 address to interface
sudo ip addr add 10.0.0.9/24 dev enp0s8
```

```bash
# Add IPv6 address to interface
sudo ip -6 addr add 2001:db8::1/64 dev enp0s8
```

```bash
# Remove IP address from interface
sudo ip addr del 10.0.0.9/24 dev enp0s8
```

### Permanent Network Configuration with Netplan

```bash
# List netplan configuration files
ls /etc/netplan/
```

```bash
# Edit netplan configuration
sudo nano /etc/netplan/00-installer-config.yaml
```

Example static IPv4 configuration for `/etc/netplan/00-installer-config.yaml`:
```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s8:
      addresses:
        - 10.0.0.9/24
      dhcp4: false
```

Example static IPv6 configuration:
```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s8:
      addresses:
        - 2001:db8::1/64
      dhcp6: false
```

Example DHCP configuration:
```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      dhcp4: true
```

```bash
# Test netplan configuration (120 second timeout to revert if issues)
sudo netplan try
```

```bash
# Apply netplan configuration immediately
sudo netplan apply
```

### Adding Routes

```bash
# Add static route temporarily
sudo ip route add 192.168.0.0/24 via 10.0.0.100
```

Example netplan configuration with routes:
```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s8:
      addresses:
        - 10.0.0.9/24
      routes:
        - to: 192.168.0.0/24
          via: 10.0.0.100
        - to: default
          via: 10.0.0.1
```

---

## Hostname Resolution

### Managing Hostnames

```bash
# Display current hostname
hostname
```

```bash
# Display detailed hostname information
hostnamectl
```

```bash
# Set hostname permanently
sudo hostnamectl set-hostname newname
```

### DNS Configuration

```bash
# View DNS resolver status and configured DNS servers
resolvectl status
```

```bash
# View DNS servers for all interfaces
resolvectl dns
```

Example netplan configuration with DNS nameservers:
```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s8:
      addresses:
        - 10.0.0.9/24
      nameservers:
        addresses:
          - 1.1.1.1
          - 9.9.9.9
        search:
          - example.com
```

```bash
# Edit global DNS resolver configuration
sudo nano /etc/systemd/resolved.conf
```

Example `/etc/systemd/resolved.conf`:
```
[Resolve]
DNS=1.1.1.1 9.9.9.9
FallbackDNS=8.8.8.8 8.8.4.4
```

```bash
# Restart systemd-resolved service
sudo systemctl restart systemd-resolved.service
```

### Local Hostname Resolution

```bash
# Edit /etc/hosts for local hostname mapping
sudo nano /etc/hosts
```

Example `/etc/hosts` entries:
```
127.0.0.1       localhost
127.0.1.1       myhost
127.0.123.123   dbserver
192.168.1.50    webserver
```

```bash
# Test local hostname resolution
ping dbserver
```

---

## Network Services

### Managing Network Services

```bash
# Start network service
sudo systemctl start networking
```

```bash
# Stop network service
sudo systemctl stop networking
```

```bash
# Restart network service
sudo systemctl restart networking
```

```bash
# Check network service status
sudo systemctl status networking
```

```bash
# Enable service to start on boot
sudo systemctl enable networking
```

```bash
# Disable service from starting on boot
sudo systemctl disable networking
```

### Network Diagnostics

```bash
# Test connectivity to host
ping google.com
```

```bash
# Trace route to destination
traceroute google.com
# Or with MTR (better alternative):
mtr google.com
```

```bash
# Display all TCP/UDP listening ports and connections
ss -tulpn
```

```bash
# Display routing table
ip route show
```

```bash
# View network statistics
netstat -s
# Or with ss:
ss -s
```

```bash
# Test DNS resolution
nslookup google.com
# Or with dig:
dig google.com
```

---

## Bridge and Bonding Devices

### Network Bridging

```bash
# Create bridge interface
sudo ip link add name br0 type bridge
```

```bash
# Add interface to bridge
sudo ip link set enp0s3 master br0
```

```bash
# Bring bridge up
sudo ip link set br0 up
```

```bash
# View bridge information
ip link show type bridge
```

```bash
# View bridge details (requires bridge-utils)
brctl show
```

Example netplan bridge configuration:
```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      dhcp4: false
    enp0s8:
      dhcp4: false
  bridges:
    br0:
      interfaces:
        - enp0s3
        - enp0s8
      dhcp4: true
```

### Network Bonding (Link Aggregation)

```bash
# Load bonding kernel module
sudo modprobe bonding
```

```bash
# Create bond interface (active-backup mode)
sudo ip link add bond0 type bond mode active-backup
```

```bash
# Add interfaces to bond
sudo ip link set enp0s3 master bond0
sudo ip link set enp0s8 master bond0
```

```bash
# Bring bond interface up
sudo ip link set bond0 up
```

```bash
# View bonding information
cat /proc/net/bonding/bond0
```

Example netplan bonding configuration:
```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      dhcp4: false
    enp0s8:
      dhcp4: false
  bonds:
    bond0:
      interfaces:
        - enp0s3
        - enp0s8
      parameters:
        mode: active-backup
        mii-monitor-interval: 100
      dhcp4: true
```

Bonding modes:
- `balance-rr` (0): Round-robin load balancing
- `active-backup` (1): Fault tolerance (one active, others backup)
- `balance-xor` (2): XOR load balancing
- `broadcast` (3): Broadcast on all interfaces
- `802.3ad` (4): IEEE 802.3ad dynamic link aggregation (LACP)
- `balance-tlb` (5): Adaptive transmit load balancing
- `balance-alb` (6): Adaptive load balancing

---

## Packet Filtering and Firewall

### UFW (Uncomplicated Firewall) - Ubuntu/Debian

```bash
# Check firewall status
sudo ufw status
# Or verbose:
sudo ufw status verbose
```

```bash
# Enable firewall
sudo ufw enable
```

```bash
# Disable firewall
sudo ufw disable
```

```bash
# Allow incoming traffic on port
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

```bash
# Deny incoming traffic on port
sudo ufw deny 23/tcp
```

```bash
# Allow from specific IP address
sudo ufw allow from 192.168.1.100
```

```bash
# Allow from specific IP to specific port
sudo ufw allow from 192.168.1.0/24 to any port 22
```

```bash
# Delete firewall rule
sudo ufw delete allow 80/tcp
```

```bash
# Reset firewall to default
sudo ufw reset
```

```bash
# Set default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

### iptables - Low-Level Firewall

```bash
# List all firewall rules
sudo iptables -L -n -v
```

```bash
# List rules with line numbers
sudo iptables -L --line-numbers
```

```bash
# Allow incoming SSH
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
```

```bash
# Allow established connections
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
```

```bash
# Drop all other incoming traffic
sudo iptables -A INPUT -j DROP
```

```bash
# Delete rule by line number
sudo iptables -D INPUT 3
```

```bash
# Save iptables rules (Ubuntu/Debian)
sudo iptables-save > /etc/iptables/rules.v4
sudo ip6tables-save > /etc/iptables/rules.v6
```

```bash
# Restore iptables rules
sudo iptables-restore < /etc/iptables/rules.v4
```

```bash
# Flush all rules
sudo iptables -F
```

### firewalld - RHEL/CentOS

```bash
# Start firewalld service
sudo systemctl start firewalld
```

```bash
# Check firewall status
sudo firewall-cmd --state
```

```bash
# List all zones
sudo firewall-cmd --get-zones
```

```bash
# List active zones
sudo firewall-cmd --get-active-zones
```

```bash
# Add service permanently
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
```

```bash
# Add port permanently
sudo firewall-cmd --permanent --add-port=8080/tcp
```

```bash
# Remove port
sudo firewall-cmd --permanent --remove-port=8080/tcp
```

```bash
# Reload firewall configuration
sudo firewall-cmd --reload
```

```bash
# List all rules
sudo firewall-cmd --list-all
```

---

## Port Redirection and NAT

### Enable IP Forwarding

```bash
# Enable IP forwarding temporarily
sudo sysctl -w net.ipv4.ip_forward=1
```

```bash
# Enable IP forwarding permanently
sudo nano /etc/sysctl.conf
# Add or uncomment: net.ipv4.ip_forward=1
```

```bash
# Apply sysctl changes
sudo sysctl -p
```

### Port Forwarding with iptables

```bash
# Redirect incoming port 8080 to port 80
sudo iptables -t nat -A PREROUTING -p tcp --dport 8080 -j REDIRECT --to-port 80
```

```bash
# Forward port to another host
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination 192.168.1.50:80
```

```bash
# Forward from specific interface
sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j DNAT --to-destination 192.168.1.50:8080
```

### Network Address Translation (NAT)

```bash
# Enable masquerading (SNAT) for outgoing traffic
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

```bash
# Source NAT to specific IP
sudo iptables -t nat -A POSTROUTING -o eth0 -j SNAT --to-source 203.0.113.1
```

```bash
# View NAT table rules
sudo iptables -t nat -L -n -v
```

---

## Reverse Proxies and Load Balancers

### Nginx Reverse Proxy

```bash
# Install nginx
sudo apt install nginx
```

```bash
# Edit nginx configuration
sudo nano /etc/nginx/sites-available/default
```

Example nginx reverse proxy configuration:
```nginx
server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass http://192.168.1.50:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

```bash
# Test nginx configuration
sudo nginx -t
```

```bash
# Reload nginx
sudo systemctl reload nginx
```

```bash
# Restart nginx
sudo systemctl restart nginx
```

### Nginx Load Balancer

Example nginx load balancer configuration:
```nginx
upstream backend {
    server 192.168.1.50:8080;
    server 192.168.1.51:8080;
    server 192.168.1.52:8080;
}

server {
    listen 80;

    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
    }
}
```

Load balancing methods:
```nginx
upstream backend {
    # Round-robin (default)
    server 192.168.1.50:8080;
    server 192.168.1.51:8080;

    # Least connections
    least_conn;
    server 192.168.1.50:8080;
    server 192.168.1.51:8080;

    # IP hash (session persistence)
    ip_hash;
    server 192.168.1.50:8080;
    server 192.168.1.51:8080;
}
```

### HAProxy Load Balancer

```bash
# Install HAProxy
sudo apt install haproxy
```

```bash
# Edit HAProxy configuration
sudo nano /etc/haproxy/haproxy.cfg
```

Example HAProxy configuration:
```
frontend web_frontend
    bind *:80
    default_backend web_servers

backend web_servers
    balance roundrobin
    server web1 192.168.1.50:80 check
    server web2 192.168.1.51:80 check
    server web3 192.168.1.52:80 check
```

```bash
# Test HAProxy configuration
sudo haproxy -c -f /etc/haproxy/haproxy.cfg
```

```bash
# Restart HAProxy
sudo systemctl restart haproxy
```

---

## System Time Synchronization

### Viewing Time Information

```bash
# Display current date and time
date
```

```bash
# Display hardware clock
sudo hwclock --show
```

```bash
# Display time and timezone settings
timedatectl
```

```bash
# List available timezones
timedatectl list-timezones
```

```bash
# List available timezones for specific region
timedatectl list-timezones | grep America
```

### Setting Time and Timezone

```bash
# Set timezone
sudo timedatectl set-timezone America/New_York
```

```bash
# Set time manually (disables NTP)
sudo timedatectl set-time "2024-01-15 10:30:00"
```

```bash
# Set date only
sudo timedatectl set-time "2024-01-15"
```

```bash
# Sync hardware clock to system clock
sudo hwclock --systohc
```

### NTP Time Synchronization

```bash
# Enable NTP synchronization
sudo timedatectl set-ntp true
```

```bash
# Disable NTP synchronization
sudo timedatectl set-ntp false
```

```bash
# Check NTP synchronization status
timedatectl status
```

```bash
# Show time synchronization status
timedatectl timesync-status
```

### Chrony (Modern NTP Client)

```bash
# Install chrony
sudo apt install chrony
```

```bash
# Start chrony service
sudo systemctl start chrony
```

```bash
# Enable chrony on boot
sudo systemctl enable chrony
```

```bash
# Check chrony status
sudo systemctl status chrony
```

```bash
# View chrony tracking information
chronyc tracking
```

```bash
# View time sources
chronyc sources
```

```bash
# View detailed source statistics
chronyc sourcestats
```

```bash
# Edit chrony configuration
sudo nano /etc/chrony/chrony.conf
```

Example chrony configuration additions:
```
server 0.pool.ntp.org iburst
server 1.pool.ntp.org iburst
server 2.pool.ntp.org iburst
server 3.pool.ntp.org iburst
```

```bash
# Restart chrony after configuration changes
sudo systemctl restart chrony
```

### NTP (Traditional NTP Daemon)

```bash
# Install ntp
sudo apt install ntp
```

```bash
# Check NTP peers
ntpq -p
```

```bash
# Edit NTP configuration
sudo nano /etc/ntp.conf
```

---

## SSH Configuration

### SSH Service Management

```bash
# Start SSH service
sudo systemctl start ssh
# Or on RHEL/CentOS:
sudo systemctl start sshd
```

```bash
# Stop SSH service
sudo systemctl stop ssh
```

```bash
# Restart SSH service
sudo systemctl restart ssh
```

```bash
# Check SSH service status
sudo systemctl status ssh
```

```bash
# Enable SSH on boot
sudo systemctl enable ssh
```

```bash
# Disable SSH from boot
sudo systemctl disable ssh
```

### SSH Server Configuration

```bash
# Edit SSH server configuration
sudo nano /etc/ssh/sshd_config
```

Common SSH server configuration options:
```
Port 22                          # Change default SSH port
ListenAddress 0.0.0.0           # IP address to listen on
PermitRootLogin no              # Disable root login
PasswordAuthentication yes      # Allow password authentication
PubkeyAuthentication yes        # Allow key-based authentication
PermitEmptyPasswords no         # Disallow empty passwords
X11Forwarding yes               # Enable X11 forwarding
MaxAuthTries 3                  # Maximum authentication attempts
ClientAliveInterval 300         # Send keepalive every 300 seconds
ClientAliveCountMax 2           # Disconnect after 2 failed keepalives
AllowUsers user1 user2          # Only allow specific users
DenyUsers baduser               # Deny specific users
AllowGroups sshusers            # Only allow specific groups
```

```bash
# Test SSH configuration syntax
sudo sshd -t
```

```bash
# Reload SSH configuration without disconnecting clients
sudo systemctl reload ssh
```

### SSH Key-Based Authentication

```bash
# Generate SSH key pair (RSA, 4096 bits)
ssh-keygen -t rsa -b 4096
```

```bash
# Generate SSH key pair (Ed25519, more secure and faster)
ssh-keygen -t ed25519
```

```bash
# Generate SSH key with custom filename
ssh-keygen -t rsa -b 4096 -f ~/.ssh/custom_key
```

```bash
# Copy public key to remote server
ssh-copy-id -i ~/.ssh/id_rsa.pub user@server
```

```bash
# Copy specific key to remote server with custom port
ssh-copy-id -i ~/.ssh/id_rsa.pub -p 2222 user@server
```

```bash
# Manually copy public key
cat ~/.ssh/id_rsa.pub | ssh user@server "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

### SSH Client Usage

```bash
# Connect to remote server
ssh user@hostname
```

```bash
# Connect on non-standard port
ssh -p 2222 user@hostname
```

```bash
# Connect using specific private key
ssh -i ~/.ssh/custom_key user@hostname
```

```bash
# Execute command on remote server
ssh user@hostname 'uptime'
```

```bash
# SSH with X11 forwarding
ssh -X user@hostname
```

```bash
# SSH with compression (useful for slow connections)
ssh -C user@hostname
```

```bash
# SSH with verbose output (debugging)
ssh -v user@hostname
# More verbose:
ssh -vv user@hostname
ssh -vvv user@hostname
```

### SSH Client Configuration

```bash
# Edit SSH client configuration (user-specific)
nano ~/.ssh/config
```

Example `~/.ssh/config`:
```
Host myserver
    HostName 192.168.1.100
    User john
    Port 2222
    IdentityFile ~/.ssh/custom_key
    ServerAliveInterval 60
    ServerAliveCountMax 3

Host *.example.com
    User admin
    IdentityFile ~/.ssh/work_key
    ForwardX11 yes

Host *
    ServerAliveInterval 60
    Compression yes
```

```bash
# Connect using config entry
ssh myserver
```

### SSH File Transfers

```bash
# Copy file to remote server (SCP)
scp localfile.txt user@server:/remote/path/
```

```bash
# Copy file from remote server
scp user@server:/remote/file.txt ./local/path/
```

```bash
# Copy directory recursively
scp -r local/directory user@server:/remote/path/
```

```bash
# SCP on non-standard port
scp -P 2222 file.txt user@server:/path/
```

```bash
# Using rsync over SSH (more efficient for large transfers)
rsync -avz -e ssh local/directory/ user@server:/remote/directory/
```

### SSH Security Best Practices

```bash
# Disable password authentication (after setting up keys)
sudo nano /etc/ssh/sshd_config
# Set: PasswordAuthentication no
sudo systemctl restart ssh
```

```bash
# Change default SSH port
sudo nano /etc/ssh/sshd_config
# Set: Port 2222
sudo systemctl restart ssh
```

```bash
# View SSH login attempts (successful and failed)
sudo grep sshd /var/log/auth.log
# Or on RHEL/CentOS:
sudo grep sshd /var/log/secure
```

```bash
# View only failed SSH login attempts
sudo grep "Failed password" /var/log/auth.log
```

---

## Additional Networking Commands

```bash
# View ARP cache
ip neigh show
```

```bash
# Flush ARP cache
sudo ip neigh flush all
```

```bash
# View network interface statistics
ip -s link
```

```bash
# Capture network traffic
sudo tcpdump -i enp0s3
```

```bash
# Capture traffic on specific port
sudo tcpdump -i enp0s3 port 80
```

```bash
# View listening ports
sudo lsof -i -P -n | grep LISTEN
```

```bash
# Test port connectivity
telnet hostname 80
# Or with nc (netcat):
nc -zv hostname 80
```

```bash
# Download file from web
wget http://example.com/file.txt
curl -O http://example.com/file.txt
```

---

## Netplan Reference Examples

```bash
# View netplan example configurations
ls /usr/share/doc/netplan/examples/
```

```bash
# View DHCP example
cat /usr/share/doc/netplan/examples/dhcp.yaml
```

```bash
# View static IP example
cat /usr/share/doc/netplan/examples/static.yaml
```

```bash
# View netplan manual
man netplan
```

```bash
# Generate networkd configuration from netplan
sudo netplan generate
```

```bash
# View generated configuration
ls /run/systemd/network/
```
