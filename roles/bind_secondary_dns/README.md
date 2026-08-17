# Role: `bind_secondary_dns`

Deploys and configures a BIND secondary (slave) DNS server with named.conf, service management, and optional firewalld rules.

## Requirements

- No special requirements beyond ansible-core 2.15+.
- `ansible.posix` (for `firewalld` module)

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `bind_secondary_dns_bind_secondary_packages` | `["bind"]` | List of packages to install for BIND secondary |
| `bind_secondary_dns_bind_service_name` | `named` | Name of the BIND service |
| `bind_secondary_dns_bind_config_src` | `named-secondary.conf` | Source path for named.conf (role `files/` or absolute path on controller) |
| `bind_secondary_dns_bind_config_dest` | `/etc/named.conf` | Destination path for named.conf |
| `bind_secondary_dns_bind_config_owner` | `root` | Owner of the named.conf file |
| `bind_secondary_dns_bind_config_group` | `named` | Group of the named.conf file |
| `bind_secondary_dns_bind_config_mode` | `0640` | File permissions for named.conf |
| `bind_secondary_dns_bind_config_setype` | `named_conf_t` | SELinux type for named.conf |
| `bind_secondary_dns_bind_manage_firewall` | `true` | Whether to configure firewalld for DNS |
| `bind_secondary_dns_bind_firewalld_service` | `dns` | Firewalld service name to allow |

## Dependencies

None.

## Example Playbook

```yaml
---
- name: Deploy and configure BIND secondary DNS servers
  hosts: secondary_dns
  become: true
  roles:
    - role: bind_secondary_dns
      vars:
        bind_secondary_dns_bind_manage_firewall: true
```

## License

GPL-3.0-or-later

## Author

Red Hat Services
