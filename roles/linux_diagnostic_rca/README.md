# Role: `linux_diagnostic_rca`

**Tags:** pillar:ops,lifecycle:day2,maturity:l2,platform:linux  
**Supported OS:** RHEL/CentOS/Rocky/AlmaLinux/OracleLinux 7/8/9

Performs targeted root cause analysis on Linux systems for zombie processes, disk I/O pressure, failed systemd units, or privileged account auditing, and generates JSON and plaintext investigation reports.

## Requirements

- No special requirements beyond ansible-core 2.15+.

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `linux_diagnostic_rca_check_type` | `""` | Type of diagnostic check: `zombie_process`, `disk_io`, `systemd_unit`, or `privileged_account` |
| `linux_diagnostic_rca_namespace_map` | `{zombie_process: zombie_process_rca, disk_io: disk_io_pressure_rca, systemd_unit: systemd_unit_deep_check, privileged_account: privileged_account_audit}` | Maps check types to report namespace names |
| `linux_diagnostic_rca_report_namespace` | `"{{ linux_diagnostic_rca_namespace_map[...] }}"` | Resolved report namespace based on check type |
| `linux_diagnostic_rca_report_local_dir` | `"/var/lib/ansible-reports/{{ linux_diagnostic_rca_report_namespace }}"` | Local directory for generated reports |
| `linux_diagnostic_rca_report_job_id` | `"{{ awx_job_id \| default(ansible_date_time.epoch) }}"` | Unique job identifier for report filenames |
| `linux_diagnostic_rca_report_text_path` | `"{{ linux_diagnostic_rca_report_local_dir }}/{{ inventory_hostname }}-{{ ... }}.txt"` | Full path for the plaintext report |
| `linux_diagnostic_rca_report_json_path` | `"{{ linux_diagnostic_rca_report_local_dir }}/{{ inventory_hostname }}-{{ ... }}.json"` | Full path for the JSON report |
| `linux_diagnostic_rca_log_lines` | `20` | Number of journal lines to collect per failed systemd unit |
| `linux_diagnostic_rca_fail_on_critical` | `false` | Abort the play on critical findings during privileged account audit |

## Dependencies

None.

## Example Playbook

```yaml
---
- name: Zombie process root cause analysis
  hosts: all_rhel
  become: true
  gather_facts: true
  roles:
    - role: linux_diagnostic_rca
      vars:
        linux_diagnostic_rca_check_type: zombie_process
```

## License

GPL-3.0-or-later

## Author

Red Hat Services
