_This project was created as part of the 42 curriculum by mobenhab_

# Description

born2beroot is an educational project aimed at learning how to securely set up and configure a Linux server from scratch. Its goal is to familiarize students with system administration, security, and the management of essential services.

**Goal:**
- Install and configure a functional and secure Linux server
- Learn to manage users, permissions, and system services
- Strengthen server security by applying best practices (firewall, sudo, passwords, logs, etc.)

**Technologies used:** Linux (Debian), Bash, LVM, SSH, UFW

This project provides hands-on experience with Linux system administration and helps develop essential skills for future system administrators or DevOps engineers.

---

#  OS Choice

## Why Debian?

For this project, I chose **Debian** as the operating system for the following reasons:

**Pros:**
- Beginner-friendly with extensive community support and documentation
- Highly stable and reliable, ideal for learning server administration
- Large package repository with well-tested software
- Uses APT package manager which is straightforward and well-documented
- AppArmor is easier to configure than SELinux for beginners
- Lighter resource usage compared to enterprise distributions

**Cons:**
- Packages are sometimes older due to stability focus
- Less enterprise-oriented than RHEL-based distributions
- Fewer certifications and enterprise training materials available

**Alternative: Rocky Linux**
- Enterprise-focused, RHEL-compatible distribution
- Long-term support suitable for production environments
- Better for those pursuing enterprise Linux careers
- Steeper learning curve with SELinux
- More complex firewall management with firewalld

---


## Partitioning Scheme

The server uses **LVM (Logical Volume Manager)** with encrypted partitions for enhanced security and flexibility:
```
/boot          (500 MB)  - Unencrypted boot partition
/home          (5 GB)    - User home directories
/var           (3 GB)    - Variable data (logs, databases)
/var/log       (2 GB)    - System logs (separate for security)
swap           (2 GB)    - Swap space
```

**Rationale:**
- Encrypted LVM provides data protection at rest
- Separate /var/log prevents log flooding from affecting the system
- Separate /tmp improves security and prevents disk space issues
- LVM allows for easy resizing of partitions if needed

## Security Policies

### Password Policy
- Minimum password length: 10 characters
- Password must contain uppercase, lowercase, and numbers
- Password expires every 30 days
- Minimum days before password change: 2
- Warning 7 days before password expiration
- Password history: last 3 passwords cannot be reused

Implementation via `/etc/login.defs` 
### Sudo Configuration
- Limited authentication attempts (3 tries)
- Custom error message for wrong password
- Log all sudo commands to `/var/log/sudo/sudo.log`
- TTY mode required for security
- Restricted PATH for sudo commands
- Input/output logs archived in `/var/log/sudo/`

Configuration in `/etc/sudoers.d/sudo_config`

### SSH Hardening
- SSH runs on custom port 4242 (not default 22)
- Root login disabled via SSH
- Password authentication allowed (required for evaluation)
- SSH protocol version 2 only
- Strict mode enabled

Configuration in `/etc/ssh/sshd_config`

## User Management

**User Groups:**
- `user42`: Standard user group for project requirements
- `sudo`: Administrative privileges group

**User Creation Workflow:**
1. Create user with `adduser username`
2. Set strong password following policy
3. Add to appropriate groups: `usermod -aG user42,sudo username`
4. Verify group membership: `groups username`

**Monitoring:**
- Regular review of `/etc/passwd` and `/etc/group`
- Audit sudo logs for privilege escalation attempts
- Track user login activity via `/var/log/auth.log`

## Services

### SSH Server
- **Service:** OpenSSH Server
- **Port:** 4242
- **Purpose:** Secure remote administration
- **Security:** Key-based or password authentication, no root login

### UFW Firewall
- **Service:** Uncomplicated Firewall (UFW)
- **Configuration:** Default deny incoming, allow outgoing
- **Open Ports:** 4242 (SSH only)
- **Purpose:** Network access control and attack surface reduction

### Monitoring Script
- **Service:** Custom bash script via cron
- **Frequency:** Every 10 minutes
- **Purpose:** Display system information (architecture, CPU, RAM, disk, network, users, etc.)
- **Location:** `/usr/local/bin/monitoring.sh`

---


## Debian vs Rocky Linux

| Aspect | Debian | Rocky Linux |
|--------|--------|-------------|
| **Target Audience** | General use, beginners | Enterprise environments |
| **Package Manager** | APT (Advanced Package Tool) | DNF/YUM |
| **Release Cycle** | ~2 years (Stable) | ~6 months (follows RHEL) |
| **Support** | Community-driven | Enterprise-grade, RHEL-compatible |
| **Learning Curve** | Easier for beginners | Steeper, enterprise-focused |
| **Security Module** | AppArmor (default) | SELinux (default) |
| **Use Case** | Learning, small servers | Production, enterprise compliance |

