# Role: `web_service_rca`

**Tags:** pillar:ops,lifecycle:day2,maturity:l2-process-oriented,platform:linux  
**Supported OS:** RHEL/CentOS/Rocky/AlmaLinux/OracleLinux 8/9/10

Performs root-cause analysis on a failed web service (httpd or nginx) by checking package state, configuration syntax, SELinux booleans, firewall rules, listening ports, and log output, then generates a diagnostic report.

## Requirements

- No special requirements beyond ansible-core 2.15+.
  SELinux and firewall state are gathered via `ansible.builtin.command`
  (`getenforce`, `getsebool`, `firewall-cmd`); no extra collections.

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `web_service_rca_service` | `httpd` | Web service to diagnose (`httpd` or `nginx`) |
| `web_service_rca_attempt_service_start` | `false` | Whether to attempt starting the service during diagnosis |
| `web_service_rca_collect_journal_lines` | `200` | Number of log/journal lines to collect for analysis |
| `web_service_rca_profiles` | *(see defaults)* | Dictionary of service profiles with paths, ports, and SELinux booleans |
| `web_service_rca_report_namespace` | `{{ web_service_rca_service }}_rca` | Namespace used for report directory naming |
| `web_service_rca_report_local_dir` | `/var/lib/ansible-reports/{{ web_service_rca_report_namespace }}` | Local directory to store generated reports |
| `web_service_rca_report_job_id` | `{{ awx_job_id \| default(ansible_date_time.epoch) }}` | Unique identifier for the report file |
| `web_service_rca_report_text_path` | `{{ web_service_rca_report_local_dir }}/{{ inventory_hostname }}-{{ web_service_rca_report_job_id }}.txt` | Full path to the rendered report file |

## Dependencies

None.

## Example Playbook

```yaml
---
- name: Web service fault diagnosis (httpd/nginx)
  hosts: all_rhel
  become: true
  roles:
    - role: web_service_rca
      vars:
        web_service_rca_service: nginx
        web_service_rca_attempt_service_start: true
```

## License

GPL-3.0-or-later

## Author

Red Hat Services
