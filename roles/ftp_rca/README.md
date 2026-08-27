# Role: `ftp_rca`

**Tags:** pillar:ops,lifecycle:day2,maturity:l2-process-oriented,platform:linux  
**Supported OS:** RHEL/CentOS/Rocky/AlmaLinux/OracleLinux 7/8/9/10

Performs root cause analysis diagnostics on the vsftpd FTP service, including server-side checks, client connectivity tests, and generates a text diagnosis report.

## Requirements

- No special requirements beyond ansible-core 2.15+.
- Inventory groups `ftp_servers` and `ftp_clients` for targeted diagnostics

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `ftp_rca_service` | `vsftpd` | Name of the FTP service |
| `ftp_rca_package` | `vsftpd` | Name of the FTP package |
| `ftp_rca_ports` | `[21]` | List of FTP ports to test connectivity |
| `ftp_rca_conf_path` | `/etc/vsftpd/vsftpd.conf` | Path to the vsftpd configuration file |
| `ftp_rca_collect_journal_lines` | `200` | Number of journal lines to collect for diagnostics |
| `ftp_rca_attempt_service_start` | `false` | Whether to attempt starting the service during diagnosis |
| `ftp_rca_ping_count` | `2` | Number of ICMP pings for client connectivity tests |
| `ftp_rca_test_timeout` | `5` | Timeout in seconds for TCP connectivity tests |
| `ftp_rca_report_dir` | `/var/lib/ansible-reports` | Directory on the managed host for the report |
| `ftp_rca_report_path` | `{{ ftp_rca_report_dir }}/ftp_rca-{{ inventory_hostname }}.txt` | Plaintext report path |

## Dependencies

None.

## Example Playbook

```yaml
---
- name: FTP service fault diagnosis
  hosts: all_rhel
  become: true
  roles:
    - role: ftp_rca
      vars:
        ftp_rca_attempt_service_start: false
        ftp_rca_collect_journal_lines: 200
```

## License

GPL-3.0-or-later

## Author

Red Hat Services
