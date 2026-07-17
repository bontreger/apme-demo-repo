# Role: `postgresql_server`

Installs PostgreSQL, initializes the database cluster, configures the listening port and Unix socket directory, and starts and enables the service.

## Requirements

- No special requirements beyond ansible-core 2.15+.

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `postgresql_server_port` | `5432` | TCP port PostgreSQL listens on |
| `postgresql_server_data_dir` | `"/var/lib/pgsql/data"` | Path to the PostgreSQL data directory |

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
