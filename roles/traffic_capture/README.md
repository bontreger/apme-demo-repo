# Role: `traffic_capture`

**Tags:** pillar:network,lifecycle:day2,maturity:l2-process-oriented,platform:linux  
**Supported OS:** RHEL/CentOS 7/8/9

Captures network traffic using tcpdump on a specified interface, automatically installing tcpdump if needed, and saves the pcap file for offline analysis.

## Requirements

- No special requirements beyond ansible-core 2.15+.

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `traffic_capture_config.enabled` | `true` | Whether to perform traffic capture |
| `traffic_capture_config.duration` | `30` | Maximum capture duration in seconds |
| `traffic_capture_config.packet_count` | `1000` | Maximum number of packets to capture |
| `traffic_capture_config.output_dir` | `/var/tmp` | Directory where pcap files are saved |
| `traffic_capture_config.filename_prefix` | `packet_capture` | Prefix for the generated pcap filename |

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
      vars:
        traffic_capture_config:
          enabled: true
          duration: 60
          packet_count: 5000
          output_dir: /var/tmp
    - role: report_generation
```

## License

GPL-3.0-or-later

## Author

Red Hat Services
