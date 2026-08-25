# Role: `user_management`

Creates an administrator user with passwordless sudo privileges and enables session lingering so the user's systemd instance persists after logout.

## Requirements

- ansible-core 2.15+.
- A password hashing backend on the **controller** for `password_hash`:
  libxcrypt, Python's `crypt`, or `passlib`. Python 3.13 removed `crypt`, so on
  a current controller `passlib` must be installed or user creation fails.

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `admin_user` | `admin` | Username for the administrator account to create |
| `admin_password` | `redhat` | Password for the administrator account (hashed with sha512 at runtime). **Override this** — the default is a publicly known value on an account with passwordless sudo |

## Behaviour

- **The password is only set when the account is created** (`update_password:
  on_create`). Re-running does not reset a password changed since, and the task
  no longer reports `changed` on every run — `password_hash` salts randomly, so
  comparing hashes always differed before.
- **Session persistence is `loginctl enable-linger`.** The systemd user
  template unit is keyed by UID, so the previous `user@<name>.service` could
  never resolve.

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
