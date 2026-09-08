# Role: `report_generation`

**Tags:** pillar:network,lifecycle:day2,maturity:l2-process-oriented,platform:linux  
**Supported OS:** RHEL/CentOS 7/8/9/10

Generates a network latency analysis report from collected diagnostic data using a Jinja2 template and writes it to a configurable local directory on the target host.

## Requirements

- No special requirements beyond ansible-core 2.15+.

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `report_generation_report_dir` | `/var/lib/ansible-reports` | Directory on the managed host for the report |
| `report_generation_report_path` | `{{ report_generation_report_dir }}/report_generation-{{ inventory_hostname }}.txt` | Plaintext report path |

## Dependencies

None.

## Example Playbook

```yaml
---
- name: RHEL Network Latency Diagnosis Automation
  hosts: all_rhel
  gather_facts: true
  become: true
  roles:
    - role: network_interface
    - role: system_diagnosis
    - role: traffic_capture
    - role: report_generation
```

## License

GPL-3.0-or-later

## Author

Red Hat Services
