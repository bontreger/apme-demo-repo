# Role: `pcp_analysis`

**Tags:** pillar:ops,lifecycle:day2,maturity:l2-process-oriented,platform:linux,tech:pcp  
**Supported OS:** RHEL/CentOS/Rocky/AlmaLinux/OracleLinux 7/8/9

Installs and enables Performance Co-Pilot (PCP) with `pmcd` and `pmlogger` services for system performance data collection and reporting.

## Requirements

- No special requirements beyond ansible-core 2.15+.

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `pcp_analysis_packages` | `["pcp", "pcp-system-tools"]` | List of PCP packages to install |
| `pcp_analysis_services` | `["pmcd", "pmlogger"]` | List of PCP services to start and enable |
| `pcp_analysis_report_namespace` | `"os_pcp"` | Report namespace for directory structure |

## Dependencies

None.

## Example Playbook

```yaml
---
- name: Install and configure PCP
  hosts: all_rhel
  become: true
  gather_facts: true
  roles:
    - role: pcp_analysis
```

## License

GPL-3.0-or-later

## Author

Red Hat Services
