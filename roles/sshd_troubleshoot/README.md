# Role: `sshd_troubleshoot`

**Tags:** pillar:ops,lifecycle:day2,maturity:l2,platform:linux  
**Supported OS:** RHEL/CentOS/Rocky/AlmaLinux/OracleLinux 7/8/9

Troubleshoots SSH connectivity by diagnosing DNS resolution, ensuring the SSHD service is running, verifying host key fingerprints, inspecting network interfaces, and generating remediation reports with a monitoring script.

## Requirements

- No special requirements beyond ansible-core 2.15+.

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `sshd_troubleshoot_target_hostname` | `{{ inventory_hostname }}` | Target hostname to diagnose |
| `sshd_troubleshoot_target_short_name` | `{{ ansible_hostname \| default(inventory_hostname) }}` | Short hostname for /etc/hosts entry |
| `sshd_troubleshoot_hosts_file_path` | `/etc/hosts` | Path to the hosts file for DNS remediation |
| `sshd_troubleshoot_dns_resolution_targets` | `["{{ sshd_troubleshoot_target_hostname }}", "redhat.com", "8.8.8.8"]` | List of targets to test DNS resolution against |
| `sshd_troubleshoot_service_name` | `sshd` | Name of the SSH service to manage |
| `sshd_troubleshoot_port` | `22` | Expected SSH listening port |
| `sshd_troubleshoot_host_keys_path` | `/etc/ssh` | Directory containing SSH host key files |
| `sshd_troubleshoot_report_base_dir` | `/tmp` | Base directory for troubleshooting report output |
| `sshd_troubleshoot_report_prefix` | `ssh_network_fix` | Prefix for the report directory name |
| `sshd_troubleshoot_log_retention_days` | `7` | Number of days to retain troubleshooting logs |
| `sshd_troubleshoot_monitor_interval_seconds` | `300` | Interval in seconds for the generated monitoring script |
| `sshd_troubleshoot_monitor_script_name` | `monitor_ssh_network.sh` | Filename of the generated monitoring script |
| `sshd_troubleshoot_supported_os_families` | `["RedHat", "CentOS"]` | Supported OS families for this role |
| `sshd_troubleshoot_supported_os_versions` | `["7", "8", "9"]` | Supported major OS versions |

## Dependencies

None.

## Example Playbook

```yaml
---
- name: SSH connection troubleshooting and auto-remediation
  hosts: all_rhel
  become: true
  gather_facts: true
  roles:
    - role: sshd_troubleshoot
      vars:
        sshd_troubleshoot_port: 22
```

## License

GPL-3.0-or-later

## Author

Red Hat Services
