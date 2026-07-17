# Role: `user_management`

Creates an administrator user with passwordless sudo privileges and enables systemd user session persistence.

## Requirements

- No special requirements beyond ansible-core 2.15+.

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `admin_user` | `admin` | Username for the administrator account to create |
| `admin_password` | `redhat` | Password for the administrator account (hashed with sha512 at runtime) |

## Dependencies

None.

## Example Playbook

```yaml
---
- name: RHEL9 System Standardization Configuration
  hosts: rhel9
  become: true
  roles:
    - role: user_management
      vars:
        admin_user: sysadmin
        admin_password: "{{ vault_admin_password }}"
```

## License

GPL-3.0-or-later

## Author

Red Hat Services