**Choice for born2beroot:** Debian is preferred for its beginner-friendly approach and extensive documentation.

## AppArmor vs SELinux

| Aspect | AppArmor | SELinux |
|--------|----------|---------|
| **Security Model** | Path-based access control | Label-based access control |
| **Configuration** | Profile-based, easier to understand | Policy-based, more complex |
| **Learning Curve** | Gentle, suitable for learning | Steep, requires deep understanding |
| **Granularity** | Moderate control | Fine-grained control |
| **Default on** | Debian, Ubuntu | RHEL, Fedora, Rocky Linux |
| **Debugging** | Simpler with `aa-status` | Complex, requires `audit2allow` |

**Choice for born2beroot:** AppArmor is more appropriate for learning system administration basics.

## UFW vs firewalld

| Aspect | UFW | firewalld |
|--------|-----|-----------|
| **Interface** | Simple command-line | XML-based + rich command-line |
| **Configuration** | Rule-based, straightforward | Zone-based, dynamic |
| **Complexity** | Low, ideal for small setups | Higher, designed for enterprise |
| **Runtime Changes** | Requires reload | Dynamic, no restart needed |
| **Default on** | Ubuntu, Debian | RHEL, Fedora, Rocky Linux |
| **Best For** | Simple servers, learning | Complex networks, production |

**Choice for born2beroot:** UFW provides simplicity and ease of use, perfect for educational purposes.

## VirtualBox vs UTM

| Aspect | VirtualBox | UTM |
|--------|------------|-----|
| **Platform** | Cross-platform (Windows, Mac, Linux) | macOS only (especially M1/M2) |
| **Backend** | Custom virtualization engine | QEMU-based |
| **Performance** | Good on Intel, moderate on ARM | Optimized for Apple Silicon |
| **Features** | Snapshots, shared folders, extensive networking | Lightweight, simpler interface |
| **Networking** | Easy setup, multiple modes | Slightly more complex |
| **Use Case** | General virtualization, widely adopted | Mac users, especially ARM-based |

**Choice for born2beroot:** VirtualBox is recommended for its cross-platform support and extensive documentation, but UTM is excellent for Mac users with Apple Silicon.

---

# Instructions

## 1. Installing sudo
```bash
su -
apt update
apt install sudo
usermod -aG sudo your_username
```

## 2. Install OpenSSH Server
```bash
sudo apt update
sudo apt install openssh-server
sudo systemctl status ssh
```

## 3. Configure SSH
```bash
sudo nano /etc/ssh/sshd_config
# Change Port to 4242
# Set PermitRootLogin no
sudo systemctl restart ssh
```

## 4. Create a New User
```bash
sudo adduser username
```

## 5. Create a New Group
```bash
sudo addgroup groupname
```

## 6. Configure UFW Firewall
```bash
sudo apt install ufw
sudo ufw enable
sudo ufw allow 4242
sudo ufw status numbered
```

## 7. Configure Password Policy
```bash
sudo apt install libpam-pwquality
sudo nano /etc/login.defs
sudo nano /etc/security/pwquality.conf
```

## 8. Configure Sudo Policy
```bash
sudo visudo -f /etc/sudoers.d/sudo_config
```

---

#  Resources

**Official Documentation**
- [Linux Kernel Docs](https://www.kernel.org/doc/) – Official Linux kernel documentation
- [Debian](https://www.debian.org/download.fr.html) – Official Debian download and documentation site
- [Sudo Manual](https://www.sudo.ws/docs/) – Comprehensive reference for configuring sudo

**Tutorials & Guides**
- [SSH Essentials](https://www.ssh.com/academy/ssh) – Guide to securely configure SSH connections
- [LVM Guide](https://wiki.debian.org/LVM) – Debian LVM documentation
- [AppArmor Wiki](https://gitlab.com/apparmor/apparmor/-/wikis/home) – AppArmor configuration guide

**Tools & Utilities**
- [UFW](https://wiki.ubuntu.com/UncomplicatedFirewall) – Simple firewall management tool
- [PAM Documentation](https://www.linux-pam.org/Linux-PAM-html/) – Pluggable Authentication Modules guide

---

**how AI was used**

- Debugging technical issues (SSH connectivity, PAM configuration)
- Understanding command documentation and Linux concepts

