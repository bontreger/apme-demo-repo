# Role: `kernel_log_monitor`

**Tags:** pillar:ops,lifecycle:day2,maturity:l2-process-oriented,platform:linux  
**Supported OS:** RHEL/CentOS/Rocky/AlmaLinux/OracleLinux 7/8/9/10

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
| `kernel_log_monitor_report_dir` | `/var/lib/ansible-reports` | Directory on the managed host for the report |
| `kernel_log_monitor_report_path` | `{{ kernel_log_monitor_report_dir }}/kernel_log_monitor-{{ inventory_hostname }}.txt` | Plaintext report path |

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
