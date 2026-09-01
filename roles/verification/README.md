# Role: `verification`

**Tags:** pillar:os,lifecycle:day1,maturity:l2-process-oriented,platform:linux,context:rhel9-only  
**Supported OS:** RHEL 9 only

Verifies post-configuration state of a RHEL system including user accounts, network connectivity, service health, listening ports, and time synchronization.

## Requirements

- No special requirements beyond ansible-core 2.15+.
- Expects variables from companion roles (e.g. `user_management`, `firewall_config`) to be defined.

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `admin_user` | *(set by user_management role)* | Username to verify existence and sudo privileges |
| `firewall_ports` | *(set by firewall_config role)* | List of ports to verify in listening state |

## Dependencies

Designed to run after `user_management`, `firewall_config`, and `chrony_config` roles within the `linux-rhel9-standard-config` playbook.

## Example Playbook

```yaml
---
- name: RHEL9 System Standardization Configuration
  hosts: rhel9
  become: true
  roles:
    - role: user_management
    - role: system_config
    - role: firewall_config
    - role: hosts_config
    - role: chrony_config
    - role: system_optimization
    - role: verification
```

## License

GPL-3.0-or-later

## Author

Red Hat Services
