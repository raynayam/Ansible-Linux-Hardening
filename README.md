# Ansible Security Hardening Playbooks

This directory contains Ansible playbooks for automated security hardening of Linux servers based on CIS Benchmarks.

## Playbooks Included

- `linux_hardening.yml`: Multi-platform playbook that detects OS type and applies appropriate hardening configurations for Ubuntu and CentOS/RHEL systems

## Prerequisites

1. Ansible 2.9 or higher installed on the control node
2. SSH connectivity to the target hosts
3. Sudo/root access on the target hosts

## Setup

1. Create an inventory file with your servers:

```ini
# Example inventory.ini
[webservers]
web1.example.com
web2.example.com

[dbservers]
db1.example.com
db2.example.com

[centos_servers]
centos1.example.com
centos2.example.com

[ubuntu_servers]
ubuntu1.example.com
ubuntu2.example.com
```

2. Create the `templates` directory and add the chrony configuration template:

```bash
mkdir -p templates
```

3. Create the chrony configuration template `templates/chrony.conf.j2`:

```
{% if ansible_distribution == 'Ubuntu' %}
# Use Ubuntu NTP servers
pool ntp.ubuntu.com iburst maxsources 4
{% elif ansible_distribution == 'CentOS' or ansible_distribution == 'RedHat' %}
# Use CentOS NTP servers
server 0.centos.pool.ntp.org iburst
server 1.centos.pool.ntp.org iburst
server 2.centos.pool.ntp.org iburst
server 3.centos.pool.ntp.org iburst
{% else %}
# Use generic NTP servers
server 0.pool.ntp.org iburst
server 1.pool.ntp.org iburst
server 2.pool.ntp.org iburst
server 3.pool.ntp.org iburst
{% endif %}

# Record the rate at which the system clock gains/losses time.
driftfile /var/lib/chrony/drift

# Allow the system clock to be stepped in the first three updates
# if its offset is larger than 1 second.
makestep 1.0 3

# Enable kernel synchronization of the real-time clock (RTC).
rtcsync

# Specify directory for log files.
logdir /var/log/chrony
```

## Usage

### Apply to All Servers

Run the playbook against all servers in your inventory:

```bash
ansible-playbook -i inventory.ini linux_hardening.yml
```

### Apply to Specific Server Group

Run the playbook against a specific group in your inventory:

```bash
ansible-playbook -i inventory.ini linux_hardening.yml --limit webservers
```

### Apply to a Specific Server

Run the playbook against a single server:

```bash
ansible-playbook -i inventory.ini linux_hardening.yml --limit web1.example.com
```

### Dry Run Mode

To see what changes would be made without actually applying them:

```bash
ansible-playbook -i inventory.ini linux_hardening.yml --check
```

## Security Controls Implemented

The playbook implements the following CIS Benchmark sections:

1. **Initial Setup**
   - Filesystem Configuration
   - Software Updates
   - Process Hardening
   - Mandatory Access Control (SELinux)
   - Warning Banners

2. **Services**
   - Legacy Services Removal
   - Time Synchronization

3. **Network Configuration**
   - Network Parameters
   - Firewall Configuration (UFW for Ubuntu, firewalld for CentOS/RHEL)

4. **Logging and Auditing**
   - Configure System Accounting (auditd)
   - Configure rsyslog

5. **Access, Authentication and Authorization**
   - Configure cron
   - SSH Server Configuration
   - Password Settings
   - User Accounts and Environment

6. **System Maintenance**
   - System File Permissions
   - Software Updates

## Customization

The playbook uses variables defined at the top of the file. You can customize these settings by modifying the values or by creating a separate variables file and importing it.

Important variables you might want to customize:

- `ssh_permit_root_login`: Controls whether root can log in via SSH (default: "no")
- `pass_max_days`: Maximum password age (default: "90")
- `ssh_ciphers`: SSH ciphers allowed (defaults to strong ciphers only)
- `firewall_allowed_services`: Services to allow through the firewall (default: ssh only)

## Post-Hardening Verification

After running the playbook, you should verify that the systems are correctly hardened and that your applications still work as expected. You can use the scripts in the parent directory to perform manual verification of the hardening measures.

## Disclaimer

This playbook is provided as-is and should be tested in a non-production environment before applying to production systems. Some hardening measures may impact functionality of certain applications or services. 