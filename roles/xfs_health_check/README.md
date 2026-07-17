# Role: `xfs_health_check`

Inspects all XFS filesystems on a host by running metadata consistency checks, collecting disk/inode usage, reviewing kernel logs for XFS errors, and generating a plaintext diagnostic report.

## Requirements

- No special requirements beyond ansible-core 2.15+.
- `xfsprogs` package recommended on target hosts (for `xfs_db`).

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `xfs_health_check_namespace` | `xfs_check` | Namespace identifier used for report directory naming |
| `xfs_health_check_local_dir` | `/var/lib/ansible-reports/{{ xfs_health_check_namespace }}` | Directory to store generated report files |
| `xfs_health_check_job_id` | `{{ awx_job_id \| default(ansible_date_time.epoch) }}` | Unique identifier for the report file name |
| `xfs_health_check_report_text_path` | `{{ xfs_health_check_local_dir }}/{{ inventory_hostname }}-{{ xfs_health_check_job_id }}.txt` | Full path to the rendered plaintext report |

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
