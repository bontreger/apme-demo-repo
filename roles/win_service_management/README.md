# Role: `win_service_management`

Manages Windows services by starting, stopping, restarting, or checking their status and configuring the startup mode.

## Requirements

- ansible-core 2.15+
- Collection `ansible.windows`

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `win_service_management_action` | `check` | Action to perform: `start`, `stop`, `restart`, or `check` |
| `win_service_management_start_mode` | `""` | Startup type applied while acting: `auto`, `delayed`, `manual` or `disabled`. Empty leaves the existing startup type untouched |
| `win_service_management_services` | `[]` | List of Windows service names to manage. An empty list is a no-op |

## Behaviour

- **`start_mode` is only applied when set.** It is applied alongside the
  requested action, so a non-empty value changes the startup type of every
  listed service — including on `stop`. Leave it empty to stop a service
  without reconfiguring whether it starts at boot.
- **Every action reports status.** After acting, the role reads each service
  back with `win_service_info` and prints its state and startup type. Services
  that do not exist are reported as `not installed` rather than failing.
- **Idempotency.** `start`, `stop` and `check` are idempotent — a second run
  with the same parameters reports no changes. `restart` is **not**, by design:
  it bounces the service on every run, so a scheduled job using it causes a
  service interruption each time it executes.
- **Empty input is a no-op.** A run with no services listed succeeds having
  done nothing, so a job template that forgets to set
  `win_service_management_services` reports green.

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

Stop a service and prevent it from starting at boot — `start_mode` must be
given explicitly, since `stop` on its own leaves the startup type alone:

```yaml
    - role: win_service_management
      vars:
        win_service_management_action: stop
        win_service_management_start_mode: disabled
        win_service_management_services:
          - Spooler
```

## License

GPL-3.0-or-later

## Author

Red Hat Services
