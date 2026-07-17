# Role: `unbound_caching_dns`

Deploy and configure an Unbound caching DNS resolver with template-based configuration, service management, and optional firewalld integration.

## Requirements

- No special requirements beyond ansible-core 2.15+.
- `ansible.posix` collection (for `firewalld` module when firewall management is enabled).

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `unbound_caching_dns_unbound_packages` | `["unbound"]` | List of packages to install |
| `unbound_caching_dns_unbound_service_name` | `unbound` | Systemd service name |
| `unbound_caching_dns_unbound_config_template` | `server.conf.j2` | Jinja2 template source file |
| `unbound_caching_dns_unbound_config_dest` | `/etc/unbound/conf.d/ansible_generated.conf` | Destination path for generated config |
| `unbound_caching_dns_unbound_config_owner` | `root` | Config file owner |
| `unbound_caching_dns_unbound_config_group` | `unbound` | Config file group |
| `unbound_caching_dns_unbound_config_mode` | `0644` | Config file permissions |
| `unbound_caching_dns_unbound_config_setype` | `named_conf_t` | SELinux file context |
| `unbound_caching_dns_unbound_manage_firewall` | `true` | Whether to open DNS port in firewalld |
| `unbound_caching_dns_unbound_firewalld_service` | `dns` | Firewalld service name to allow |
| `unbound_caching_dns_unbound_allowed_networks` | `["172.25.0.0/6", "192.168.62.0/24"]` | Networks permitted to query the resolver |
| `unbound_caching_dns_unbound_insecure_domain` | `pvt.example.com` | Domain to mark as insecure (skip DNSSEC validation) |
| `unbound_caching_dns_unbound_forwarder_ip` | `172.25.254.254` | Upstream forwarder IP address |

## Dependencies

None.

## Example Playbook

```yaml
---
- name: Deploy Unbound caching DNS resolver
  hosts: caching_dns
  become: true
  roles:
    - role: unbound_caching_dns
      vars:
        unbound_caching_dns_unbound_allowed_networks:
          - "10.0.0.0/8"
        unbound_caching_dns_unbound_forwarder_ip: "8.8.8.8"
        unbound_caching_dns_unbound_insecure_domain: "internal.example.com"
```

## License

GPL-3.0-or-later

## Author

Red Hat Services
