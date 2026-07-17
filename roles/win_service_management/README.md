# Role: `win_service_management`

Manages Windows services by starting, stopping, restarting, or checking their status and configuring the startup mode.

## Requirements

- ansible-core 2.15+
- Collection `ansible.windows`

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `win_service_management_action` | `check` | Action to perform: `start`, `stop`, `restart`, or `check` |
| `win_service_management_start_mode` | `auto` | Service startup mode (`auto`, `manual`, `disabled`) |
| `win_service_management_services` | `[]` | List of Windows service names to manage |

## Dependencies

None.

## Example Playbook

```yaml
---
- name: Manage Windows services
  hosts: all_windows
  gather_facts: false
  roles:
    - role: win_service_management
      vars:
        win_service_management_action: restart
        win_service_management_services:
          - W3SVC
          - WinRM
```

## License

GPL-3.0-or-later

## Author

Red Hat Services
