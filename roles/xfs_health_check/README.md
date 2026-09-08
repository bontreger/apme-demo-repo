# Role: `xfs_health_check`

**Tags:** pillar:ops,lifecycle:day2,maturity:l2-process-oriented,platform:linux  
**Supported OS:** RHEL/CentOS/Rocky/AlmaLinux/OracleLinux 7/8/9/10

Inspects all XFS filesystems on a host by running metadata consistency checks, collecting disk/inode usage, reviewing kernel logs for XFS errors, and generating a plaintext diagnostic report.

## Requirements

- No special requirements beyond ansible-core 2.15+.
- `xfsprogs` package recommended on target hosts (for `xfs_db`).

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `xfs_health_check_report_dir` | `/var/lib/ansible-reports` | Directory on the managed host for the report |
| `xfs_health_check_report_path` | `{{ xfs_health_check_report_dir }}/xfs_health_check-{{ inventory_hostname }}.txt` | Plaintext report path |

## Dependencies

None.

## Example Playbook

```yaml
---
- name: XFS filesystem health inspection
  hosts: all_rhel
  become: true
  roles:
    - role: xfs_health_check
```

## License

GPL-3.0-or-later

## Author

Red Hat Services
