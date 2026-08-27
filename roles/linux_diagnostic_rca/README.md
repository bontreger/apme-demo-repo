# Role: `linux_diagnostic_rca`

**Tags:** pillar:ops,lifecycle:day2,maturity:l2-process-oriented,platform:linux  
**Supported OS:** RHEL/CentOS/Rocky/AlmaLinux/OracleLinux 7/8/9/10

Performs targeted root cause analysis on Linux systems for zombie processes, disk I/O pressure, failed systemd units, or privileged account auditing, and generates JSON and plaintext investigation reports.

## Requirements

- No special requirements beyond ansible-core 2.15+.

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `linux_diagnostic_rca_check_type` | `""` | Type of diagnostic check: `zombie_process`, `disk_io`, `systemd_unit`, or `privileged_account` |
| `linux_diagnostic_rca_report_dir` | `/var/lib/ansible-reports` | Directory on the managed host for reports |
| `linux_diagnostic_rca_report_path` | `{{ linux_diagnostic_rca_report_dir }}/linux_diagnostic_rca-<check_type>-{{ inventory_hostname }}.txt` | Plaintext report path |
| `linux_diagnostic_rca_report_json_path` | same stem with `.json` | JSON investigation report |
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
