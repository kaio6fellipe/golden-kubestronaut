# Users and Groups

> Reference: [Kodekloud public notes](https://notes.kodekloud.com/docs/Linux-Foundation-Certified-System-Administrator-LFCS/Users-and-Groups/Users-and-Groups)

## Table of Contents

- [Users and Groups](#users-and-groups)
  - [Table of Contents](#table-of-contents)
  - [Local User Accounts](#local-user-accounts)
    - [Creating User Accounts](#creating-user-accounts)
    - [Managing User Passwords](#managing-user-passwords)
    - [Deleting User Accounts](#deleting-user-accounts)
    - [Modifying User Accounts](#modifying-user-accounts)
    - [Viewing User Information](#viewing-user-information)
  - [Local Groups](#local-groups)
    - [Creating and Managing Groups](#creating-and-managing-groups)
    - [Modifying Groups](#modifying-groups)
  - [System-Wide Environment Profiles](#system-wide-environment-profiles)
    - [Managing Environment Variables](#managing-environment-variables)
    - [System-Wide Configuration Files](#system-wide-configuration-files)
  - [Template User Environment](#template-user-environment)
    - [Managing /etc/skel](#managing-etcskel)
  - [User Resource Limits](#user-resource-limits)
    - [Managing ulimit (Current Session)](#managing-ulimit-current-session)
    - [Persistent Limits (/etc/security/limits.conf)](#persistent-limits-etcsecuritylimitsconf)
  - [User Privileges](#user-privileges)
    - [Managing sudo Access](#managing-sudo-access)
    - [sudoers File Syntax](#sudoers-file-syntax)
  - [Access to Root Account](#access-to-root-account)
    - [Using su (Switch User)](#using-su-switch-user)
    - [Using sudo](#using-sudo)
    - [Restricting Root Access](#restricting-root-access)
  - [LDAP User and Group Accounts](#ldap-user-and-group-accounts)
    - [Installing Required Packages](#installing-required-packages)
    - [Configuring LDAP Authentication](#configuring-ldap-authentication)
    - [Using SSSD (System Security Services Daemon)](#using-sssd-system-security-services-daemon)
  - [Additional Useful Commands](#additional-useful-commands)

---

## Local User Accounts

### Creating User Accounts

```bash
# Create a new user account (interactive: prompts for password and optional info)
# Creates home directory at /home/username, primary group, and sets default shell to /bin/bash
sudo adduser john
```

```bash
# Create a user with specific UID
sudo adduser --uid 1100 smith
```

```bash
# Create a system account (for services/daemons)
# System accounts have UIDs below 1000, no home directory, no interactive login
sudo adduser --system --no-create-home sysacc
```

### Managing User Passwords

```bash
# Set or change user password
sudo passwd john
```

```bash
# Force immediate password change at next login
sudo chage --lastday 0 jane
# Or shorthand:
sudo chage -d 0 jane
```

```bash
# Set password to expire every 30 days
sudo chage --maxdays 30 jane
```

```bash
# Set password to never expire
sudo chage --maxdays -1 jane
```

```bash
# View password and account aging settings
sudo chage --list jane
```

### Deleting User Accounts

```bash
# Delete user account (keeps home directory)
sudo deluser john
```

```bash
# Delete user and remove home directory
sudo deluser --remove-home john
```

### Modifying User Accounts

```bash
# Change user's home directory and move existing files
sudo usermod --home /home/otherdirectory --move-home john
```

```bash
# Rename a user (change login name)
sudo usermod --login jane john
# Or shorthand:
sudo usermod -l jane john
```

```bash
# Change user's login shell
sudo usermod --shell /bin/othershell jane
# Or shorthand:
sudo usermod -s /bin/othershell jane
```

```bash
# Lock user account (disable password login)
sudo usermod --lock jane
```

```bash
# Unlock user account
sudo usermod --unlock jane
```

```bash
# Set account expiration date (YYYY-MM-DD format)
sudo usermod --expiredate 2028-12-10 jane
```

```bash
# Remove account expiration date
sudo usermod --expiredate "" jane
```

```bash
# Change user's primary group
sudo usermod --gid developers john
# Or shorthand:
sudo usermod -g developers john
```

### Viewing User Information

```bash
# Check current user
whoami
```

```bash
# Display user ID, group ID, and group memberships
id
```

```bash
# View /etc/passwd file (username, UID, GID, home directory, shell)
cat /etc/passwd
```

```bash
# List file ownership with numeric UIDs/GIDs
ls -ln /home/
```

---

## Local Groups

### Creating and Managing Groups

```bash
# Create a new group
sudo groupadd developers
```

```bash
# Add user to a secondary (supplementary) group
sudo gpasswd --add john developers
# Or shorthand:
sudo gpasswd -a john developers
```

```bash
# Remove user from a secondary group
sudo gpasswd --delete john developers
# Or shorthand:
sudo gpasswd -d john developers
```

```bash
# View user's group memberships
groups john
```

### Modifying Groups

```bash
# Rename a group
sudo groupmod --new-name programmers developers
# Or shorthand:
sudo groupmod -n programmers developers
```

```bash
# Delete a group (fails if it's a user's primary group)
sudo groupdel programmers
```

---

## System-Wide Environment Profiles

### Managing Environment Variables

```bash
# Display all environment variables
printenv
# Or:
env
```

```bash
# Display specific environment variable
echo $HOME
echo $PATH
```

```bash
# Set temporary environment variable (current session only)
export MYVAR="value"
```

```bash
# Set temporary environment variable for single command
MYVAR="value" command_to_run
```

### System-Wide Configuration Files

```bash
# View /etc/environment file (system-wide environment variables)
# Format: KEY="VALUE" (no export needed, no variable expansion)
cat /etc/environment
```

```bash
# Edit system-wide environment variables
sudo vim /etc/environment
```

```bash
# Create login script in /etc/profile.d/ (executed for all users at login)
# Example: log last login time
sudo vim /etc/profile.d/lastlogin.sh
```

Example script content for `/etc/profile.d/lastlogin.sh`:
```bash
echo "Your last login was: " > $HOME/lastlogin
date >> $HOME/lastlogin
```

---

## Template User Environment

### Managing /etc/skel

```bash
# View default files copied to new user home directories
ls -a /etc/skel
```

```bash
# Add files to /etc/skel that will be copied to all new user accounts
sudo cp custom_file /etc/skel/
```

```bash
# Edit default bash configuration for new users
sudo vim /etc/skel/.bashrc
```

> **Note:** Files in `/etc/skel` are only copied during user creation. Existing users won't automatically get these files.

---

## User Resource Limits

### Managing ulimit (Current Session)

```bash
# View all current resource limits
ulimit -a
```

```bash
# View soft limit for open files
ulimit -Sn
```

```bash
# View hard limit for open files
ulimit -Hn
```

```bash
# Set soft limit for open files (current session)
ulimit -n 2048
```

```bash
# Set hard limit for max processes (requires root/sudo)
ulimit -u 100
```

### Persistent Limits (/etc/security/limits.conf)

```bash
# Edit system-wide persistent resource limits
sudo vim /etc/security/limits.conf
```

Format: `<domain> <type> <item> <value>`
- **domain:** username, @groupname, or * (all users)
- **type:** soft, hard, or - (both)
- **item:** nofile, nproc, memlock, etc.
- **value:** limit value

Example entries:
```
john soft nofile 2048
john hard nofile 4096
@developers soft nproc 100
* hard memlock 64
```

```bash
# Create drop-in configuration file
sudo vim /etc/security/limits.d/custom.conf
```

---

## User Privileges

### Managing sudo Access

```bash
# Edit sudoers file safely (with syntax checking)
sudo visudo
```

```bash
# Edit sudoers file for specific user
sudo visudo -f /etc/sudoers.d/john
```

```bash
# Add user to sudo group (Debian/Ubuntu)
sudo usermod -aG sudo john
```

```bash
# Add user to wheel group (RHEL/CentOS/Fedora)
sudo usermod -aG wheel john
```

### sudoers File Syntax

```
# Allow user to run all commands with sudo
john ALL=(ALL:ALL) ALL

# Allow user to run specific commands without password
john ALL=(ALL) NOPASSWD: /usr/bin/systemctl, /usr/bin/apt

# Allow group to run all commands
%developers ALL=(ALL:ALL) ALL

# Allow user to run commands as specific user
john ALL=(postgres) ALL
```

```bash
# Test sudo configuration for specific user
sudo -l -U john
```

```bash
# Run command as another user with sudo
sudo -u postgres psql
```

---

## Access to Root Account

### Using su (Switch User)

```bash
# Switch to root user (requires root password)
su -
# Or:
su - root
```

```bash
# Switch to another user
su - john
```

```bash
# Run command as root without switching shell
su -c "command" root
```

### Using sudo

```bash
# Execute command with root privileges (requires user's password)
sudo command
```

```bash
# Open root shell using sudo
sudo -i
# Or:
sudo -s
```

```bash
# Run command as another user
sudo -u username command
```

### Restricting Root Access

```bash
# Lock root account (disable password login)
sudo passwd -l root
```

```bash
# Unlock root account
sudo passwd -u root
```

```bash
# Disable root SSH login (edit sshd_config)
sudo vim /etc/ssh/sshd_config
# Set: PermitRootLogin no
```

```bash
# Restart SSH service after config change
sudo systemctl restart sshd
```

---

## LDAP User and Group Accounts

### Installing Required Packages

```bash
# Install LDAP client packages (Debian/Ubuntu)
sudo apt install libnss-ldap libpam-ldap ldap-utils nscd
```

```bash
# Install LDAP client packages (RHEL/CentOS)
sudo yum install nss-pam-ldapd pam_ldap openldap-clients nscd
```

### Configuring LDAP Authentication

```bash
# Configure LDAP authentication (Debian/Ubuntu)
sudo dpkg-reconfigure ldap-auth-config
```

```bash
# Edit LDAP client configuration
sudo vim /etc/ldap/ldap.conf
```

Example `/etc/ldap/ldap.conf`:
```
BASE dc=example,dc=com
URI ldap://ldap.example.com
```

```bash
# Edit NSS configuration
sudo vim /etc/nsswitch.conf
```

Ensure these lines include `ldap`:
```
passwd: files ldap
group:  files ldap
shadow: files ldap
```

```bash
# Edit PAM LDAP configuration
sudo vim /etc/pam_ldap.conf
```

```bash
# Restart nscd service
sudo systemctl restart nscd
```

```bash
# Test LDAP connection
ldapsearch -x -H ldap://ldap.example.com -b "dc=example,dc=com"
```

```bash
# View LDAP users
getent passwd
```

```bash
# View LDAP groups
getent group
```

### Using SSSD (System Security Services Daemon)

```bash
# Install SSSD
sudo apt install sssd sssd-ldap
```

```bash
# Configure SSSD
sudo vim /etc/sssd/sssd.conf
```

Example `/etc/sssd/sssd.conf`:
```
[sssd]
services = nss, pam
config_file_version = 2
domains = LDAP

[domain/LDAP]
id_provider = ldap
auth_provider = ldap
ldap_uri = ldap://ldap.example.com
ldap_search_base = dc=example,dc=com
```

```bash
# Set proper permissions on sssd.conf
sudo chmod 600 /etc/sssd/sssd.conf
```

```bash
# Enable and start SSSD
sudo systemctl enable sssd
sudo systemctl start sssd
```

```bash
# Check SSSD status
sudo systemctl status sssd
```

---

## Additional Useful Commands

```bash
# View user's last login information
lastlog -u john
```

```bash
# View currently logged-in users
who
# Or:
w
```

```bash
# View login history
last
```

```bash
# View failed login attempts
sudo lastb
```

```bash
# View user creation default settings
useradd -D
```

```bash
# View detailed group information
getent group groupname
```

```bash
# View detailed user information
getent passwd username
```
