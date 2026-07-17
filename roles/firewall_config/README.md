# Role: `firewall_config`

Ensures firewalld is running and opens a configurable list of TCP and UDP ports, with service reload and verification.

## Requirements

- No special requirements beyond ansible-core 2.15+.
- `ansible.posix` (for `firewalld` module)

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `firewall_ports` | `[50051, 53, 80, 443, 5432, 6379, 27199, 9000, 9001, 9002, 9003]` | List of ports to open in firewalld (both TCP and UDP) |

## Dependencies

None.

## Example Playbook

```yaml
---
- name: RHEL9 System Standardization Configuration
  hosts: rhel9
  become: true
  roles:
    - role: firewall_config
      vars:
        firewall_ports:
          - 80
          - 443
          - 8080
```

## License

GPL-3.0-or-later

## Author

Red Hat Services
