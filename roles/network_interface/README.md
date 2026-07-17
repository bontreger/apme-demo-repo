# Role: `network_interface`

Checks a network interface's link status, IP address, MTU, and error/drop statistics using `ip` and `ethtool`, and produces a diagnostic summary.

## Requirements

- No special requirements beyond ansible-core 2.15+.

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `network_interface` | `"ens192"` | Primary network interface name to inspect |
| `network_interface_backup` | `"eth0"` | Backup network interface name |

## Dependencies

None.

## Example Playbook

```yaml
---
- name: RHEL Network Latency Diagnosis Automation
  hosts: all_rhel
  become: true
  gather_facts: true
  roles:
    - role: network_interface
      vars:
        network_interface: "ens192"
```

## License

GPL-3.0-or-later

## Author

Red Hat Services
