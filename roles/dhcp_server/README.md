# Role: `dhcp_server`

**Tags:** pillar:network,lifecycle:day1,maturity:l2-process-oriented,platform:linux,tech:dhcp  
**Supported OS:** RHEL/CentOS/Rocky/AlmaLinux/OracleLinux 7/8/9

Deploys and configures a DHCPv4 and DHCPv6 server with package installation, configuration file deployment, service management, and optional firewalld rules.

## Requirements

- No special requirements beyond ansible-core 2.15+.
- `ansible.posix` (for `firewalld` module)

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `dhcp_server_dhcp_packages` | `["dhcp-server"]` | List of DHCP packages to install |
| `dhcp_server_dhcp_services` | `["dhcpd", "dhcpd6"]` | List of DHCP services to enable and start |
| `dhcp_server_dhcp_config_file` | `dhcpd.conf` | Source filename for DHCPv4 configuration |
| `dhcp_server_dhcp6_config_file` | `dhcpd6.conf` | Source filename for DHCPv6 configuration |
| `dhcp_server_dhcp_config_setype` | `dhcp_etc_t` | SELinux context type for DHCP config files |
| `dhcp_server_dhcp_manage_firewall` | `true` | Whether to configure firewalld for DHCP |
| `dhcp_server_dhcp_firewall_services` | `["dhcp", "dhcpv6"]` | Firewalld service names to allow |

## Dependencies

None.

## Example Playbook

```yaml
---
- name: Deploy and configure DHCPv4 and DHCPv6 servers
  hosts: dhcp_servers
  become: true
  roles:
    - role: dhcp_server
      vars:
        dhcp_server_dhcp_manage_firewall: true
```

## License

GPL-3.0-or-later

## Author

Red Hat Services
