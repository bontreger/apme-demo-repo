# Role: `firewall_config`

**Tags:** pillar:os,lifecycle:day1,maturity:l2-process-oriented,platform:linux,context:rhel9-only  
**Supported OS:** RHEL 9 only

Installs firewalld, ensures it is running, and opens a configurable list of TCP and UDP ports, with service reload and verification.

## Requirements

- No special requirements beyond ansible-core 2.15+.
- `ansible.posix` (for `firewalld` module)
- Package repositories reachable from the host; the role installs the
  `firewalld` package if it is absent, as `chrony_config` does for chrony.
  Without it the role has nothing to configure and skips every port.

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
