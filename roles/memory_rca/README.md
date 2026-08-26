# Role: `memory_rca`

**Tags:** pillar:ops,lifecycle:day2,maturity:l2,platform:linux  
**Supported OS:** RHEL/CentOS/Rocky/AlmaLinux/OracleLinux 7/8/9

Performs memory root cause analysis for high utilization or OOM events by collecting system memory data, hardware info, and virtual memory statistics, then generates a diagnostic report.

## Requirements

- No special requirements beyond ansible-core 2.15+.

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `memory_rca_mode` | `"high_utilization"` | Analysis mode: `high_utilization` or `oom` |
| `memory_rca_collect_journal_lines` | `200` | Number of journal lines to collect for analysis |
| `memory_rca_report_namespace` | `"memory_{{ memory_rca_mode }}"` | Report namespace derived from the selected mode |
| `memory_rca_report_local_dir` | `"/var/lib/ansible-reports/{{ memory_rca_report_namespace }}"` | Local directory for generated reports |
| `memory_rca_report_job_id` | `"{{ awx_job_id \| default(ansible_date_time.epoch) }}"` | Unique job identifier for report filenames |
| `memory_rca_report_text_path` | `"{{ memory_rca_report_local_dir }}/{{ inventory_hostname }}-{{ ... }}.txt"` | Full path for the plaintext report |

## Dependencies

None.

## Example Playbook

```yaml
---
- name: Memory diagnosis (high utilization / OOM)
  hosts: all_rhel
  become: true
  gather_facts: true
  roles:
    - role: memory_rca
      vars:
        memory_rca_mode: high_utilization
```

## License

GPL-3.0-or-later

## Author

Red Hat Services
