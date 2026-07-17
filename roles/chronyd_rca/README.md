# Role: `chronyd_rca`

Performs root cause analysis diagnostics on the chronyd NTP service, collecting system facts, configuration validation, connectivity tests, and generating a text report.

## Requirements

- No special requirements beyond ansible-core 2.15+.

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `chronyd_rca_service` | `chronyd` | Name of the chronyd service |
| `chronyd_rca_package` | `chrony` | Name of the chrony package |
| `chronyd_rca_ntp_server` | `pool.ntp.org` | NTP server to test connectivity against |
| `chronyd_rca_conf_path` | `/etc/chrony.conf` | Path to the chrony configuration file |
| `chronyd_rca_log_dir` | `/var/log/chrony` | Path to the chrony log directory |
| `chronyd_rca_run_dir` | `/run/chrony` | Path to the chrony runtime directory |
| `chronyd_rca_collect_journal_lines` | `200` | Number of journal lines to collect for diagnostics |
| `chronyd_rca_attempt_service_start` | `false` | Whether to attempt starting the service during diagnosis |
| `chronyd_rca_report_namespace` | `chronyd_rca` | Namespace used for report file organization |
| `chronyd_rca_report_local_dir` | `/var/lib/ansible-reports/chronyd_rca` | Local directory for storing diagnosis reports |
| `chronyd_rca_report_job_id` | `{{ awx_job_id \| default(ansible_date_time.epoch) }}` | Unique identifier for the report file name |
| `chronyd_rca_report_text_path` | `<report_local_dir>/<hostname>-<job_id>.txt` | Full path for the generated text report |

## Dependencies

None.

## Example Playbook

```yaml
---
- name: Chronyd service fault diagnosis
  hosts: all_rhel
  become: true
  roles:
    - role: chronyd_rca
      vars:
        chronyd_rca_ntp_server: pool.ntp.org
        chronyd_rca_attempt_service_start: false
```

## License

GPL-3.0-or-later

## Author

Red Hat Services
