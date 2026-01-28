# Tips for the LFCS Exam

- install `tldr` (`apt install -y tldr`) : simplified man page information with examples (e.g. `tldr docker run`)
  - It would be necessary to update the tldr database with `tldr --update` and maybe you will need to create the directory `~/.local/share/tldr`
- if you don't remember a command, search with `apropos` (e.g. `apropos -s 1,8 "director"`, it will search for man page information and descriptions for the word "director")
- Kodekloud notes to review: https://notes.kodekloud.com/docs/Linux-Foundation-Certified-System-Administrator-LFCS/Introduction/Course-Introduction

## Domains and Competencies

1. Operations Deployment (25%)
   - [x] Configure kernel parameters, persistent and non-persistent
   - [x] Diagnose, identify, manage, and troubleshoot processes and services
   - [x] Manage or schedule jobs for executing commands
   - [x] Search for, install, validate, and maintain software packages or repositories
   - [x] Recover from hardware, operating system, or filesystem failures
   - [x] Manage Virtual Machines (libvirt)
     - [ ] Need to review
   - [x] Configure container engines, create and manage containers
   - [x] Create and enforce MAC using SELinux
     - [ ] Need to review

2. Networking (25%)
   - [x] Configure IPv4 and IPv6 networking and hostname resolution
   - [x] Set and synchronize system time using time servers
   - [x] Monitor and troubleshoot networking
   - [x] Configure the OpenSSH server and client
   - [x] Configure packet filtering, port redirection, and NAT
   - [x] Configure static routing
   - [x] Configure bridge and bonding devices
   - [x] Implement reverse proxies and load balancers

3. Storage (20%)
   - [ ] Configure and manage LVM storage
   - [ ] Manage and configure the virtual file system
   - [ ] Create, manage, and troubleshoot filesystems
   - [ ] Use remote filesystems and network block devices
   - [ ] Configure and manage swap space
   - [ ] Configure filesystem automounters
   - [ ] Monitor storage performance

4. Essential Commands (20%)
   - [x] Basic Git Operations
   - [x] Create, configure, and troubleshoot services
   - [x] Monitor and troubleshoot system performance and services
   - [x] Determine application and service specific constraints
   - [x] Troubleshoot diskspace issues
   - [x] Work with SSL certificates

5. Users and Groups (10%)
   - [x] Create and manage local user and group accounts
   - [x] Manage personal and system-wide environment profiles
   - [x] Configure user resource limits
   - [x] Configure and manage ACLs
   - [x] Configure the system to use LDAP user and group accounts

## Notes

- [Operations Deployment](operations-deployment.md)
- [Networking](networking.md)
- [Storage](storage.md)
- [Essential Commands](essential-commands.md)
- [Users and Groups](users-and-groups.md)
