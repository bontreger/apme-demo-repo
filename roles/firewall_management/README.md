# Role: `firewall_management`

**Tags:** pillar:security,lifecycle:day1,maturity:l2,platform:linux,tech:firewall  
**Supported OS:** RHEL/CentOS/Rocky/AlmaLinux/OracleLinux 7/8/9

Manages the firewalld service lifecycle (start, stop, or check status) with validation and status reporting.

## Requirements

- No special requirements beyond ansible-core 2.15+.

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `firewall_management_package_name` | `firewalld` | Name of the firewall package to install |
| `firewall_management_service_name` | `firewalld` | Name of the firewall service to manage |
| `firewall_management_action` | `check` | Action to perform: 'start', 'stop', or 'check' |

## Dependencies

None.

## Example Playbook

```yaml
---
- name: Firewall management
  hosts: all_rhel
  become: true
  roles:
    - role: firewall_management
      vars:
        firewall_management_action: "start"
```

## License

GPL-3.0-or-later

## Author

Red Hat Services
