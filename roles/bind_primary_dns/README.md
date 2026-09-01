# Role: `bind_primary_dns`

**Tags:** pillar:network,lifecycle:day1,maturity:l2-process-oriented,platform:linux,tech:bind  
**Supported OS:** RHEL/CentOS/Rocky/AlmaLinux/OracleLinux 7/8/9

Deploys and configures a BIND primary (master) DNS server with named.conf, zone files, service management, and optional firewalld rules.

## Requirements

- No special requirements beyond ansible-core 2.15+.
- `ansible.posix` (for `firewalld` module)

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `bind_primary_dns_bind_primary_packages` | `["bind"]` | List of packages to install for BIND primary |
| `bind_primary_dns_bind_service_name` | `named` | Name of the BIND service |
| `bind_primary_dns_bind_config_src` | `named-primary.conf` | Source path for named.conf (role `files/` or absolute path on controller) |
| `bind_primary_dns_bind_config_dest` | `/etc/named.conf` | Destination path for named.conf |
| `bind_primary_dns_bind_config_owner` | `root` | Owner of the named.conf file |
| `bind_primary_dns_bind_config_group` | `named` | Group of the named.conf file |
| `bind_primary_dns_bind_config_mode` | `0640` | File permissions for named.conf |
| `bind_primary_dns_bind_config_setype` | `named_conf_t` | SELinux type for named.conf |
| `bind_primary_dns_bind_zone_src_dir` | `zones` | Source directory for zone files (relative to role files/) |
| `bind_primary_dns_bind_zone_dest_dir` | `/var/named/` | Destination directory for zone files |
| `bind_primary_dns_bind_zone_owner` | `root` | Owner of zone files |
| `bind_primary_dns_bind_zone_group` | `named` | Group of zone files |
| `bind_primary_dns_bind_zone_mode` | `0640` | File permissions for zone files |
| `bind_primary_dns_bind_zone_setype` | `named_zone_t` | SELinux type for zone files |
| `bind_primary_dns_bind_manage_firewall` | `true` | Whether to configure firewalld for DNS |
| `bind_primary_dns_bind_firewalld_service` | `dns` | Firewalld service name to allow |

## Dependencies

None.

## Example Playbook

```yaml
---
- name: Deploy and configure BIND primary DNS servers
  hosts: primary_dns
  become: true
  roles:
    - role: bind_primary_dns
      vars:
        bind_primary_dns_bind_manage_firewall: true
```

## License

GPL-3.0-or-later

## Author

Red Hat Services
