# Ansible Security Hardening

Ansible playbooks for automated Linux server hardening based on CIS Benchmarks.

## Why This Exists

Every time I spun up a new Linux box, I was doing the same 20 things manually:
- Disable root SSH
- Configure firewall
- Set up fail2ban
- Harden SSH config
- Remove unnecessary services

That's dumb. Ansible can do this in 5 minutes across 100 servers.

## What's In Here

```
ansible-security-hardening/
├── playbooks/
│   ├── full_hardening.yml     # Run everything
│   ├── ssh_hardening.yml      # SSH config only
│   └── firewall.yml           # UFW/firewalld setup
├── roles/
│   └── hardening/
│       ├── tasks/
│       │   ├── main.yml
│       │   ├── ssh.yml
│       │   ├── firewall.yml
│       │   ├── users.yml
│       │   └── services.yml
│       ├── handlers/
│       │   └── main.yml
│       ├── templates/
│       │   └── sshd_config.j2
│       └── defaults/
│           └── main.yml
├── inventory/
│   └── hosts.example
└── ansible.cfg
```

## Prerequisites

```bash
# Install Ansible
pip install ansible --break-system-packages

# Or on macOS
brew install ansible

# Verify
ansible --version

# You need SSH access to target servers
# Test with:
ssh user@your-server hostname
```

## Quick Start

```bash
# Clone the repo
git clone https://github.com/fbabalola/ansible-security-hardening.git
cd ansible-security-hardening

# Copy and edit inventory
cp inventory/hosts.example inventory/hosts
nano inventory/hosts
# Add your server IPs

# Test connectivity (no changes made)
ansible all -i inventory/hosts -m ping

# Dry run - see what WOULD change
ansible-playbook playbooks/full_hardening.yml -i inventory/hosts --check

# Actually run it
ansible-playbook playbooks/full_hardening.yml -i inventory/hosts
```

## What Gets Hardened

### SSH Configuration (CIS 5.2)
- Disable root login
- Disable password authentication (key-only)
- Reduce login grace time
- Set max auth tries to 3
- Disable X11 forwarding
- Set idle timeout

### Firewall (CIS 3.5)
- Enable UFW (Ubuntu) or firewalld (RHEL/CentOS)
- Default deny incoming
- Allow SSH from trusted networks only
- Allow specific ports you define

### User Accounts (CIS 5.4, 5.5)
- Set password expiration policies
- Lock inactive accounts
- Remove unnecessary users
- Set proper permissions on home directories

### Services (CIS 2.1, 2.2)
- Disable unused services
- Remove unnecessary packages
- Enable automatic security updates

## Configuration

Edit `roles/hardening/defaults/main.yml`:

```yaml
# SSH settings
ssh_port: 22
ssh_permit_root_login: "no"
ssh_password_authentication: "no"
ssh_max_auth_tries: 3
ssh_client_alive_interval: 300
ssh_client_alive_count_max: 2

# Allowed SSH networks (CIDR notation)
ssh_allowed_networks:
  - "10.0.0.0/8"
  - "192.168.1.0/24"
  # Add your IP here

# Firewall - additional ports to allow
firewall_allowed_ports:
  - { port: 80, proto: tcp }
  - { port: 443, proto: tcp }

# Services to disable
disable_services:
  - cups
  - avahi-daemon
  - rpcbind
```

## Run Specific Playbooks

```bash
# Just SSH hardening
ansible-playbook playbooks/ssh_hardening.yml -i inventory/hosts

# Just firewall
ansible-playbook playbooks/firewall.yml -i inventory/hosts

# Target specific hosts
ansible-playbook playbooks/full_hardening.yml -i inventory/hosts --limit webservers
```

## Testing Changes First

**Always run with `--check` first:**

```bash
# Dry run - shows what would change without doing it
ansible-playbook playbooks/full_hardening.yml -i inventory/hosts --check --diff
```

**Test on one server first:**

```bash
# Limit to single host
ansible-playbook playbooks/full_hardening.yml -i inventory/hosts --limit 192.168.1.10
```

## Inventory Example

```ini
# inventory/hosts

[webservers]
web1 ansible_host=192.168.1.10
web2 ansible_host=192.168.1.11

[databases]
db1 ansible_host=192.168.1.20

[all:vars]
ansible_user=admin
ansible_become=yes
ansible_become_method=sudo
```

## CIS Benchmark Mapping

| Playbook Task | CIS Control | Description |
|---------------|-------------|-------------|
| SSH hardening | 5.2.x | Secure SSH configuration |
| Firewall setup | 3.5.x | Network firewall rules |
| Service disable | 2.1.x, 2.2.x | Remove unnecessary services |
| Password policy | 5.4.x | Account security settings |
| Filesystem perms | 6.1.x | File permission hardening |

## Troubleshooting

**"Permission denied"**
```bash
# Make sure you can SSH manually first
ssh -v user@server

# Check if sudo works
ssh user@server "sudo whoami"
```

**"SSH connection timed out after lockout"**
```bash
# You probably locked yourself out by restricting SSH too much
# Before running, make sure your IP is in ssh_allowed_networks
# Or keep a console session open as backup
```

**"Package not found"**
```bash
# Different distros have different package names
# Check you're targeting the right OS
# The playbooks detect Ubuntu vs CentOS/RHEL automatically
```

## Don't Lock Yourself Out

⚠️ **Before running SSH hardening:**

1. Make sure your IP is in `ssh_allowed_networks`
2. Keep an existing SSH session open
3. Test on one server first
4. Have console access as backup (especially for cloud VMs)

## Author

Firebami Babalola  
Security Operations Analyst | SC-200 | Security+

Built this after manually hardening the same servers too many times.
