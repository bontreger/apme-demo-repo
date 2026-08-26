# Role: `mariadb_server`

**Tags:** pillar:db,lifecycle:day1,maturity:l2,platform:linux,tech:mariadb  
**Supported OS:** RHEL/CentOS/Rocky/AlmaLinux/OracleLinux 7/8/9

Deploys and configures a MariaDB server with security hardening, database and user creation, initial backup, and optional firewall configuration.

## Requirements

- No special requirements beyond ansible-core 2.15+.
- `community.mysql` (for `mysql_user` and `mysql_db` modules)
- `ansible.posix` (for `firewalld` module)

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `mariadb_server_mariadb_packages` | `["mariadb-server", "python3-PyMySQL"]` | List of packages to install |
| `mariadb_server_mariadb_service_name` | `"mariadb"` | Name of the MariaDB systemd service |
| `mariadb_server_mariadb_root_password` | `"YourSecureRootPassword"` | Root password for MariaDB (override with Ansible Vault) |
| `mariadb_server_mariadb_general_user_name` | `"webapp_user"` | Name of the application database user to create |
| `mariadb_server_mariadb_general_user_password` | `"AnotherSecurePassword"` | Password for the application database user (override with Ansible Vault) |
| `mariadb_server_mariadb_database_name` | `"webapp_db"` | Name of the application database to create |
| `mariadb_server_mariadb_backup_path` | `"/root"` | Directory for the initial database backup |
| `mariadb_server_mariadb_backup_filename` | `"{{ mariadb_server_mariadb_database_name }}_initial_backup.sql"` | Filename for the initial database backup |
| `mariadb_server_mariadb_manage_firewall` | `true` | Whether to open the firewall for MariaDB |
| `mariadb_server_mariadb_firewall_service` | `"mysql"` | Firewalld service name to allow through the firewall |

## Dependencies

None.

## Example Playbook

```yaml
---
- name: Deploy and configure MariaDB server
  hosts: all_rhel
  become: true
  gather_facts: true
  roles:
    - role: mariadb_server
      vars:
        mariadb_server_mariadb_root_password: "{{ vault_mariadb_root_password }}"
        mariadb_server_mariadb_database_name: "webapp_db"
```

## License

GPL-3.0-or-later

## Author

Red Hat Services
