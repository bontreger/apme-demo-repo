# Role: `filesystem_check`

Performs read-only filesystem health checks (xfs_scrub, fsck -n) on mounted filesystems and generates HTML/text reports with disk space and inode usage.

## Requirements

- No special requirements beyond ansible-core 2.15+.
- RHEL/CentOS 7, 8, or 9

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `filesystem_check_exclude_fs_types` | `["tmpfs", "devtmpfs", "iso9660", "squashfs", "overlay", "nfs", "cifs", "sysfs", "proc", "devfs", "autofs"]` | Filesystem types to exclude from health checks |
| `filesystem_check_exclude_mount_points` | `["/"]` | Mount points to exclude from health checks |
| `report_namespace` | `filesystem_check` | Namespace for report file organization |
| `report_local_dir` | `/var/lib/ansible-reports/filesystem_check` | Local directory for storing reports |
| `report_job_id` | `{{ awx_job_id \| default(ansible_date_time.epoch) }}` | Unique identifier for report file names |
| `report_text_path` | `<report_local_dir>/<hostname>-<job_id>.txt` | Full path for the generated text report |
| `report_html_path` | `<report_local_dir>/<hostname>-<job_id>.html` | Full path for the generated HTML report |
| `filesystem_check_report_base_path` | `{{ report_local_dir }}` | Deprecated alias for report_local_dir |
| `filesystem_check_collection_dest_path` | `""` | Deprecated collection destination path |

## Dependencies

None.

## Example Playbook

```yaml
---
- name: Check filesystem health and generate reports
  hosts: all_rhel
  become: true
  roles:
    - role: filesystem_check
      vars:
        filesystem_check_exclude_mount_points:
          - /
```

## License

GPL-3.0-or-later

## Author

Red Hat Services
