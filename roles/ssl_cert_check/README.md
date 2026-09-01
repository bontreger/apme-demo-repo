# Role: `ssl_cert_check`

**Tags:** pillar:security,lifecycle:day2,maturity:l2-process-oriented,platform:linux,tech:ssl,context:localhost  
**Supported OS:** Any (runs on controller localhost)

Checks SSL/TLS certificate expiry dates for a list of remote sites and generates a Markdown report with warning and critical thresholds.

## Requirements

- ansible-core 2.15+
- `community.crypto` collection

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `ssl_cert_check_sites` | `["www.google.com", "github.com"]` | List of hostnames whose SSL certificates will be checked |
| `ssl_cert_check_warn_days` | `30` | Number of days before expiry to trigger a warning |
| `ssl_cert_check_critical_days` | `10` | Number of days before expiry to trigger a critical alert |
| `ssl_cert_check_namespace` | `ssl_expiry` | Report namespace for organizing published reports |
| `ssl_cert_check_job_id` | `{{ awx_job_id \| default(ansible_date_time.epoch) }}` | Unique identifier for the report run |
| `ssl_cert_check_report_filename` | `ssl_report.md` | Filename for the generated Markdown report |
| `ssl_cert_check_report_path` | `/tmp/{{ ssl_cert_check_report_filename }}` | Full path where the report is written |
| `ssl_cert_check_timezone_offset_seconds` | `28800` | Timezone offset in seconds for display (28800 = UTC+8 Beijing) |

## Dependencies

None.

## Example Playbook

```yaml
---
- name: SSL certificate expiry check and report
  hosts: localhost
  connection: local
  gather_facts: true
  roles:
    - role: ssl_cert_check
      vars:
        ssl_cert_check_sites:
          - www.example.com
          - api.example.com
        ssl_cert_check_warn_days: 30
        ssl_cert_check_critical_days: 10
```

## License

GPL-3.0-or-later

## Author

Red Hat Services
