# Role: `os_info_report`

Collects comprehensive operating system information from managed hosts and generates plaintext and optional HTML reports.

## Requirements

- No special requirements beyond ansible-core 2.15+.

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `os_info_report_title` | `"System Information Report"` | Title displayed in the generated report |
| `os_info_report_namespace` | `"os_info"` | Report namespace for directory structure |
| `os_info_report_local_dir` | `"/var/lib/ansible-reports/{{ os_info_report_namespace }}"` | Local directory for generated reports |
| `os_info_report_job_id` | `"{{ awx_job_id \| default(ansible_facts['date_time'].epoch) }}"` | Unique job identifier for report filenames |
| `os_info_report_text_path` | `"{{ os_info_report_local_dir }}/{{ inventory_hostname }}-{{ ... }}.txt"` | Full path for the plaintext report |
| `os_info_report_html_path` | `"{{ os_info_report_local_dir }}/{{ inventory_hostname }}-{{ ... }}.html"` | Full path for the HTML report |
| `os_info_report_generate_html` | `false` | Whether to generate an HTML report in addition to plaintext |

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
      vars:
        os_info_report_generate_html: true
```

## License

GPL-3.0-or-later

## Author

Red Hat Services
