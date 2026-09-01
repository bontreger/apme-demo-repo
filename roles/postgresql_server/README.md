# Role: `postgresql_server`

**Tags:** pillar:db,lifecycle:day1,maturity:l2-process-oriented,platform:linux,tech:postgresql  
**Supported OS:** RHEL/CentOS/Rocky/AlmaLinux/OracleLinux 8/9/10

Installs PostgreSQL, initializes the database cluster, configures the listening
port and Unix socket directory via a `conf.d` drop-in, and starts and enables
the service. Scope is deliberately minimal: no `pg_hba`, listen addresses,
auth, or firewall management.

## Requirements

- RHEL-family 8–10 (Red Hat, CentOS, Rocky, AlmaLinux, OracleLinux)
- ansible-core 2.15+

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `postgresql_server_port` | `5432` | TCP port PostgreSQL listens on |
| `postgresql_server_data_dir` | `"/var/lib/pgsql/data"` | Path to the PostgreSQL data directory |
| `postgresql_server_unix_socket_directories` | `"/var/run/postgresql"` | Unix socket directory (RHEL default) |

## Tags

| Tag | Purpose |
|-----|---------|
| `postgresql_server` | All role tasks |
| `postgresql_server_validation` | OS compatibility assert |
| `postgresql_server_install` | Package installation |
| `postgresql_server_configure` | Init, conf.d drop-in, service enable |
| `postgresql_server_verify` | Status check / verbose summary |

## Dependencies

None.

## Example Playbook

```yaml
---
- name: Simple PostgreSQL deployment
  hosts: postgres_servers
  become: true
  gather_facts: true
  roles:
    - role: postgresql_server
      vars:
        postgresql_server_port: 5432
```

## License

GPL-3.0-or-later

## Author

Red Hat Services
