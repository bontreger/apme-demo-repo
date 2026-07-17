# Role: `report_generation`

Generates a network latency analysis report from collected diagnostic data using a Jinja2 template and writes it to a configurable local directory on the target host.

## Requirements

- No special requirements beyond ansible-core 2.15+.

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `report_namespace` | `network_diagnosis` | Namespace used to organize reports by category |
| `report_local_dir` | `/var/lib/ansible-reports/{{ report_namespace }}` | Local directory on the target host where reports are stored |
| `report_job_id` | `{{ awx_job_id \| default(ansible_date_time.epoch) }}` | Unique identifier for the report (AAP job ID or epoch timestamp) |
| `report_text_path` | `{{ report_local_dir }}/{{ inventory_hostname }}-{{ report_job_id }}.txt` | Full path to the generated text report file |
| `report_output_dir` | `{{ report_local_dir }}` | Directory where the rendered report file is written |
| `report_filename` | `{{ inventory_hostname }}-{{ report_job_id }}.txt` | Filename of the generated report |

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
