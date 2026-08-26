# Role: `chrony_config`

**Tags:** pillar:os,lifecycle:day1,maturity:l2,platform:linux,context:rhel9-only  
**Supported OS:** RHEL 9 only

Installs and configures Chrony for NTP time synchronization, deploying a templated chrony.conf and enabling the chronyd service.

## Requirements

- No special requirements beyond ansible-core 2.15+.

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `ntp_servers` | `["ntp.aliyun.com", "ntp1.aliyun.com", "ntp2.aliyun.com"]` | List of NTP servers to configure in chrony.conf |

## Dependencies

None.

## Example Playbook

```yaml
---
- name: RHEL9 System Standardization Configuration
  hosts: rhel9
  become: true
  roles:
    - role: chrony_config
      vars:
        ntp_servers:
          - ntp.example.com
          - ntp2.example.com
```

## License

GPL-3.0-or-later

## Author

Red Hat Services
