# Role: `system_config`

Validates hostname resolution (localhost and FQDN) and configures SELinux state and policy as part of RHEL9 system standardization.

## Requirements

- ansible-core 2.15+
- `ansible.posix` collection

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `network.allowed_subnets` | `["10.66.0.0/16", "192.168.0.0/16"]` | Allowed network subnets |
| `template_server.host` | `10.66.208.237` | Template server IP address |
| `template_server.hosts_file` | `/etc/hosts` | Path to the hosts file on the template server |
| `selinux.state` | `permissive` | Desired SELinux state (`enforcing`, `permissive`, or `disabled`) |
| `selinux.policy` | `targeted` | SELinux policy type |
| `backup.enabled` | `true` | Whether to create backups before modifying configuration files |
| `backup.suffix` | `.backup.$(date +%Y%m%d_%H%M%S)` | Suffix appended to backup filenames |

## Dependencies

None.

## Example Playbook

```yaml
---
- name: RHEL9 System Standardization Configuration
  hosts: rhel9
  gather_facts: true
  become: true
  roles:
    - role: system_config
      vars:
        selinux:
          state: permissive
          policy: targeted
```

## License

GPL-3.0-or-later

## Author

Red Hat Services
