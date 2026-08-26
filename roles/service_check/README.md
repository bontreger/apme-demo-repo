# Role: `service_check`

**Tags:** pillar:ops,lifecycle:day2,maturity:l2,platform:linux  
**Supported OS:** RHEL/CentOS/Rocky/AlmaLinux/OracleLinux 7/8/9

Checks the status of critical systemd services on RHEL hosts using bulk service facts collection and generates a per-host text report.

## Requirements

- No special requirements beyond ansible-core 2.15+.

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `service_check_critical_services_list` | `["sshd", "firewalld", "chronyd", "tuned", "NetworkManager", "rsyslog", "auditd", "crond", "dbus", "systemd-journald"]` | List of service names to check |
| `report_namespace` | `service_check` | Namespace used to organize reports by category |
| `report_local_dir` | `/var/lib/ansible-reports/{{ report_namespace }}` | Local directory on the target host where reports are stored |
| `report_job_id` | `{{ awx_job_id \| default(ansible_date_time.epoch) }}` | Unique identifier for the report run |
| `report_text_path` | `{{ report_local_dir }}/{{ inventory_hostname }}-{{ report_job_id }}.txt` | Full path to the generated text report |
| `service_check_report_base_path` | `{{ report_local_dir }}` | Deprecated alias for report directory used by validation tasks |
| `service_check_collection_dest_path` | `""` | Deprecated alias for report collection destination |

## Dependencies

None.

## Example Playbook

```yaml
---
- name: Inspect critical Linux service status and generate reports
  hosts: all_rhel
  become: true
  gather_facts: true
  roles:
    - role: service_check
      vars:
        service_check_critical_services_list:
          - sshd
          - firewalld
          - chronyd
```

## License

GPL-3.0-or-later

## Author

Red Hat Services
