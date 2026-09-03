# Role: `system_diagnosis`

**Tags:** pillar:network,lifecycle:day2,maturity:l2-process-oriented,platform:linux  
**Supported OS:** RHEL/CentOS 7/8/9

Performs system-level diagnostics including TCP connection statistics, network-related log analysis, system load, and memory usage as part of network latency troubleshooting.

## Requirements

- No special requirements beyond ansible-core 2.15+.

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `system_diagnosis_thresholds.error_packets` | `10` | Threshold for error packet count before flagging |
| `system_diagnosis_thresholds.dropped_packets` | `5` | Threshold for dropped packet count before flagging |
| `system_diagnosis_thresholds.overrun_packets` | `3` | Threshold for overrun packet count before flagging |
| `system_diagnosis_thresholds.tcp_retransmit_rate` | `0.1` | TCP retransmit rate threshold |
| `system_diagnosis_log_keywords` | `["bnx2x", "e1000", "igb", "ixgbe", "network", "ethtool", "link", "carrier"]` | Keywords to search for in system logs |
| `system_diagnosis_performance.max_log_lines` | `1000` | Maximum number of log lines to collect per keyword |
| `system_diagnosis_performance.timeout_seconds` | `30` | Timeout in seconds for diagnostic commands |
| `system_diagnosis_performance.retry_attempts` | `3` | Number of retry attempts for failed commands |
| `system_diagnosis_cleanup_temp_files` | `true` | Whether to remove temporary files after diagnosis |
| `system_diagnosis_backup_original_files` | `true` | Whether to back up original files before modification |

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
      vars:
        diagnosis_thresholds:
          error_packets: 10
          dropped_packets: 5
    - role: traffic_capture
    - role: report_generation
```

## License

GPL-3.0-or-later

## Author

Red Hat Services
