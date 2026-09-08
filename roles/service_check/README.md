# Role: `service_check`

**Tags:** pillar:ops,lifecycle:day2,maturity:l2-process-oriented,platform:linux  
**Supported OS:** RHEL/CentOS/Rocky/AlmaLinux/OracleLinux 7/8/9/10

Checks the status of critical systemd services on RHEL hosts using bulk service facts collection and generates a per-host text report.

## Requirements

- No special requirements beyond ansible-core 2.15+.

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `service_check_critical_services_list` | `["sshd", "firewalld", "chronyd", "tuned", "NetworkManager", "rsyslog", "auditd", "crond", "dbus", "systemd-journald"]` | List of service names to check |
| `service_check_report_dir` | `/var/lib/ansible-reports` | Directory on the managed host for the report |
| `service_check_report_path` | `{{ service_check_report_dir }}/service_check-{{ inventory_hostname }}.txt` | Plaintext report path |

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
