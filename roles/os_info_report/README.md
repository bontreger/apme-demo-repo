# Role: `os_info_report`

**Tags:** pillar:ops,lifecycle:day2,maturity:l2-process-oriented,platform:linux  
**Supported OS:** RHEL/CentOS/Rocky/AlmaLinux/OracleLinux 7/8/9/10

Collects comprehensive operating system information from managed hosts and generates plaintext reports.

## Requirements

- No special requirements beyond ansible-core 2.15+.

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `os_info_report_title` | `"System Information Report"` | Title displayed in the generated report |
| `os_info_report_dir` | `/var/lib/ansible-reports` | Directory on the managed host for the report |
| `os_info_report_path` | `{{ os_info_report_dir }}/os_info_report-{{ inventory_hostname }}.txt` | Plaintext report path (overwritten each run) |

## Dependencies

None.

## Example Playbook

```yaml
---
- name: Collect OS information and generate reports
  hosts: all_rhel
  become: true
  gather_facts: true
  roles:
    - role: os_info_report
```

## License

GPL-3.0-or-later

## Author

Red Hat Services
