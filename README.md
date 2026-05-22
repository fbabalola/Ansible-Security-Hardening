# Ansible Security Hardening

Ansible playbooks for repeatable Linux hardening practice based on common CIS Benchmark ideas.

I built this because I kept doing the same Linux hardening steps manually: locking down SSH, setting firewall defaults, cleaning up unnecessary services, and checking basic account settings. Ansible gives me a way to apply the same checklist consistently across multiple Linux hosts.

This is a lab hardening baseline, not a one-size-fits-all production standard. The point is to show how I think through Linux hardening and configuration management.

## What is in here

```text
ansible-security-hardening/
├── playbooks/
│   ├── full_hardening.yml
│   ├── ssh_hardening.yml
│   └── firewall.yml
├── roles/
│   └── hardening/
│       ├── tasks/
│       ├── handlers/
│       ├── templates/
│       └── defaults/
├── inventory/
│   └── hosts.example
└── ansible.cfg
```

## Prerequisites

```bash
# Install Ansible
pip install ansible

# Or on macOS
brew install ansible

ansible --version
```

You also need SSH access to your target host:

```bash
ssh user@your-server hostname
```

## Quick start

```bash
git clone https://github.com/fbabalola/Ansible-Security-Hardening.git
cd Ansible-Security-Hardening

cp inventory/hosts.example inventory/hosts
nano inventory/hosts

ansible all -i inventory/hosts -m ping
ansible-playbook playbooks/full_hardening.yml -i inventory/hosts --check --diff
ansible-playbook playbooks/full_hardening.yml -i inventory/hosts
```

## What gets hardened

### SSH configuration

- Disable root login
- Disable password authentication when key-based access is ready
- Reduce login grace time
- Limit authentication attempts
- Disable X11 forwarding
- Set idle timeout values

### Firewall basics

- Enable UFW or firewalld depending on distro
- Default deny incoming traffic
- Allow SSH from trusted networks only
- Allow only the ports needed for the host role

### User and service cleanup

- Review password expiration settings
- Lock inactive accounts where appropriate
- Remove or disable unnecessary services
- Apply safer permissions where possible

## Testing changes safely

Always test first:

```bash
ansible-playbook playbooks/full_hardening.yml -i inventory/hosts --check --diff
```

Test on one host before running against a group:

```bash
ansible-playbook playbooks/full_hardening.yml -i inventory/hosts --limit 192.168.1.10
```

## Do not lock yourself out

Before running SSH hardening:

1. Make sure your current IP is allowed.
2. Keep an existing SSH session open.
3. Test on one host first.
4. Have console or cloud dashboard access as a backup.

## CIS-style mapping

| Area | Example control idea | What this repo practices |
|---|---|---|
| SSH hardening | Secure remote access | Safer SSH configuration |
| Firewall setup | Limit network exposure | Default-deny style rules |
| Service cleanup | Reduce attack surface | Disable unnecessary services |
| Account settings | Improve account hygiene | Review inactive users and permissions |

## Known limitations

- This is a lab hardening baseline, not a complete production policy.
- SSH and firewall changes should always be tested with `--check` first.
- Some tasks may need adjustment depending on Ubuntu, CentOS, or RHEL versions.
- Real environments may require exceptions for business or application needs.

## Author

Firebami Babalola  
Security Operations | SC-200 | Security+ | Linux Hardening Practice
