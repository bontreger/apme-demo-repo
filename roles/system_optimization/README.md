# Role: `system_optimization`

Configures system resource limits in `/etc/security/limits.conf` and kernel sysctl parameters via `/etc/sysctl.d/99-ansible.conf` as part of RHEL9 system standardization.

## Requirements

- ansible-core 2.15+
- `ansible.posix` collection

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `system_limits` | `["prometheus soft nofile 65536", "prometheus hard nofile 65536", "* soft nofile 65536", "* hard nofile 65536"]` | List of entries to add to `/etc/security/limits.conf` |
| `sysctl_params` | `{"net.core.somaxconn": 65535, "net.ipv4.tcp_max_syn_backlog": 65535, "vm.max_map_count": 262144}` | Dictionary of sysctl key-value pairs to apply |

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
    - role: system_optimization
      vars:
        system_limits:
          - "* soft nofile 65536"
          - "* hard nofile 65536"
        sysctl_params:
          net.core.somaxconn: 65535
          vm.max_map_count: 262144
```

## License

GPL-3.0-or-later

## Author

Red Hat Services
