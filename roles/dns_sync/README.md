# Role: `dns_sync`

**Tags:** pillar:network,lifecycle:day2,maturity:l2-process-oriented,platform:linux,tech:bind  
**Supported OS:** RHEL/CentOS/Rocky/AlmaLinux/OracleLinux 7/8/9

Synchronizes a DNS resolver configuration file (resolv.conf) from the control node to target hosts with validation, backup, and rollback on failure.

## Requirements

- No special requirements beyond ansible-core 2.15+.

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `dns_sync_source_file_path` | `/etc/resolv.conf` | Path to the source resolv.conf on the control node |
| `dns_sync_dest_file_path` | `/etc/resolv.conf` | Destination path on target hosts |
| `dns_sync_backup_enabled` | `true` | Whether to create a backup before overwriting |
| `dns_sync_file_owner` | `root` | Owner of the deployed file |
| `dns_sync_file_group` | `root` | Group of the deployed file |
| `dns_sync_file_mode` | `0644` | File permissions for the deployed file |

## Dependencies

None.

## Example Playbook

```yaml
---
- name: Synchronize DNS configuration files across network
  hosts: all_rhel
  become: true
  roles:
    - role: dns_sync
      vars:
        dns_sync_source_file_path: "/etc/resolv.conf"
        dns_sync_backup_enabled: true
```

## License

GPL-3.0-or-later

## Author

Red Hat Services
