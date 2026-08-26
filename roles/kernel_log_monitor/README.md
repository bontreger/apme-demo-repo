# Role: `kernel_log_monitor`

**Tags:** pillar:ops,lifecycle:day2,maturity:l2-process-oriented,platform:linux  
**Supported OS:** RHEL/CentOS/Rocky/AlmaLinux/OracleLinux 7/8/9

Monitors kernel logs for critical and severe errors, classifies detected issues, triggers multi-level alerts, and generates plaintext and HTML diagnostic reports.

## Requirements

- No special requirements beyond ansible-core 2.15+.

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `kernel_log_monitor_kernel_errors.critical` | `["Call Trace", "panic", "hung_task_timeout_secs"]` | List of regex patterns classified as critical errors |
| `kernel_log_monitor_kernel_errors.severe` | `["oom", "segfault", "hardware error"]` | List of regex patterns classified as severe errors |
| `kernel_log_monitor_alert_message` | `"System detected kernel-level anomalies..."` | Alert message broadcast when errors are found |
| `kernel_log_monitor_log_lines` | `100` | Number of recent kernel log lines to capture |
| `kernel_log_monitor_kernel_events_log` | `"/var/log/kernel_events.log"` | Path to the kernel events log file for priority alerts |
| `report_namespace` | `"kernel_log_monitor"` | Namespace used for report directory structure |
| `report_local_dir` | `"/var/lib/ansible-reports/{{ report_namespace }}"` | Local directory for generated reports |
| `report_job_id` | `"{{ awx_job_id \| default(ansible_facts['date_time'].epoch) }}"` | Unique job identifier for report filenames |
| `report_text_path` | `"{{ report_local_dir }}/{{ inventory_hostname }}-{{ report_job_id }}.txt"` | Full path for the plaintext report |
| `report_html_path` | `"{{ report_local_dir }}/{{ inventory_hostname }}-{{ report_job_id }}.html"` | Full path for the HTML report |
| `kernel_log_monitor_report_base_path` | `"{{ report_local_dir }}"` | Deprecated alias for `report_local_dir` |
| `kernel_log_monitor_collection_dest_path` | `""` | Deprecated collection destination path |

## Dependencies

None.

## Example Playbook

```yaml
---
- name: Monitor kernel logs for errors and generate reports
  hosts: all_rhel
  become: true
  gather_facts: true
  roles:
    - role: kernel_log_monitor
      vars:
        kernel_log_monitor_log_lines: 200
```

## License

GPL-3.0-or-later

## Author

Red Hat Services
