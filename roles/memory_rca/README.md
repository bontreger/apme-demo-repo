# Role: `memory_rca`

**Tags:** pillar:ops,lifecycle:day2,maturity:l2-process-oriented,platform:linux  
**Supported OS:** RHEL/CentOS/Rocky/AlmaLinux/OracleLinux 7/8/9/10

Performs memory root cause analysis for high utilization or OOM events by collecting system memory data, hardware info, and virtual memory statistics, then generates a diagnostic report.

## Requirements

- No special requirements beyond ansible-core 2.15+.

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `memory_rca_mode` | `"high_utilization"` | Analysis mode: `high_utilization` or `oom` |
| `memory_rca_collect_journal_lines` | `200` | Number of journal lines to collect for analysis |
| `memory_rca_report_dir` | `/var/lib/ansible-reports` | Directory on the managed host for the report |
| `memory_rca_report_path` | `{{ memory_rca_report_dir }}/memory_rca-{{ inventory_hostname }}.txt` | Plaintext report path |

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
